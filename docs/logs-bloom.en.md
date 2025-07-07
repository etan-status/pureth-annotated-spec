# Bloom filters

To speed up [`eth_getLogs`](./event-logs.md#eth_getlogs) queries, a [Bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) is maintained in each block's [`logsBloom`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getblockbyhash) field whenever a [smart contract event log](./event-logs.md#log-opcodes) is emitted. This Bloom filter can be consulted to check whether a block or transaction possibly contains any logs that are relevant for a given query. If the Bloom filter check fails, it is guaranteed that no relevant log is present; if the check succeeds, the result is inconclusive and relevant logs may or may not exist.

## Bloom filter

The Bloom filter is a 2048-bit bitmask. For the log `address` and each of the indexed `topics`, up through 3 bits are set (there may be fewer in case of collisions). Then, to query if an entry is possibly present, the same method can be applied to the query value. If any of the bits corresponding to the query are absent in the Bloom filter, there is no match. From the [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) and [ethereum/execution-specs](https://github.com/ethereum/execution-specs/blob/master/src/ethereum/frontier/bloom.py):

```python
def add_to_bloom(bloom: bytearray, bloom_entry: Bytes) -> None:
    """
    Add a bloom entry to the bloom filter (`bloom`).

    The number of hash functions used is 3. They are calculated by taking the
    least significant 11 bits from the first 3 16-bit words of the
    `keccak_256()` hash of `bloom_entry`.

    Parameters
    ----------
    bloom :
        The bloom filter.
    bloom_entry :
        An entry which is to be added to bloom filter.
    """
    hash = keccak256(bloom_entry)

    for idx in (0, 2, 4):
        # Obtain the least significant 11 bits from the pair of bytes
        # (16 bits), and set this bit in bloom bytearray.
        # The obtained bit is 0-indexed in the bloom filter from the least
        # significant bit to the most significant bit.
        bit_to_set = Uint.from_be_bytes(hash[idx : idx + 2]) & Uint(0x07FF)
        # Below is the index of the bit in the bytearray (where 0-indexed
        # byte is the most significant byte)
        bit_index = 0x07FF - int(bit_to_set)

        byte_index = bit_index // 8
        bit_value = 1 << (7 - (bit_index % 8))
        bloom[byte_index] = bloom[byte_index] | bit_value
```

## Verification

To verify that an [`eth_getLogs`](./event-logs.md#eth_getlogs) response is complete, i.e., no log entries were withheld, the client follows this process:

1. Download all canonical execution block headers and consult their `logsBloom` fields
2. If there is a potential match in the block: Proceed by downloading all [transaction receipts](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_gettransactionreceipt) within that block
3. Validate the receipts against the `receiptsRoot` in the corresponding block header (Merkle-Patricia Trie)
4. Inspect each receipt and report all relevant event logs

Note that besides the block, each individual receipt also contains a `logsBloom` field. However, this inner `logsBloom` can only be trusted after validating against the `receiptsRoot` in the block header, for which the full receipt data is required. Therefore, the raw logs are already available, and the per-receipt `logsBloom` is useless for verifying clients.

## False positives

While the Bloom filter never produces false negatives (missing a match), false positives (extra matches) can occur. As more data is added to the Bloom filter, the [false positive rate](https://en.wikipedia.org/wiki/Bloom_filter#Probability_of_false_positives) increases.

In Ethereum, with:

- $n$, the total number of unique addresses and topics across all logs within the block
- $m := 2048$, the number of bits in the Bloom filter
- $k := 3$, the number of bits set per entry

$$
P(fp) = \left( 1 - e^{-kn / m} \right) ^k = \left( 1 - e^{-3 \cdot n / 2048} \right) ^3
$$

```python
# mkdocs: render
# mkdocs: hidecode
import matplotlib.pyplot as plt
import numpy as np

m = 2048
k = 3

ns = np.arange(0, 5000)
ps = 100 * (1 - np.exp(-k * ns / m)) ** k

# Plotting
plt.plot(ns, ps, label="False positive rate [%]")
plt.xlabel("Total number of unique addresses and topics $n$")
plt.ylabel("False positive rate [%]")
plt.grid(True)
```

At a conservative [100 transactions per block](https://etherscan.io/chart/tx), each emitting 2 logs with a unique address and 3 indexed topics, the false positive rate is already at ~33%. To enable Ethereum's [scale L1 strategic initiative](https://protocol.ethereum.foundation/strategic-initiatives), the logs Bloom filter must be replaced.
