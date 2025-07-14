# Proofs (EIP-7745)

The proof idea is described in the [EIP-7745 finding potential matches](https://eips.ethereum.org/EIPS/eip-7745#finding-potential-matches) section. Note that the proof endpoint can be added at a later stage and is not strictly required to exist at the activation fork. However, it is needed to evaluate whether the design makes sense, and to understand how queries perform.

Early proof prototype for REST API:

- [Test vectors + proof verification doc](https://github.com/zsfelfoldi/eip-7745/)
- [`verifyProof` function](https://github.com/zsfelfoldi/go-ethereum/blob/proof-poc/core/filtermaps/proof.go#L639)

This is not optimized at this stage, and serves as a proof of concept only.
