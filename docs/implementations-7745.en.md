# Implementations

The following table lists current implementation efforts. Note that implementations are not complete, and specifications may still change.

| Execution client | Contact | Notes |
| - | - | - |
| Besu | _open_ | |
| Erigon | [@DarkLord017](https://github.com/DarkLord017) | EPF permissionless |
| Geth | [@zsfelfoldi](https://github.com/zsfelfoldi) | [EIP-7745 author](https://eips.ethereum.org/EIPS/eip-7745) |
| Nethermind | _open_ | |
| Nimbus | [@vineetpant](https://github.com/vineetpant), [@RazorClient](https://github.com/RazorClient) | [EPF proposal](https://hackmd.io/@vineetpant/SJzcWzYBeg) |
| Reth | [@18aaddy](https://github.com/18aaddy) | [EPF proposal](https://hackmd.io/@0xAaddy/SJoxVs9Exl) |

## SSZ ProgressiveList

If [EIP-7916 ProgressiveList](https://eips.ethereum.org/EIPS/eip-7916) is unavailable in the underlying [SSZ library](./implementations-ssz.en.md), use `List[type, 999999999]` as a temporary placeholder to get unblocked. Once `ProgressiveList` is implemented in the library, the type can be switched easily.
