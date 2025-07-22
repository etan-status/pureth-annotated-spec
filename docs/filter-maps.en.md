# Filter maps (EIP-7745)

[EIP-7745](https://eips.ethereum.org/EIPS/eip-7745) defines a sequential [_log value_ index](./log-value-index.md) that enables correctness and completeness proofs for [`eth_getLogs`](./event-logs.md#eth_getlogs) responses. However, the server would still have to send the entire _log value_ index for the queried range, resulting in gigabytes of data even for the most basic queries. Therefore, [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745) further defines the concept of filter maps.

## Filter maps

A filter map can be visualized as a 2D bitmap.

- [`MAP_WIDTH`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{24}$
- [`MAP_HEIGHT`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{16}$

For every [_log value_](./log-value-index.md#log-values) exactly a single bit is set within a filter map. To achieve a bounded false positive rate even at high log frequencies, a new map is initialized periodically.

- [`VALUES_PER_MAP`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{16}$

The filter maps are further grouped into epochs, matching the [pruning](./log-value-index.md#pruning) mechanism for the [_log value_ index](./log-value-index.md). From the type definitions in [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745#container-types):

```python
type FilterRow = ProgressiveByteList

class LogIndexEpoch(Container):
    filter_maps: Vector[Vector[FilterRow, MAPS_PER_EPOCH], MAP_HEIGHT]
    log_entries: Vector[LogEntry, MAPS_PER_EPOCH * VALUES_PER_MAP]
        # LogEntry containers at the first index of each log event,
        # default(LogEntry) otherwise

class LogIndex(Container):
    epochs: Vector[LogIndexEpoch, MAX_EPOCH_HISTORY]
    next_index: uint64  # next log value index to be added
```

## Filter rows

Within each of the rows of the filter map, one can visualize that the full [`MAP_WIDTH`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants) is further subdivided into [`VALUES_PER_MAP`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants) groups of 8 bits (1 byte) each. For each _log value_ index, the corresponding bit is set in the corresponding group, i.e., _log value_ index 0 sets a bit within `0 ..< 256`, _log value_ index 1 sets a bit within `256 ..< 512`, and so on. Therefore, there cannot be any collisions where a bit is set multiple times.

The [_log value_](./log-value-index.md#log-values) itself, namely the `address` or an individual `topic`, is then used to determine the exact bit position within the group. Vice versa, this means that if a different bit is set than the one pertaining to any given query that the corresponding [log entry](./log-value-index.md#log-entries) is irrelevant and, hence, does not have to be sent.

Further, the _log value index_ itself is also incorporated into the process to determine the bit position. This avoids repeated false positives when the query bit happens to collide with the bit for a popular event such as the [ERC-20 token transfer](https://eips.ethereum.org/EIPS/eip-20#events).

From the [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745#column-mapping) logic:

```python
def get_column_index(log_value_index, log_value):
    log_value_width = MAP_WIDTH // VALUES_PER_MAP                      # constant
    column_hash = fnv_1a(to_binary64(log_value_index) + log_value)     # 64-bit FNV-1A hash
    collision_filter = (
        column_hash // (2**64 // log_value_width) +
        column_hash // (2**32 // log_value_width)
    ) % log_value_width
    return log_value_index % VALUES_PER_MAP * log_value_width + collision_filter
```

This uses the [FNV-1a](https://en.wikipedia.org/wiki/Fowler–Noll–Vo_hash_function#FNV-1a_hash) non-cryptographic hash function, similar to the one used by Python. Further note that the input `log_value` is [additionally hashed](https://eips.ethereum.org/EIPS/eip-7745#updating-the-log-index) with `sha2(log_value)` before calling `get_column_index`.

```python
def address_value(address):
    return sha2(address)

def topic_value(topic):
    return sha2(topic)
```

## Row mapping

A filter row spanning [`VALUES_PER_MAP`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants) still uses 64 KB, which is still a lot of data for queries spanning a large number of blocks, especially since it is also required to send for all negative search results (to prove completeness). Therefore, each filter map consists of [`MAP_HEIGHT`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants) filter rows; each _log event_ only sets a bit in one of them.

To enable efficient proving, the row assignment should be somewhat stable: if there are multiple entries matching a query, they should all be in the same row. To avoid the collision problem when a popular event such as the [ERC-20 token transfer](https://eips.ethereum.org/EIPS/eip-20#events) is assigned to the same row, a maximum length is also introduced after which the row assignment is changed. The [_mapping layer_ index](https://eips.ethereum.org/EIPS/eip-7745#epochs-and-mapping-layers) indicates the number of times that the assignment was changed, and the maximum length increases exponentially on every reassignment until a maximum.

- [`MAX_BASE_ROW_LENGTH`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{3}$
- [`LAYER_COMMON_RATIO`](https://eips.ethereum.org/EIPS/eip-7745#proposed-constants): $2^{4}$

| Layer index | Row capacity |
| - | - |
| 0 | `MAX_BASE_ROW_LENGTH * LAYER_COMMON_RATIO**0` (= $2^{3}$) |
| 1 | `MAX_BASE_ROW_LENGTH * LAYER_COMMON_RATIO**1` (= $2^7$) |
| 2+ | `MAX_BASE_ROW_LENGTH * MAPS_PER_EPOCH` (= $2^{10}$) |

Within a [log epoch](./log-value-index.md#pruning), it is further ensured that the row assignment is stable across multiple maps. The initial row assignment (_mapping layer_ 0) for a _log value_ is only changed once per log epoch, while the row assignment at higher _mapping layers_ is changed more frequently, making proofs even more efficient. [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745#row-mapping) also contains a diagram that visualizes how the row mapping for a particular _log value_ evolves over time at _mapping layers_ 0, 1, and 2.

```python
def get_row_index(map_index, log_value, layer_index):
    layer_factor = MIN(LAYER_COMMON_RATIO ** layer_index, MAPS_PER_EPOCH)
    row_frequency = MAPS_PER_EPOCH // layer_factor
    return from_binary32(sha2(
        log_value +
        to_binary32(map_index - map_index % row_frequency) +
        to_binary32(layer_index)
    )[0:4]) % MAP_HEIGHT
```

Note that the input `log_value` is [additionally hashed](https://eips.ethereum.org/EIPS/eip-7745#updating-the-log-index) with `sha2(log_value)` before calling `get_row_index`.

## Representation in the data structure

Filter maps are sparse, i.e., they only have a very small number of bits set relative to their full capacity. To represent them more efficiently, each filter row is encoded as the list of column indices for which the bit has been set. Given the grouping from [`get_column_index`](#filter-rows), this means that for each _log value_, a 24-bit column index is computed, with the top 16 bits indicating the _log value_ index, and the bottom 8 bits indicating the FNV-1a anti-collision hash value. This sequence of 24-bit values is then stored in the log index, in place of the bitmask. From [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745#updating-the-log-index):

```python
# Mark a single log value on the filter maps
def add_log_value(log_index, log_value):
    bytes_per_column = log2(MAP_WIDTH) // 8   # assumes that map width is a power of 256

    map_index = log_index.next_index // VALUES_PER_MAP
    epoch_index = map_index // MAPS_PER_EPOCH
    map_subindex = map_index % MAPS_PER_EPOCH
    column_index = get_column_index(log_index.next_index, log_value)

    row = []
    layer_index = 0
    while True:
        layer_factor = MIN(LAYER_COMMON_RATIO ** layer_index, MAPS_PER_EPOCH)
        max_row_length = MAX_BASE_ROW_LENGTH * layer_factor
        row_index = get_row_index(map_index, log_value, layer_index)
        row = log_index.epochs[epoch_index].filter_maps[row_index][map_subindex]
        if len(row) < max_row_length * bytes_per_column:
            break
        layer_index += 1
        max_row_length = MIN(
            max_row_length * LAYER_COMMON_RATIO,
            MAX_BASE_ROW_LENGTH * MAPS_PER_EPOCH
        )
    row.append(to_binary32(column_index)[:bytes_per_column])

    log_index.next_index += 1
```

This replaces the [stub](./log-value-index.md#updating-the-log-index) from the [_log value_ index](./log-value-index.md) section.
