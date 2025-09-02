# Meeting notes

## 2025-09-02

- Erigon
    - No updates
- Nimbus
    - [@vineetpant](https://github.com/vineetpant) extended Nimbus `BaseVMState` for stateful log index (no persistence yet) and has Nim unit tests, expects [reaching M0 in 2 weeks](https://hackmd.io/@vineetpant/rkiO4fQ9ll)
    - [@RazorClient](https://github.com/RazorClient) has completed SSZ `hash_tree_root` for receipts and transactions, and has Nim tests that verify SSZ encoding and hashing against expectations
        - Integrating tests into STEEL pending, structure to be decided together with [@danceratops](https://github.com/danceratopz)
- Reth
    - [@18aaddy](https://github.com/18aaddy) finished writing the `log_index_hasher` logic, next working on basic unit tests
- [@etan-status](https://github.com/etan-status) bumped [EIP-7495](https://eips.ethereum.org/EIPS/eip-7495) to revert the mix-in back to `ProgerssiveBitlist` for its simplicity, and implemented M3 in Python and Nim. `CompatibleUnion` was split off into [EIP-8016](https://eips.ethereum.org/EIPS/eip-8016), implementations for Python and Nim in progress
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
