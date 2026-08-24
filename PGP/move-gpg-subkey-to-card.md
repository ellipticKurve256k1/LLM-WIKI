## Move a GPG Subkey to an OpenPGP Card

Check that the card is detected:

```bash
gpg --card-status
````

Edit the key:

```bash
gpg --edit-key <PRIMARY_FINGERPRINT>
```

Select the subkey to move:

```text
gpg> key 1
```

Make sure only the intended subkey is selected (`*`).

Move it to the card:

```text
gpg> keytocard
```

Choose the appropriate slot:

* `1` - Signature key
* `2` - Encryption key
* `3` - Authentication key

Save the changes:

```text
gpg> save
```

Verify:

```bash
gpg --list-secret-keys --keyid-format long
gpg --card-status
```

A subkey stored on the card is typically shown with `>`:

```text
ssb>  ed25519/1234567890ABCDEF
```

**Back up the secret key before using `keytocard`.**
