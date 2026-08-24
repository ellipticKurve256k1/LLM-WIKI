# GPG Command Reference

## Key Identifier

In most commands, `"fingerprint"` can be replaced with a sufficiently unique **Key ID**.

To list secret keys:

```bash
gpg --list-secret-keys --keyid-format long
```

Example output:

```text
sec   rsa4096/0123456789ABCDEF 2026-01-01 [SC]
```

The value after the slash, such as `0123456789ABCDEF`, is the **long Key ID**.

A full fingerprint is longer and can be displayed with:

```bash
gpg --fingerprint
```

For scripts or security-sensitive operations, using the **full fingerprint** is recommended because Key IDs can collide.

List all keys with subkeys

```bash
gpg --list-keys --keyid-format long
```

List keys with subkeys of certain **fingerprint**

```bash
gpg --list-keys --keyid-format long --fingerprint <last-4-bytes-fp>
```

 - SC: Signing, Certify other subkeys belongs to primary key
 - E: Encryption
 - S: Signing

### Subkeys

- Subkeys are separate cryptographic key pairs associated with a primary key.
- They are usually generated independently, not derived from the primary private key.
- The primary key binds a subkey to itself using a **subkey binding signature**.
- Subkeys can have specific capabilities:
  - `S` — Signing
  - `E` — Encryption
  - `A` — Authentication
- The primary key usually has the `C` (**Certify**) capability.
- Subkeys can be rotated, revoked, or replaced without changing the primary key identity.
- The primary private key can be kept offline while subkey private keys are used on daily-use devices.
- A signature made by a signing subkey can still be traced and verified against its associated primary key.

### Subkey Binding Signature 

- Created by the **primary private key**. - Proves that a subkey belongs to a specific primary key. 
- Can define the subkey's capabilities and expiration.
- Prevents unauthorized subkeys from being attached to the primary key.

---

## Create a Detached Signature

Creates a separate ASCII-armored signature file without modifying the original message:

```bash
gpg -a --detach-sign -u "fingerprint" message.txt
```
- `-a`: ascii-armored
- `-u`: user

This usually creates:

```text
message.txt.asc
```

To verify it:

```bash
gpg --verify message.txt.asc message.txt
```

- `<signature file> <message file>`
---

## Encrypt a File for a Recipient

Encrypts the file using the recipient's public key:

```bash
gpg -a --encrypt --recipient "fingerprint" message.txt
```

This usually creates:

```text
message.txt.asc
```

To decrypt it:

```bash
gpg --decrypt message.txt.asc
```

To save the decrypted content to a file:

```bash
gpg --output message.txt --decrypt message.txt.asc
```

---

## Create a Clear-Signed Message

Creates a human-readable message with an embedded signature:

```bash
gpg -a -u "fingerprint" --clear-sign message.txt
```

- optional:
	- `--digest-algo`: select which hash algorithm
	- ex: `gpg -a -u fingerprint --digest-algo sha512 --clear-sign message.txt`

This usually creates:

```text
message.txt.asc
```

The message remains readable as plain text, while the signature is included in the same file.

To verify it:

```bash
gpg --verify message.txt.asc
```

---

## Encrypt a File with a Password

Encrypts the file using symmetric encryption instead of a public key:

```bash
gpg -a --symmetric message.txt
```

Short form:

```bash
gpg -ac message.txt
```

GPG will prompt for a password and create:

```text
message.txt.asc
```

To decrypt it:

```bash
gpg --decrypt message.txt.asc
```

---

## Import a Public Key from a Keyserver

Downloads and imports a public key using its full fingerprint:

```bash
gpg --keyserver keys.openpgp.org --recv-keys FULL_FINGERPRINT
```

Example:

```bash
gpg --keyserver keys.openpgp.org \
    --recv-keys 0123456789ABCDEF0123456789ABCDEF01234567
```

After importing the key, verify its fingerprint through a trusted channel:

```bash
gpg --fingerprint FULL_FINGERPRINT
```

## Indicating End of Line

gpg --import

  Paste the entire armored public key, including:

  ```bash
  -----BEGIN PGP PUBLIC KEY BLOCK-----

  ...

  -----END PGP PUBLIC KEY BLOCK-----
  ```
  
  Then press **Ctrl+D(Mac / Linux)** **Ctrl + Z(Windows)** after hit a new line to signal end-of-file and import it.

## Creating an Authentication-Only Subkey

Run GPG in expert mode:

```bash
gpg --expert --edit-key <KEY_ID>
```

Then:

```text
gpg> addkey
```

Choose an Ed25519 key and configure its capabilities so that only `Authenticate` remains enabled.

After creating the subkey, verify it with:

```bash
gpg -K
```

You should see something similar to:

```text
ssb   ed25519/XXXXXXXX [A]
```

## Why `--expert` Is Needed

The normal `addkey` menu may only show options such as signing or encryption keys.

Using:

```bash
gpg --expert --edit-key
```

exposes additional key types and capability settings, including authentication-only subkeys.

## SSH Usage

With `gpg-agent` configured for SSH support, the OpenPGP `[A]` subkey can act as an SSH authentication key.

The SSH-formatted public key can also be exported with:

```bash
gpg --export-ssh-key <KEY_ID>
```

This public key can then be added to services such as GitHub as an SSH authentication key.

### Using pgp to ssh
For using openpgp to the ssh-key for git push, then refer to this [[using-openpgpauth-to-ssh]]

# export keys

## export public key

```bash
gpg -a --export <KEYID> > public-key.asc
```

## export private key

```bash
gpg -a --export-secret-keys <KEYID> > secret-key.asc
```