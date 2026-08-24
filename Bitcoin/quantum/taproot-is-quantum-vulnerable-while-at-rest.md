---
title: Taproot is quantum vulnerable while at rest
tags:
  - quantum
  - bitcoin
  - taproot
  - p2tr
type: reference
status: finished
---

# Taproot is quantum vulnerable while at rest

## Abstract

Taproot `p2tr` is quantum-vulnerable in the same basic way as plain public-key outputs: the locking script contains the public key directly rather than a hash of it. That means a sufficiently capable quantum computer running Shor's algorithm could recover the private key from the public key before the owner spends the coin. This note explains where that exposure comes from in Taproot and why it is materially different from hashed public-key constructions such as `p2pkh` and `p2wpkh`.

## Key Points

- A `p2tr` output commits directly to an x-only public key in the `scriptPubKey`.
- Because the public key is already visible on-chain, an attacker does not need to wait for the owner to spend before attempting a quantum attack.
- This makes an unspent `p2tr` UTXO closer to `p2pk` than to `p2pkh` or `p2wpkh`.
- The main difference is exposure timing, not whether the signature scheme itself is quantum-safe. Schnorr and ECDSA are both broken by large enough quantum computers.
- Taproot's efficiency and privacy benefits do not change this cryptographic fact.

## Details

Taproot outputs use the form:

```text
OP_1 <32-byte output key>
```

That 32-byte value is the Taproot output key `Q`, an x-only secp256k1 public key. In other words, the spend condition visible in the UTXO set already contains the public key itself.

For a key-path spend, the owner proves control of the corresponding private key by producing a Schnorr signature. For a script-path spend, the spender reveals the script and control block, but the important point for quantum analysis is earlier than that: the public key was already exposed when the output was created.

The Taproot construction is usually written as:

```text
Q = P + H(P || m)G
```

Where:

- `P` is the **internal public key**
- `m` is the Merkle root of the script tree
- `H(P || m)` is the tweak scalar
- `Q` is the public output key committed in the `scriptPubKey`

From a Bitcoin spending perspective, `Q` is the relevant locked-to public key. If a quantum computer can solve the elliptic-curve discrete logarithm for secp256k1, then learning the secret corresponding to `Q` is enough to authorize a key-path spend and steal the coin.

That is why Taproot is considered "quantum vulnerable while at rest." The vulnerability exists as soon as the UTXO appears on-chain:

1. The attacker observes the `p2tr` output and extracts `Q`.
2. A large enough quantum computer runs Shor's algorithm against `Q`.
3. The attacker derives the corresponding private key.
4. The attacker signs a conflicting spend before the legitimate owner migrates the funds.

This is different from `p2pkh` and `p2wpkh`:

- In `p2pkh`, the output contains `HASH160(pubkey)`, not the pubkey.
- In `p2wpkh`, the witness program also commits to a hash of the pubkey.
- The public key is only revealed when the owner spends.

So for hashed public-key outputs, an untouched UTXO does not immediately expose the elliptic-curve public key needed for Shor-style key recovery. There is still quantum risk after first spend or address reuse, but the exposure window is much smaller.

By contrast, `p2tr` removes that hiding layer. This is one reason many people say Taproot is "as bad as `p2pk` under quantum attack." That is directionally correct with respect to exposure timing: both publish a spendable public key in the output itself.

Important nuance: this does not mean Taproot is uniquely broken. It means Taproot inherits the same fundamental post-quantum weakness as any output type that directly exposes an elliptic-curve public key. The tradeoff is that Taproot gains efficiency, composability, and privacy properties in the classical-security world, while giving up the pre-spend pubkey hiding that `p2wpkh` still provides.

## Insights

- The real distinction is not `ECDSA` versus `Schnorr`; both fail under Shor's algorithm. The relevant distinction is `hashed pubkey` versus `exposed pubkey`.
- Taproot improves many things in Bitcoin, but quantum resistance is not one of them.
- A good mental model is: `p2tr` optimizes the present-day design space, while `p2wpkh` preserves a limited form of "security by delaying pubkey revelation."
- This also means migration urgency would be higher for coins sitting in `p2tr` outputs if a credible cryptographically relevant quantum computer ever appears.

## Related

- [[Knowledges/References/Bitcoin/README|Bitcoin README]]
- [[how-seed-works]]
