# Implementations

The following table lists current [EIP-7916](https://eips.ethereum.org/EIPS/eip-7916) / [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495) implementation efforts. Note that implementations are not complete, and specifications may still change.

| Language | Library | Implementer | Progress |
| - | - | - | - |
| C# | [Nethermind](https://github.com/NethermindEth/nethermind/tree/master/src/Nethermind/Nethermind.Serialization.Ssz) | _open_ | |
| Go | [dynssz](https://github.com/pk910/dynamic-ssz) | [@pk910](https://github.com/pk910) | M4 (tested through M2) |
| Go | Erigon ([1](https://github.com/ledgerwatch/erigon/tree/main/erigon-lib/types/ssz) [2](https://github.com/ledgerwatch/erigon/tree/main/cl/cltypes/solid) [3](https://github.com/ledgerwatch/erigon/tree/main/cl/merkle_tree)) | [@DarkLord017](https://github.com/DarkLord017) | |
| Go | [FastSSZ](https://github.com/prysmaticlabs/fastssz) | _open_ | |
| Go | [karalabe/ssz](https://github.com/karalabe/ssz) | _open_ | |
| Go | [methodical-ssz](https://github.com/OffchainLabs/methodical-ssz) | _open_ | |
| Java | [Teku](https://github.com/Consensys/teku/tree/master/infrastructure/ssz) | _open_ | |
| Nim | [nim-ssz-serialization](https://github.com/status-im/nim-ssz-serialization) | [@etan-status](https://github.com/etan-status) | M3 |
| Python | [remerkleable](https://github.com/ethereum/remerkleable) | [@etan-status](https://github.com/etan-status) | M3 |
| Rust | [ethereum_ssz](https://github.com/sigp/ethereum_ssz) | [@18aaddy](https://github.com/18aaddy) | |
| Rust | [Grandine](https://github.com/grandinetech/grandine/tree/develop/ssz) | _open_ | |
| TypeScript | [ChainSafe](https://github.com/ChainSafe/ssz) | [@wemeetagain](https://github.com/wemeetagain) | |
| TypeScript | [micro-eth-signer](https://github.com/paulmillr/micro-eth-signer) | _open_ | |
| Zig | [ssz.zig](https://github.com/gballet/ssz.zig) | _open_ | |

## Specs

- [ethereum/consensus-specs](https://github.com/ethereum/consensus-specs) defines all [SSZ specifications](https://github.com/ethereum/consensus-specs/blob/master/ssz/simple-serialize.md).

## Tests

- [ethereum/remerkleable](https://github.com/ethereum/remerkleable) contains static tests in [test_impl.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_impl.py) and [test_typing.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_typing.py).
- [ethereum/consensus-spec-tests](https://github.com/ethereum/consensus-spec-tests) contains random tests in [ssz_generic](https://github.com/ethereum/consensus-spec-tests/tree/master/tests/general/phase0/ssz_generic), generated according to a [format](https://github.com/ethereum/consensus-specs/blob/master/tests/formats/ssz_generic/README.md).

## M1 - ProgressiveList

[EIP-7916](https://eips.ethereum.org/EIPS/eip-7916) introduces `ProgressiveList[type]` and `ProgressiveBitlist`. The bitlist can be deferred, as [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745) only requires `ProgressiveList[type]` for bytes.

### Tests

- In [test_impl.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_impl.py) and [test_typing.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_typing.py): Look for `ProgressiveList`.
- In [ssz_generic](https://github.com/ethereum/consensus-spec-tests/tree/master/tests/general/phase0/ssz_generic): Look for `basic_progressive_list`, and for `ProgressiveTestStruct` in the `containers` category.

## M2 - ProgressiveBitlist

Complete implementation of [EIP-7916](https://eips.ethereum.org/EIPS/eip-7916), including `ProgressiveBitlist`. The bitlist follows a similar Merkle tree shape as the regular list, but uses a slightly different encoding and code path.

### Tests

- In [test_impl.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_impl.py) and [test_typing.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_typing.py): Look for `ProgressiveBitlist`.
- In [ssz_generic](https://github.com/ethereum/consensus-spec-tests/tree/master/tests/general/phase0/ssz_generic): Look for `progressive_bitlist`, and for `ProgressiveBitsStruct` in the `containers` category.

## M3 - ProgressiveContainer

[EIP-7495](https://eips.ethereum.org/EIPS/eip-7495) introduces `ProgressiveContainer` and `CompatibleUnion`. The `ProgressiveContainer` is used by more EIPs, so focus on that one first.

### Tests

- In [test_impl.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_impl.py) and [test_typing.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_typing.py): Look for `ProgressiveContainer`.
- In [ssz_generic](https://github.com/ethereum/consensus-spec-tests/tree/master/tests/general/phase0/ssz_generic): Look for `progressive_containers`.

## M4 - CompatibleUnion

Complete implementation of [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495), including `CompatibleUnion`. This unlocks [EIP-6404](https://eips.ethereum.org/EIPS/eip-6404) `Transaction` serialization.

### Tests

- Implementations and tests are not yet available.

## M5 - Optimizations

Optimized overloads of the data structures may be necessary, e.g., with additional caches. While not strictly required for the initial implementation, they should still eventually be implemented. As optimizations are implementation specific, there are no formal completion criteria.
