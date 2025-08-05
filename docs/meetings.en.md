# Meeting notes

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
