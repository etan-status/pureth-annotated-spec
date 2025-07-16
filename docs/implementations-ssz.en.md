# Implementations

The following table lists current [EIP-7916](https://eips.ethereum.org/EIPS/eip-7916) / [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495) implementation efforts. Note that implementations are not complete, and specifications may still change.

| Language | Library | Contact | Notes |
| - | - | - | - |
| C# | [Nethermind](https://github.com/NethermindEth/nethermind/tree/master/src/Nethermind/Nethermind.Serialization.Ssz) | _open_ | |
| Go | [dynssz](https://github.com/pk910/dynamic-ssz) | _open_ | |
| Go | Erigon ([1](https://github.com/ledgerwatch/erigon/tree/main/erigon-lib/types/ssz) [2](https://github.com/ledgerwatch/erigon/tree/main/cl/cltypes/solid) [3](https://github.com/ledgerwatch/erigon/tree/main/cl/merkle_tree)) | _open_ | |
| Go | [FastSSZ](https://github.com/prysmaticlabs/fastssz) | _open_ | |
| Go | [karalabe/ssz](https://github.com/karalabe/ssz) | _open_ | |
| Go | [methodical-ssz](https://github.com/OffchainLabs/methodical-ssz) | _open_ | |
| Java | [Teku](https://github.com/Consensys/teku/tree/master/infrastructure/ssz) | _open_ | |
| Nim | [nim-ssz-serialization](https://github.com/status-im/nim-ssz-serialization) | [@etan-status](https://github.com/etan-status) | 🚧 `ProgressiveList` only |
| Python | [remerkleable](https://github.com/ethereum/remerkleable) | [@etan-status](https://github.com/etan-status) | 🚧 `ProgressiveList` only |
| Rust | [ethereum_ssz](https://github.com/sigp/ethereum_ssz) | _open_ | |
| Rust | [Grandine](https://github.com/grandinetech/grandine/tree/develop/ssz) | _open_ | |
| TypeScript | [ChainSafe](https://github.com/ChainSafe/ssz) | _open_ | |
| TypeScript | [micro-eth-signer](https://github.com/paulmillr/micro-eth-signer) | _open_ | |
| Zig | [ssz.zig](https://github.com/gballet/ssz.zig) | _open_ | |

## Tests

- [ethereum/remerkleable](https://github.com/ethereum/remerkleable) contains static tests in [test_impl.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_impl.py) and [test_typing.py](https://github.com/ethereum/remerkleable/blob/master/remerkleable/test_typing.py)
- [ethereum/consensus-spec-tests](https://github.com/ethereum/consensus-spec-tests) contains random tests in [ssz_generic](https://github.com/ethereum/consensus-spec-tests/tree/master/tests/general/phase0/ssz_generic), generated from a [format](https://github.com/ethereum/consensus-specs/blob/master/tests/formats/ssz_generic/README.md) defined in [ethereum/consensus-specs](https://github.com/ethereum/consensus-specs)
