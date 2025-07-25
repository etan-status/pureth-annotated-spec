# Event logs

Smart contracts can emit events when certain operations happen, e.g., an [ERC-20 token transfer](https://eips.ethereum.org/EIPS/eip-20#events). In Solidity, the `event` and `emit` keywords are used for this purpose. The compiler then transforms `emit` instances to one of the `LOG*` [opcodes](https://ethereum.org/en/developers/docs/evm/opcodes/): `LOG0`, `LOG1`, `LOG2`, `LOG3`, or `LOG4`. These opcodes solely differ by the number of `indexed` topics. To enable interoperability between smart contracts written in different programming languages or compiled using different versions, the [smart contract ABI specification](https://docs.soliditylang.org/en/latest/abi-spec.html#events) defines how events should be mapped to `LOG*` opcodes.

## LOG* opcodes

`LOG*` opcodes operate on a simple structure consisting of:

- `address`: the address that's using the opcode; indexed for search
- `topics`: a list of up through four 32-byte values; indexed for search
- `data`: arbitrary data of arbitrary length; not indexed

When using Solidity, the [smart contract ABI](https://docs.soliditylang.org/en/latest/abi-spec.html#events) applies to all `topics` and `data` values, and `topics[0]` is implicitly set to `keccak(event_signature)` over the event name and all argument types (including non-indexed arguments), unless the `event` is marked as `anonymous`. [External databases](https://www.4byte.directory/event-signatures) can be consulted to reverse look up event signatures by their hash.

Note that smart contracts are not required to follow the ABI. For example, this [transaction](https://etherscan.io/tx/0x3618e5f15ea20435ea3b76b3d65e268085a8995f6989b463891e66bc21bb30dc#eventlog#132) packs data more compactly into a `LOG0` opcode using the `assembly` keyword; with ABI encoding, the data length would be a multiple of 32, increasing [gas cost](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a8-log-operations) but simplifying interoperability with other smart contracts. The ethereum.org documentation currently [does not mention this possibility](https://github.com/ethereum/ethereum-org-website/pull/15096).

## eth_getLogs

Logs that have been emitted can be queried using the [`eth_getLogs`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getlogs) JSON-RPC API. The API supports filtering by `address` and indexed `topics`. For the `topics`, the filter is positional, meaning that separate calls are needed to query matches for `topics[0]`, `topics[1]`, `topics[2]` and `topics[3]`. Further, for both the `address` and each of the `topics`, the query can either specify an individual string or an array of strings. Each result will match the individual string or one of the array items at every position. If an empty array is specified, it is treated as a wildcard for that position; this allows searching for matches in later topic positions while ignoring earlier ones.

For example, this filter:

```python
address in [
    "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
    "0x4200000623d0242cccd4e907008583dcb4af6472",
] and topics[1] in [
    "0x00000000000000000000000066a9893cc07d91d95644aedd05d03f95e1dba8af",
    "0x000000000000000000000000001087bc197aa40bb0c6893112fec044e32156c4",
] and topics[2] == (
    "0x00000000000000000000000066a9893cc07d91d95644aedd05d03f95e1dba8af"
)
```

can be queried using:

```json
curl "https://docs-demo.quiknode.pro" \
    -X POST -H "Content-Type: application/json" --data @- <<EOF | jq .
{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "eth_getLogs",
    "params": [
        {
            "address": [
                "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
                "0x4200000623d0242cccd4e907008583dcb4af6472"
            ],
            "topics": [
                [],
                [
                    "0x00000000000000000000000066a9893cc07d91d95644aedd05d03f95e1dba8af",
                    "0x000000000000000000000000001087bc197aa40bb0c6893112fec044e32156c4"
                ],
                "0x00000000000000000000000066a9893cc07d91d95644aedd05d03f95e1dba8af"
            ],
            "blockHash": "0xd8cb79ef9860a7a7f16558270786ca8c58c4b8bf0807f604a1887717064374ad"
        }
    ]
}
EOF
```
