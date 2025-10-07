# Meeting notes

## 2025-10-07

- Nimbus
    - [@RazorClient](https://github.com/RazorClient) synced with [@danceratops](https://github.com/danceratopz) regarding tests, suggestion to use [Nimbus t8n tool](https://github.com/status-im/nimbus-eth1/blob/master/tools/t8n/readme.md). Questions regarding best way to implement reorg tests, maybe doable with Hive, a sequence of `newPayload`, `forkchoiceUpdated` + checks should be good enough. As SSZ work is mostly done, proposed to also implement [EIP-7708](https://eips.ethereum.org/EIPS/eip-7708) and [EIP-7799](https://eips.ethereum.org/EIPS/eip-7799), the SSZ branch is best for this, as [EIP-7745](https://eips.ethereum.org/EIPS/eip-7745) is unaffected by them, and is already challenging to do standalone
    - [@vineetpant](https://github.com/vineetpant) reported core implementation of EIP-7745 is done, wondering about how to test. Discussed either using `eth_getBlockByNumber`, extending t8n tests (but reorgs are challenging), or waiting for Reth to be ready and launching a Kurtosis network that supports both Nimbus and Reth

## 2025-09-30

- Nimbus
    - [@vineetpant](https://github.com/vineetpant) added support for the EIP-7745 fork, and tested locally as the Kurtosis config issue persisted
    - [@RazorClient](https://github.com/RazorClient) attended [ETHGlobal New Delhi](https://ethglobal.com/events/newdelhi), and is on the final portion of SSZ block implementation (remaining challenges: [SetCode](https://eips.ethereum.org/EIPS/eip-7702) authorizations, some u64 / u256 mismatches, and a few SSZ root mismatches). Regarding testing, reached out to [@danceratops](https://github.com/danceratopz) for contributing to STEEL
- Reth
    - [@18aaddy](https://github.com/18aaddy) attended [ETHGlobal New Delhi](https://ethglobal.com/events/newdelhi), synced up with [@SkandaBhat](https://github.com/SkandaBhat), and continued with the EIP-7745 work. Testing is based on comparing against Geth's log root hash; debugging the tree structure is not easy
- [@etan-status](https://github.com/etan-status) figured out why Kurtosis is failing and updated the [Kurtosis config](./implementations-7745.en.md#testing) (the `el_image` has to be prefixed with the GitHub account name, if a forked repo is used as source). Continued rebasing ePBS on top of [EIP-7688](https://eips.ethereum.org/EIPS/eip-7688) to at least get [SSZ ProgressiveList](https://eips.ethereum.org/EIPS/eip-7916) from the [Pureth](https://ethereum.org/EIPS/eip-7919) scope into consensus. Also explained the [process of proposing an EIP for inclusion](https://ethereum-magicians.org/t/eip-7773-glamsterdam-network-upgrade-meta-thread/21195)

## 2025-09-16

- Erigon
    - No updates
- Nimbus
    - [@vineetpant](https://github.com/vineetpant) mentioned issues with Kurtosis startup (possibly related to the new `eip7745Time`), and also raised a conflict with the existing RLP log type that's still used in receipts when using STEEL
        - Renaming the log type that's used for the index could resolve this, so that receipts are kept untouched
        - [@etan-status](https://github.com/etan-status) will look into the Kurtosis issue
    - [@RazorClient](https://github.com/RazorClient) is mostly ready, but roots don't match yet between STEEL and Nimbus (including for withdrawals etc); some test cases cannot get the root in Nim. Idea to research SSZ engine API
        - [@etan-status](https://github.com/etan-status) expressed interest in feedback that could be upstreamed into the specs (inefficiencies, ideas etc). Extending STEEL tests with [@danceratops](https://github.com/danceratopz) would be very valuable for other implementers
- Reth
    - [@SkandaBhat](https://github.com/SkandaBhat) raised a [PR for Reth](https://github.com/paradigmxyz/reth/pull/18305) (feedback welcome) with most types in `crates/log-index/common/src/types.rs` that is on par with the Geth functionality. Note that this is only for log query acceleration, it does not come with the consensus changes. Speedup is about 2x, receipts handling slowing it down
        - Demo video as a resource for other implementers
        - Sync with [@18aaddy](https://github.com/18aaddy) for feedback regarding proofs
        - Sync with [@vineetpant](https://github.com/vineetpant) for aligning the implementation
        - Sync with [@RazorClient](https://github.com/RazorClient) for integration into STEEL tests
- [@etan-status](https://github.com/etan-status) reported that all [SSZ data types](./implementations-ssz.en.md) are now [fully specced out with tests](https://github.com/ethereum/consensus-specs/blob/master/ssz/simple-serialize.md) and implemented in [Python](https://github.com/ethereum/remerkleable) and [Nim](https://github.com/status-im/nim-ssz-serialization). Started work to rebase ePBS on top of the new SSZ data types to potentially get a first step in as part of [Glamsterdam](https://eips.ethereum.org/EIPS/eip-7773)
    - [@SkandaBhat](https://github.com/SkandaBhat) offered to try implementing the new SSZ types in [ethereum_ssz](https://github.com/sigp/ethereum_ssz); up through [M3](./implementations-ssz.en.md#m3---progressivecontainer) is needed for [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495), [M4](./implementations-ssz.en.md#m4---compatibleunion) is only needed for [EIP-7807](https://eips.ethereum.org/EIPS/eip-7807)

## 2025-09-02

- Erigon
    - No updates
- Nimbus
    - [@vineetpant](https://github.com/vineetpant) extended Nimbus `BaseVMState` for stateful log index (no persistence yet) and has Nim unit tests, expects [reaching M0 in 2 weeks](https://hackmd.io/@vineetpant/rkiO4fQ9ll)
    - [@RazorClient](https://github.com/RazorClient) has completed SSZ `hash_tree_root` for receipts and transactions, and has Nim tests that verify SSZ encoding and hashing against expectations
        - Integrating tests into STEEL pending, structure to be decided together with [@danceratops](https://github.com/danceratopz)
- Reth
    - [@18aaddy](https://github.com/18aaddy) finished writing the `log_index_hasher` logic, next working on basic unit tests
- [@etan-status](https://github.com/etan-status) bumped [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495) to revert the mix-in back to `ProgressiveBitlist` for its simplicity, and implemented M3 in Python and Nim. `CompatibleUnion` was split off into [EIP-8016](https://eips.ethereum.org/EIPS/eip-8016), implementations for Python and Nim in progress
    - For testing, Kurtosis config will be updated to have both `eip7745Time` and `eip7807Time` keys (controllable via `eip7745_fork_epoch` / `eip7807_fork_epoch` config keys). Activation order is flexible, can also be both at same time
- [@taxmeifyoucan](https://github.com/taxmeifyoucan) to check whether [@DarkLord017](https://github.com/DarkLord017) is still on board, did not join last few calls

## 2025-08-19

- Erigon
    - [@DarkLord017](https://github.com/DarkLord017) made progress towards M0
- Nimbus
    - [@vineetpant](https://github.com/vineetpant) progressed on log index implementation and also started extending [EELS](https://github.com/ethereum/execution-spec-tests) with unit tests; Logs bloom is not yet updated for M0
    - [@RazorClient](https://github.com/RazorClient) is adopting SSZ transaction and receipt types in the code base, and adding tests with help of [@advaita-saha](https://github.com/advaita-saha)
- Reth
    - [@18aaddy](https://github.com/18aaddy) attended [ETHVietnam](https://www.eth-vietnam.com) (small event with ~16 projects), and working on the log index root hash and tree structure; tests are being based on the [Go implementation](https://github.com/zsfelfoldi/go-ethereum/tree/proof-poc), the EELS tests could possibly also be integrated
    - [@SkandaBhat](https://github.com/SkandaBhat) added log index as a sync stage to Reth, and updated eth_getLogs to access the log index for acceleration if available
- [@etan-status](https://github.com/etan-status) integrated the feedback from [@pk910](https://github.com/pk910) into [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495), `ProgressiveContainer` now mixes in a `ProgressiveBitlist` instead of a `Bitvector[256]` to avoid design space limits
- [@taxmeifyoucan](https://github.com/taxmeifyoucan) asked if a reduced Pureth scope may be integrated into Glamsterdam
    - [@etan-status](https://github.com/etan-status) mentioned that the SSZ library changes might be small enough to combine with the ePBS headliner, so that SSZ library readiness cannot be an argument against Pureth for H fork anymore

## 2025-08-05

- Erigon
    - [@DarkLord017](https://github.com/DarkLord017) made an [EPF proposal](https://github.com/eth-protocol-fellows/cohort-six/pull/214), setup Orbstack and Kurtosis
- Nimbus
    - [@vineetpant](https://github.com/vineetpant) learning Nim language, implemented the new data structures relevant for EIP-7745, and focusing on helper functions next. [M0](./implementations-7745.en.md#m0---simplified-on-chain-log-index) as a target looks good
    - [@RazorClient](https://github.com/RazorClient) made progress on a test suite for SSZ execution blocks, and started implementing SSZ transactions and receipts in Nimbus
- Reth
    - [@SkandaBhat](https://github.com/SkandaBhat) made progress on the Reth [log filter implementation]( https://github.com/SkandaBhat/reth/pull/4)
    - [@18aaddy](https://github.com/18aaddy) is implementing log filter hashing for Reth, also looking into Geth for proof format (note, the format is not final)
- [@etan-status](https://github.com/etan-status) got EIP-7916 into [SSZ specifications](https://github.com/ethereum/consensus-specs/blob/master/ssz/simple-serialize.md), working on EIP-7495 next to complete the SSZ data types
    - [@pk910](https://github.com/pk910) (Dora block explorer) implemented progressive types in Go [dynssz](https://github.com/pk910/dynamic-ssz/pull/17). Should `ProgressiveContainer` limit to 256 fields be dropped?
- [@ShiroObiJohn](https://github.com/ShiroObiJohn) mentioned [ETH Shenzhen conference](https://lu.ma/iqh54330) (~600 attendees, 2 days), with an [Ethereum protocol day](https://lu.ma/toicyty8) side event (co-hosted by Bing from Lodestar)
    - Proposed to give a Chinese talk about Pureth, or at least translate the available resources to Chinese language
    - There will be another event in Spring with focus on the business side

## 2025-07-22

- [@etan-status](https://github.com/etan-status) to set up a page with subtasks for EIP-7745
    - Emit `hash_tree_root` of log index into the existing `logsBloom` field to avoid mechanical followup changes (similar to current Geth branch)
    - `List[type, large N]` instead of `ProgressiveList` until SSZ libraries are ready
    - Provide list of milestones, with a concrete format to target
    - Provide Kurtosis config with eip7745 and eip7919 fork timestamps for feature activation. All others should provide a branch that can be pulled into Kurtosis
- [@vineetpant](https://github.com/vineetpant), [@18aaddy](https://github.com/18aaddy), [@DarkLord017](https://github.com/DarkLord017) all starting with log filter track
    - Specs are somewhat clear, guide website is useful
    - All would like an additional mentor for codebase specific questions (receipts processing, logs, reorgs)
        - Nimbus: [@advaita-saha](https://github.com/advaita-saha) will help out
        - Reth / Erigon: [@etan-status](https://github.com/etan-status) will reach out to the teams and request availability
- [@RazorClient](https://github.com/RazorClient) has FOCIL tasks, interested in looking into STEEL tests for SSZ track
    - STEEL is split into two repos, [ethereum/execution-specs](https://github.com/ethereum/execution-specs) and [ethereum/execution-spec-tests](https://github.com/ethereum/execution-spec-tests) that depend on each other cyclically
    - SSZ EIP-7916 and EIP-7495 work will be provided by [@etan-status](https://github.com/etan-status) in [ethereum/remerkleable](https://github.com/ethereum/remerkleable) repo, can be pulled into STEEL repos
- [@ShiroObiJohn](https://github.com/ShiroObiJohn) is interested in learning about Pureth and providing Chinese translation for this website
    - Simply place new files with ending `.zh.md` next to the `.en.md` files and it should be automatically picked up
- [@taxmeifyoucan](https://github.com/taxmeifyoucan) requested questions to [@etan-status](https://github.com/etan-status) to remain low volume to avoid overload, and shared Vineet's [EPF project proposal](https://github.com/eth-protocol-fellows/cohort-six/pull/175) for review
    - Tasks to be tracked on the website, GitHub projects was brought up as an alternative
