# Implementations

The following table lists current implementation efforts. Note that implementations are not complete, and specifications may still change.

| Execution client | Implementer | Assistance | Progress | Notes |
| - | - | - | - | - |
| Besu | _open_ | | | |
| Erigon | [@DarkLord017](https://github.com/DarkLord017) | [@Giulio2002](https://github.com/Giulio2002) | | [EPF proposal](https://github.com/eth-protocol-fellows/cohort-six/pull/214) |
| Geth | [@zsfelfoldi](https://github.com/zsfelfoldi) | | | [EIP author](https://eips.ethereum.org/EIPS/eip-7745) |
| Nethermind | _open_ | | | [Log filter PR](https://github.com/NethermindEth/nethermind/pull/8464) (incompatible design) |
| Nimbus | [@vineetpant](https://github.com/vineetpant), [@RazorClient](https://github.com/RazorClient) | [@advaita-saha](https://github.com/advaita-saha) | | [EPF proposal](https://github.com/eth-protocol-fellows/cohort-six/pull/175) |
| Reth | [@18aaddy](https://github.com/18aaddy), [@SkandaBhat](https://github.com/SkandaBhat) | [@mattsse](https://github.com/mattsse) | | [EPF proposal](https://github.com/eth-protocol-fellows/cohort-six/pull/207), [Log filter PR](https://github.com/paradigmxyz/reth/issues/16999) |

## M0 - Simplified on-chain log index

Initially, a partial implementation of [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745) is targeted with certain modifications.

### `Log` type

The `Log` type is modified to use SSZ `ByteList` instead of `ProgressiveByteList`. Rationale: Not all [SSZ libraries](./implementations-ssz.md) support [EIP-7916](https://eips.ethereum.org/EIPS/eip-7916) at this time.

```python
MAX_TOPICS_PER_LOG = 4
MAX_LOG_DATA_SIZE = uint64(2**24)

class Log(Container):
    address: ExecutionAddress
    topics: List[Bytes32, MAX_TOPICS_PER_LOG]
    data: ByteList[MAX_LOG_DATA_SIZE]
```

### `FilterRow` type

The `FilterRow` type is modified to use SSZ `ByteList` instead of `ProgressiveByteList`. Rationale: Not all [SSZ libraries](./implementations-ssz.md) support [EIP-7916](https://eips.ethereum.org/EIPS/eip-7916) at this time.

```python
MAP_WIDTH = uint64(2**24)
MAPS_PER_EPOCH = uint64(2**10)
MAX_BASE_ROW_LENGTH = uint64(2**3)

type FilterRow = ByteList[MAX_BASE_ROW_LENGTH * log2(MAP_WIDTH) // 8 * MAPS_PER_EPOCH]
```

### `LogIndex` type

The `LogIndex` type is extended with additional fields for debugging, to be filled when [updating the log index](https://eips.ethereum.org/EIPS/eip-7745#updating-the-log-index). Rationale: Makes it easier to detect inconsistencies across implementations.

```python
class LogIndex(Container):
    epochs: Vector[LogIndexEpoch, MAX_EPOCH_HISTORY]
    next_index: uint64

    # Update before incrementing `log_index.next_index`, after
    # `block_delimiter_entry` is added to `log_index.epochs[a].log_entries[b]`
    latest_block_delimiter_index: uint64  # log_index.next_index
    latest_block_delimiter_root: Root     # block_delimiter_entry.hash_tree_root()

    # Update before incrementing `log_index.next_index`, after
    # `log_entry` is added to `log_index.epochs[a].log_entries[b]`
    latest_log_entry_index: uint64  # log_index.next_index
    latest_log_entry_root: Root     # log_entry.hash_tree_root()

    # Update before incrementing `log_index.next_index`, after
    # `row.append` is called inside `add_log_value()`
    latest_value_index: uint32   # log_index.next_index
    latest_layer_index: uint32   # layer_index
    latest_row_index: uint32     # row_index
    latest_column_index: uint32  # column_index
    latest_log_value: Bytes32    # log_value, as given to `add_log_value()`, after `sha2()` hash
    latest_row_root: Root        # row.hash_tree_root()
```

### Persistence

Initially, no persistence to disk is expected, and the log index can be kept in memory. All clients are started before genesis and don't need to perform an initial sync. However, reorgs may occur, and the log index may have to be partially rewinded to switch to a different chain head.

### Other EIPs

At this stage, no other Pureth EIPs should be bundled.

### Network configuration

The various genesis config objects are extended with a new timestamp.

| File | JSON path |
| - | - |
| `genesis.json` | `.config.eip7745Time` |
| `chainspec.json` | `.params.eip7745TransitionTimestamp` |
| `besu.json` | `.config.eip7745Time` |

Each execution client typically reads only one of these files.

### Activation

The log index is initialized from genesis and tracked inside the execution clients. If no timestamp is configured, or if the block timestamp indicates that the configured timestamp has not yet been reached, the log index is still maintained, but the block header stays unchanged.

For blocks with a timestamp `>=` the configured timestamp, a `LogIndexSummary` is derived from the corresponding `LogIndex`, and the block header's `logsBloom` field is replaced with `ssz.serialize(log_index_summary)`, matching the same size as before: 2048 bits (256 bytes). The engine API is unchanged.

```python
class LogIndexSummary(Container):
    root: Root                            # 0x00 - log_index.hash_tree_root()
    epochs_root: Root                     # 0x20 - log_index.epochs.hash_tree_root()
    epoch_0_filter_maps_root: Root        # 0x40 - log_index.epochs[0].filter_maps.hash_tree_root()

    latest_block_delimiter_index: uint64  # 0x60 - log_index.latest_block_delimiter_index
    latest_block_delimiter_root: Root     # 0x68 - log_index.latest_block_delimiter_root
    latest_log_entry_index: uint64        # 0x88 - log_index.latest_log_entry_index
    latest_log_entry_root: Root           # 0x90 - log_index.latest_log_entry_root
    latest_value_index: uint32            # 0xb0 - log_index.latest_value_index
    latest_layer_index: uint32            # 0xb4 - log_index.latest_layer_index
    latest_row_index: uint32              # 0xb8 - log_index.latest_row_index
    latest_column_index: uint32           # 0xbc - log_index.latest_column_index
    latest_log_value: Bytes32             # 0xc0 - log_index.latest_log_value
    latest_row_root: Root                 # 0xe0 - log_index.latest_row_root
```

### Validation

When processing a block with a timestamp `>=` the configured timestamp, its `logsBloom` field has to be compared against the expected value. If it does not match, an error should be logged, indicating both the expected and actual values. The field can be decoded using `LogIndexSummary.deserialize()` to detect differences more easily.

Block processing may be triggered by all of [`engine_newPayloadV4`](https://github.com/ethereum/execution-apis/blob/main/src/engine/prague.md#engine_newpayloadv4), [`engine_forkchoiceUpdatedV3`](https://github.com/ethereum/execution-apis/blob/main/src/engine/cancun.md#engine_forkchoiceupdatedv3), and forward syncing tasks, and validation is required regardless of the originating trigger.

### Testing

Kurtosis can be used to locally simulate a network with the participating clients. It can be set up by following the [installation instructions](https://docs.kurtosis.com/install) and is configured using a [YAML schema](https://github.com/etan-status/ethereum-package?tab=readme-ov-file#configuration).

Save the following config as `~/Downloads/network_params_pureth.yaml`. Note that this uses a [fork](https://github.com/etan-status/ethereum-package) of [ethpandaops/ethereum-package](https://github.com/ethpandaops/ethereum-package) to enable EIP-7745 testing.

```yaml
participants_matrix:
  el:
    - el_type: erigon
      el_image: ethpandaops/erigon:eip-7745-m0
    - el_type: nimbus
      el_image: ethpandaops/nimbus-eth1:eip-7745-m0
    - el_type: reth
      el_image: ethpandaops/reth:eip-7745-m0
  cl:
    - cl_type: nimbus
      cl_image: ethpandaops/nimbus-eth2:eip-7745-m0-minimal

# global_log_level: debug

network_params:
  fulu_fork_epoch: 1
  eip7745_fork_epoch: 2
  preset: minimal

additional_services:
  - spamoor

spamoor_params:
  spammers:
    - scenario: erctx
      config:
        throughput: 10
```

To interact with the network, first start [Orbstack](https://orbstack.dev), and then use the following commands. [Docker](https://docker.com) has a [known issue](https://github.com/ethpandaops/ethereum-package/issues/1127) with Erigon, if the network fails to start up, exit Docker and start Orbstack, then try again.

| Action | Command |
| - | - |
| Stop | `kurtosis enclave rm pureth --force` |
| Start | `kurtosis run --enclave pureth --image-download always github.com/etan-status/ethereum-package "$(cat ~/Downloads/network_params_pureth.yaml)"` |
| Ports | `kurtosis enclave inspect pureth` |
| Logs | `kurtosis service logs pureth -n 999999 -f` |

To test the RPC, use `kurtosis enclave inspect pureth` to find the real port to which the `rpc` / `ws-rpc` port (8545) is mapped, e.g., if it says `ws-rpc: 8545/tcp -> 127.0.0.1:60147`:

```json
curl '127.0.0.1:60147' \
    -X POST -H "Content-Type: application/json" --data @- <<EOF | jq .
{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "eth_getBlockByNumber",
    "params": ["latest", false]
}
EOF
```

To trigger a new build, join [Ethereum R&D Discord](https://discord.gg/aaGHsUaBQB), and use one of the following slash commands in any of the channels. The build takes several minutes, progress can be monitored on [ethpandaops/eth-client-docker-image-builder](https://github.com/ethpandaops/eth-client-docker-image-builder/actions). After the build completes successfully, Kurtosis will automatically use the latest image on subsequent starts.

| Execution client | Build command |
| - | - |
| Erigon | `/build client-el docker_tag:eip-7745-m0 client:Erigon repository:DarkLord017/erigon ref:filtermaps` |
| Nimbus | `/build client-el docker_tag:eip-7745-m0 client:NimbusEL repository:vineetpant/nimbus-eth1 ref:eip-7745-log-value-index` |
| Reth | `/build client-el docker_tag:eip-7745-m0 client:Reth repository:18aaddy/reth ref:feat/2d-logs-filter` |

Please ensure that the branches are regularly rebased on top of the latest stable client release to keep the diff to the upstream project minimal.
