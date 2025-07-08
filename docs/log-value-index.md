# Log value index (EIP-7745)

[EIP-7745](https://eips.ethereum.org/EIPS/eip-7745) aims to replace the [Bloom filter](./logs-bloom.en.md) for [smart contract event logs](./event-logs.en.md) with a mechanism that efficiently supports queries with:

- Correctness proofs, i.e., all logs are part of the canonical chain
- Completeness proofs, i.e., no log is missing from the response
- Scalability, i.e., low false positive rate even at high log frequency

This section focuses on the _log value_ index component of the EIP.

## Simple Serialize

The new log index uses [Simple Serialize (SSZ)](https://github.com/ethereum/consensus-specs/blob/dev/ssz/simple-serialize.md) with the [EIP-7916 ProgressiveList](https://eips.ethereum.org/EIPS/eip-7916) extension.

## Log entries

Each log is represented as an SSZ `Container`, with:

- [`ExecutionAddress`](https://github.com/ethereum/consensus-specs/blob/dev/specs/bellatrix/beacon-chain.md#custom-types): `Bytes20` (an address, e.g., of a smart contract)
- `MAX_TOPICS_PER_LOG`: 4 (the maximum number of indexed topics per log)

```python
class Log(Container):
    address: ExecutionAddress
    topics: List[Bytes32, MAX_TOPICS_PER_LOG]
    data: ProgressiveByteList
```

It is subsequently paired with position information to form a log entry:

- [`Root`](https://github.com/ethereum/consensus-specs/blob/b3e83f6691c61e5b35136000146015653b22ed38/specs/phase0/beacon-chain.md#custom-types): `Bytes32` (a Merkle root)

```python
class LogMeta(Container):
    block_number: uint64
    transaction_hash: Root
    transaction_index: uint64
    log_in_tx_index: uint64

class LogEntry(Container):
    log: Log
    meta: LogMeta
```

## Log values

The indexed properties that can be queried through [`eth_getLogs`](./event-logs.md#eth_getlogs), namely the `address` and each of the present `topics`, are called _log values_. The `data` field is not indexed and is not considered a _log value_. A sequential numeric index is assigned to each _log value_ in order of emission. The index is stateful, it does not reset on a new block. However, it can rewind when the blockchain reorgs, as only _log values_ from the canonical chain are stored.

| _Log value_ indices | _Log values_ |
|  - | - |
| **0**, 1, 2, 3 | **Addr**, Topic1, Topic2, Topic3 |
| **4**, 5, 6, 7 | **Addr**, Topic1, Topic2, Topic3 |
| **8**, 9, 10 | **Addr**, Topic1, Topic2 |
| **11**, 12 | **Addr**, Topic1 |
| **13**, 14, 15 | **Addr**, Topic1, Topic2 |
| **16**, 17, 18, 19, 20 | **Addr**, Topic1, Topic2, Topic3, Topic4 |

Next, a large SSZ `Vector` is created. For every `address` _log value_ (highlighted in **bold** above), the matching `LogEntry` is stored at the corresponding _log value_ index in the vector. All other vector indices remain default-initialized as `default(LogEntry)`.

This structure enables proving completeness: Merkle proofs over thie SSZ `Vector` can be used to verify that the server supplied all logs within a given _log value_ index range.

## Pruning

To keep the size of the _log value_ index bounded, values are grouped into epochs which can be pruned incrementally by discarding the data for earlier epochs and retaining only the [`hash_tree_root` summaries](https://github.com/ethereum/consensus-specs/blob/dev/ssz/simple-serialize.md#summaries-and-expansions). This way, Merkle proofs can still be produced for more recent epochs, even as historical data is pruned.

- [`VALUES_PER_MAP`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{16}$
- [`MAPS_PER_EPOCH`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{10}$
- [`MAX_EPOCH_HISTORY`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{24}$

```python
class LogIndexEpoch(Container):
    log_entries: Vector[LogEntry, MAPS_PER_EPOCH * VALUES_PER_MAP]
        # LogEntry containers at the first index of each log event,
        # default(LogEntry) otherwise

class LogIndex(Container):
    epochs: Vector[LogIndexEpoch, MAX_EPOCH_HISTORY]
    next_index: uint64  # next log value index to be added
```

This is a simplified definition of the [EIP-7745 structure](https://eips.ethereum.org/EIPS/eip-7745#container-types) which focuses solely on the _log value_ index.

## Block delimiters

[`eth_getLogs`](./event-logs.md#eth_getlogs) supports querying by block range (`fromBlock` and `toBlock` parameters), and also by individual `blockhash`. Because the `blockhash` is not known at the time when logs are emitted, at the start of processing receipts for a new block, an additional _log value_ is inserted first to indicate the parent `blockhash` and timestamp. The client can verify completeness by obtaining all log entries up through the next block delimiter (for the head block, up to the end of the log index).

```python
class BlockDelimiterMeta(Container):
    block_number: uint64
    block_hash: Root
    timestamp: uint64
    dummy_value: uint64  # 2**64-1

class BlockDelimiterEntry(Container):
    dummy_log: Log        # zero address and empty lists
    meta: BlockDelimiterMeta
```

| Block number | _Log value_ indices | _Log values_ |
| -: |  - | - |
| 0 | **0** | _**block delimiter for block 0**_, inserted at start of block 1 |
| 1 | **1**, 2, 3, 4 | **Addr**, Topic1, Topic2, Topic3 |
| 1 | **5**, 6, 7, 8 | **Addr**, Topic1, Topic2, Topic3 |
| 1 | **9**, 10, 11 | **Addr**, Topic1, Topic2 |
| 1 | **12**, 13 | **Addr**, Topic1 |
| 1 | **14**, 15, 16 | **Addr**, Topic1, Topic2 |
| 1 | **17** | _**block delimiter for block 1**_, inserted at start of block 2 |
| 2 | **18**, 19, 20, 21, 22 | **Addr**, Topic1, Topic2, Topic3, Topic4 |

The _**block delimiter for block 2**_ is only inserted once processing of block 3 starts.

## Verification

To enable verification by clients, the block header is extended with the `log_index_root`, which is computed from the `hash_tree_root` over the entire `LogIndex` structure. Any log Merkle proof describes a subset of the data that was hashed into the `log_index_root`.

```python
# Add all log values emitted in the block to the log index;
# should be called even if the block is empty
def add_block_logs(log_index, block):
    if block.number > 0:
        # add block delimiter entry
        block_delimiter_meta = BlockDelimiterMeta(
            block_hash = block.parent_hash,
            block_number = block.number - 1,
            timestamp = block.parent.timestamp,
            dummy_value = 2**64-1,
        )
        block_delimiter_entry = BlockDelimiterEntry(
            meta = block_delimiter_meta,
        )
        epoch, epoch_index = divmod(
            log_index.next_index,
            VALUES_PER_MAP * MAPS_PER_EPOCH,
        )
        log_index.epochs[epoch].log_entries[epoch_index] = block_delimiter_entry
        log_index.next_index += 1

    # add log entries
    for tx_index, receipt in enumerate(block.receipts):
        tx_hash = sha3(block.transactions[tx_index])
        for log_in_tx_index, log in enumerate(receipt.logs):
            log_meta = LogMeta(
                transaction_hash = tx_hash,
                block_number = block.number,
                transaction_index = tx_index,
                log_in_tx_index = log_in_tx_index,
            )
            log_entry = LogEntry(
                meta = log_meta,
                log = log,
            )
            epoch, epoch_index = divmod(
                log_index.next_index,
                VALUES_PER_MAP * MAPS_PER_EPOCH,
            )
            log_index.epochs[epoch].log_entries[epoch_index] = log_entry
            log_index.next_index += 1
            for topic in log.topics:
                log_index.next_index += 1

    block.log_index_root = hash_tree_root(log_index)
```

This is a simplified version of the [EIP-7745 logic](https://eips.ethereum.org/EIPS/eip-7745#updating-the-log-index) which focuses solely on the _log value_ index.
