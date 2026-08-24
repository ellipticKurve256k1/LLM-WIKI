---
title: how mnemonic seed works
tags:
  - bip39
  - mnemonic
  - bitcoin
type: reference
status: finished
---

# How mnemonic seed works

## Abstract

Briefly explains how BIP39 `mnemonic` words work and how they are used to derive a wallet seed, from which private keys are later generated.

## Key Points

- A BIP39 mnemonic encodes random entropy plus a checksum in a human-readable word list.
- The mnemonic itself is not a private key. It is converted into a seed, and wallet private keys are derived from that seed.
- A Bitcoin private key is one value within an extremely large keyspace, roughly on the order of 10<sup>77</sup> possible values.
- A Bitcoin address is generally derived from a public key through hashing, commonly using SHA256 followed by RIPEMD160 in the traditional P2PKH flow.

## Details

1. The wallet generates random entropy, usually 128 bits for 12 words or 256 bits for 24 words.
2. A checksum is added from the SHA256 hash of that entropy.
3. The combined bits are split into 11-bit groups.
4. Each group maps to one word from the 2048-word BIP39 list, creating the mnemonic phrase.
5. The mnemonic is converted into a 512-bit seed using PBKDF2-HMAC-SHA512 with 2048 rounds.
6. That seed is used by BIP32-style hierarchical deterministic wallets to derive many private keys.
	- HMAC-SHA512 → split into master private key (256-bit) + chain code (256-bit)
7. Each private key produces a public key, and addresses are derived from those public keys.

## Image References

![[entropy-encoding-mnemonic-words.png]]
![[mnemonic-to-seed.png]]

## Insights

- BIP39 is mainly a backup and recovery format for wallet entropy.
- The seed produced from the mnemonic is the root material used by hierarchical deterministic wallets.
- The mnemonic, seed, private key, public key, and address are related, but they are not the same thing.

## Links

- [reference_link](https://tremplin.io/bitcoin-how-is-seed-created/)
