# Light clients in Glamsterdam

In favor for Glamsterdam:

- [EIP-7688: Forward compatible consensus data structures (CL)](https://forkcast.org/upgrade/glamsterdam#eip-7688)
- Production-grade SSZ libraries (EL)

Acceptable in Glamsterdam (but also okay in H*):

- [EIP-2926: Chunk-Based Code Merkleization (EL)](https://forkcast.org/upgrade/glamsterdam#eip-2926)
- [EIP-6404: SSZ transactions (EL)](https://forkcast.org/upgrade/glamsterdam#eip-6404)
- [EIP-6466: SSZ receipts (EL)](https://forkcast.org/upgrade/glamsterdam#eip-6466)
- [EIP-7668: Remove bloom filters (EL)](https://forkcast.org/upgrade/glamsterdam#eip-7668)

Defer to H* (see below for rationale):

- [EIP-7708: ETH transfers emit a log (EL)](https://forkcast.org/upgrade/glamsterdam#eip-7708)
- [EIP-7745: Trustless log index (EL)](https://forkcast.org/upgrade/glamsterdam/#eip-7745)
- [EIP-7919: Pureth - Provable RPC responses (EL)](https://forkcast.org/upgrade/glamsterdam#eip-7919)

## Execution

Wallets and dApps access Ethereum data using [JSON-RPC](https://ethereum.org/developers/docs/apis/json-rpc). While [Helios](https://github.com/a16z/helios) and [Nimbus](https://github.com/status-im/nimbus-eth1/tree/master/nimbus_verified_proxy) enable trust-minimized access to _current_ account data (ETH and token balances, `eth_call`, `eth_estimateGas` and so on), historical data cannot be fetched reliably and efficiently. This is problematic, as trusted infrastructure can be [misconfigured](https://status.alchemy.com/incidents/w9mk7r3d1989), [unavailable](https://status.infura.io/incidents/pwgyw2552rwk), [region-locked](https://decrypt.co/94315/ethereum-infura-cuts-off-users-separatist-areas-ukraine-accidentally-blocks-venezuela), or have [questionable default privacy](https://dexinsider.com/metamask-to-start-logging-user-ip-ethereum-wallet-addresses-with-this-new-policy). Invalid data, e.g., due to [inconsistent responses](https://xcancel.com/URozmej/status/1968238473766478163) or due to a hack, can also lead to [UI manipulation](https://xcancel.com/hosseeb/status/1894769440669204780).

Even when running a personal full-node, JSON-RPC does not provide a complete picture. Any balance may potentially increase as a side-effect of a [`SELFDESTRUCT`](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#ab-selfdestruct) opcode in an arbitrary transaction, without emitting any logs. To support certain smart-contract based wallets using that mechanism, exchanges thus have to rely on trusted indexing infrastructure to [trace every single transaction](https://ethereum-magicians.org/t/eip-7708-eth-transfers-emit-a-log/20034/49) for potential side effects and credit user deposits in a timely manner.

It is essential for good user experience that a basic wallet can be implemented that:

- Only trusts infrastructure for availability, but not for correctness
- Has interactive latency even when using a privacy-preserving link
- Contains the complete account history (completeness proof)
- Uses no external trusted indexers

### Glamsterdam

The following roadmap is proposed for Glamsterdam:

1. Implement SSZ libraries ([EIP-7916](https://eips.ethereum.org/EIPS/eip-7916), [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495), [EIP-8016](https://eips.ethereum.org/EIPS/eip-8016), [Specs](https://github.com/ethereum/consensus-specs/blob/master/ssz/simple-serialize.md), [Implementations](https://pureth.guide/implementations-ssz/))

    SSZ is easy to implement, natively supports proofs of partial data (tree hashes), offers forward compatibility, and is battle-tested in the beacon chain across the years. No more MPT / RLP should be added; SSZ ["is significantly better in many ways"](https://vitalik.eth.limo/general/2024/10/26/futures5.html). Exhaustive tests are available, implementations can be cross-checked. The proposed scope for Glamsterdam only involves _hashing_ of simple objects; no serialization, no large nested objects.

2. Use SSZ for _hashing_ transactions / receipts ([EIP-6404](https://eips.ethereum.org/EIPS/eip-6404), [EIP-6466](https://eips.ethereum.org/EIPS/eip-6466), [RPC add-on](https://github.com/ethereum/EIPs/pull/8884/files), [EPF prototype](https://github.com/RazorClient/nimbus-eth1/tree/Pureth-Upstream))

    On-chain transactions / receipts do not contain all the information from JSON-RPC. Notably, the RPC txhash is not on-chain, it cannot be proven to be included in a transactions-root. `from`, `contractAddress`, `authority`, `logIndex`, `logsBloom` and `gasUsed` are also inefficient to prove and require downloading extra data. Certain L2s (e.g., [Optimism](https://github.com/ethereum/pm/issues/982#issuecomment-2023685073), Offchain Labs) would benefit from proving partial transactions. Web3 purifiers (e.g., [Helios](https://github.com/a16z/helios), [Nimbus](https://github.com/status-im/nimbus-eth1/tree/master/nimbus_verified_proxy)) would benefit from general provability. If also selected, [EIP-8011](https://eips.ethereum.org/EIPS/eip-8011), [EIP-8053](https://eips.ethereum.org/EIPS/eip-8053), and [EIP-8059](https://eips.ethereum.org/EIPS/eip-8059) should be aligned with the SSZ gas types.

3. Deprecating Bloom filters ([EIP-7668](https://eips.ethereum.org/EIPS/eip-7668))

    The [log Bloom filters](./logs-bloom.en.md) have a very high false positive rate and are unreliable for practical use. SSZ receipts no longer contain Bloom filter. In the block header, they could be deprecated in a backwards-compatible way by replacing the field with an all-1's value, effectively raising the false positive rate to 100%.

4. Chunk based code ([EIP-2926](https://eips.ethereum.org/EIPS/eip-2926))

    Hashing code as a tree would be in line with the proposed change for transactions / receipts, and likely shares some maintenance burden for buidlers consuming Ethereum data in a verifying manner. [`ProgressiveByteList`](https://eips.ethereum.org/EIPS/eip-7919) should be considered as an alternative to the current MPT based proposal.

### Future

These EIPs need more time to design and are aimed at readiness for a fork _after_ Glamsterdam.

1. ETH transfer logs ([EIP-7708](https://eips.ethereum.org/EIPS/eip-7708), [add-on](https://github.com/ethereum/EIPs/pull/9003/files), [EIP-7799](https://eips.ethereum.org/EIPS/eip-7799), [EIP-6465](https://eips.ethereum.org/EIPS/eip-6465), [EPF prototype](https://github.com/RazorClient/nimbus-eth1/tree/Pureth-Upstream))

    Ensuring that the ETH balance is _exactly_ the difference of all inputs minus all outputs is essential for reliable accounting, and removes the requirement for external indexers for basic wallet and dApp usecases. This should cover all initiated transactions, all inner non-zero value transfers, all fee payments (base + priority have different recipients!), beacon chain operations, as well as original genesis balances. How exactly this is done needs more discussion. _Only_ doing [EIP-7708](https://eips.ethereum.org/EIPS/eip-7708) is insufficient to remove external indexers as certain ETH balance changes are still not covered.

2. SSZ execution blocks ([EIP-7807](https://eips.ethereum.org/EIPS/eip-7807), [EPF prototype](https://github.com/RazorClient/nimbus-eth1/tree/Pureth-Upstream))

    Completing the transition to SSZ harmonizes the serialization across both consensus and execution layers, and provides a clear path to a binary engine API, which will become important for scaling.

3. SSZ _serialization_ based networking

    By now everything is hashed with SSZ, and we should transition all networking, databases, RPCs etc to use SSZ serialization as well. This can happen side by side to the existing JSON-RPC to be backwards compatible, i.e., new verifying clients use SSZ, existing non-verifying clients continue to use JSON and don't break.

4. Native SSZ transactions ([EIP-6493](https://eips.ethereum.org/EIPS/eip-6493))

    Any future transaction types (e.g., [EIP-7793](https://eips.ethereum.org/EIPS/eip-7793), [EIP-7932](https://eips.ethereum.org/EIPS/eip-7932), [EIP-8030](https://eips.ethereum.org/EIPS/eip-8030)) should be based on native SSZ to reduce processing overhead for computing the signing root. Using native SSZ could be encouraged by providing a gas fee discount in a backwards compatible way, avoiding legacy transactions getting more expensive than commonly assumed 21k gas.

5. Log index ([EIP-7745](https://eips.ethereum.org/EIPS/eip-7745), [EIP-7792](https://eips.ethereum.org/EIPS/eip-7792), [Guide](./event-logs.en.md), [EPF prototype](https://github.com/vineetpant/nimbus-eth1/tree/eip-7745-log-value-index), [Geth proofs](https://github.com/zsfelfoldi/go-ethereum/tree/proof-poc), [Geth database](https://github.com/zsfelfoldi/eip-7745), [Reth database](https://github.com/paradigmxyz/reth/pull/18305))

    A structure is needed to query `eth_getLogs` efficiently, and to prove correctness and completeness, i.e., no results were withheld. There are multiple distribution approaches for the log index, either enshrining an index on-chain, or distributing in an external zk based system. While the current log index proposal has well-thought-through concepts and solves a real need for any wallet or dApp, it is quite tough to implement, the syncing mechanism is still undefined, and the SSZ usage is partially non-standard. The proof format could also benefit from more validation, making this something more fitted as an individual headliner.

## Consensus

On the consensus side, light clients would benefit from these in Glamsterdam:

1. Forward compatible data structures ([EIP-7688](https://eips.ethereum.org/EIPS/eip-7688), [Spec](https://github.com/ethereum/consensus-specs/pull/4630))

    This reduces the maintenance churn (involving security councils) for builders that consume beacon data in their smart contracts, and is mostly benefiting staking pools (e.g., [Rocket Pool](https://xcancel.com/KaneWallmann/status/1816729724145795258), [Diva](https://github.com/ethereum/pm/issues/1754#issuecomment-3409601784), [Lido](https://xcancel.com/d_gusakov/status/1901976368084070780)). Last proposed to discuss on [ACDC](https://github.com/ethereum/pm/issues/1754#issuecomment-3374088024). There are a few cleanup unlocks that should also be considered, e.g., [EIP-8015](https://eips.ethereum.org/EIPS/eip-8015) removes legacy fields from the CL state.

2. Decentralized checkpoint sync ([Idea](https://hackmd.io/@etan-status/electra-lc), [Notes](https://github.com/etan-status/consensus-specs/commit/lc-snapsync#diff-aa2f5c9997dc39e643314490575e8993f91669f66cf24203b3cb670311a45e47), [Scratchpad](https://github.com/status-im/nimbus-eth2/commit/b6cee3d046f0660eb0cc79e7e276951f8fef144c))

    The default method to set up a new Ethereum node is to point it to a trusted beacon-API to download a finalized checkpoint. In practice, this is frequently not verified. Checkpoint distribution services have also proven unreliable during Goerli and Holesky incidents. The bootstrap mechanism should be updated to download the checkpoint via libp2p. This is still in idea stage, but was already confirmed to be possible without consensus changes; it can be shipped incrementally, similar to the existing light client protocol. Steps involve: (1) Efficient SSZ multiproof implementation, (2) Backfill protocol for historical light client data, (3) Adding a historical trusted root into [eth-client configs](https://github.com/eth-clients/mainnet/blob/main/metadata/config.yaml), (4) Integrate [safe long range syncing](https://github.com/metacraft-labs/DendrETH/tree/main/docs/long-range-syncing), (5) Develop the beacon snap sync protocol.
