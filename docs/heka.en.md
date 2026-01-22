# Purge EIPs for Bogota headliner

After focusing on scaling with PeerDAS (CL) and Block Level Access Lists (EL), a [focus on simplification](https://www.reddit.com/r/ethereum/comments/1qg4ay1/protocol_simplicity_as_necessary_part_of/) is proposed.

## EIP List

### Serialization library

1. [Basic SSZ implementation](https://github.com/ethereum/consensus-specs/blob/master/ssz/simple-serialize.md)
2. [EIP-7916: SSZ ProgressiveList](https://eips.ethereum.org/EIPS/eip-7916)
3. [EIP-7495: SSZ ProgressiveContainer](https://eips.ethereum.org/EIPS/eip-7495)
4. [EIP-8016: SSZ CompatibleUnion](https://eips.ethereum.org/EIPS/eip-8016)

These EIPs lay the groundwork for adopting Ethereum's SimpleSerialize (SSZ) binary format in the EL.

Note that SSZ libraries that are backing CLs would already contain mature support for most of this scope by Bogota (except CompatibleUnion), as part of [EIP-7688](https://eips.ethereum.org/EIPS/eip-7688) (currently CFI).

### Transaction revamp

1. [EIP-7932: Secondary Signature Algorithms](https://eips.ethereum.org/EIPS/eip-7932) (minus precompile)
2. [EIP-6404: SSZ transactions](https://eips.ethereum.org/EIPS/eip-6404)
3. [EIP-8116: Replace cumulative receipt fields](https://eips.ethereum.org/EIPS/eip-8116)
4. [EIP-6466: SSZ receipts](https://eips.ethereum.org/EIPS/eip-6466)
5. [EIP-6493: SSZ transaction signature scheme](https://eips.ethereum.org/EIPS/eip-6493)

These EIPs improve verifiability by trust-minimized client applications, by enabling proving partial calldata and logdata. They further provide a native hash for individual logs, reducing the added complexity from enshrined log indexing schemes. The amount of data to fetch from databases is reduced when serving related transaction and receipt RPCs, as the on-chain data matches relevant data more closely. Finally, the design allows signature schemes to evolve independently of transaction types.

### Removing conversion overhead

1. [EIP-6465: SSZ withdrawals root](https://eips.ethereum.org/EIPS/eip-6465)
2. [EIP-7807: SSZ execution blocks](https://eips.ethereum.org/EIPS/eip-7807)
3. SSZ binary engine API (not yet defined)

This eliminates all Merkle Patricia Tries except the state trie, aligning the structures of EL blocks and CL ExecutionPayloads. The engine API would transition to the same REST based approach that is used in the CL beacon-APIs, and would use native SSZ for communication, reducing latencies when scaling to higher blob counts, and cutting hash overhead.

### Out of scope

To keep scope manageable, log indexing based EIPs ([EIP-7799](https://eips.ethereum.org/EIPS/eip-7799), [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745)) are deferred to a later fork. This enables those proposals to mature, while working towards building blocks that log indices can build on.
