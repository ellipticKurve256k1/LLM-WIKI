# Using an OpenPGP Authentication Subkey for SSH

OpenPGP authentication subkeys (`[A]`) can be used for SSH authentication through `gpg-agent`.

## 1. Enable SSH support in `gpg-agent`

Add this to:

```text
~/.gnupg/gpg-agent.conf
```

```text
enable-ssh-support
```

## 2. Find the authentication subkey keygrip

```bash
gpg -K --with-keygrip
```

Find the `Keygrip` belonging to the `[A]` subkey.

## 3. Allow the subkey for SSH

Add its keygrip to:

```text
~/.gnupg/sshcontrol
```

Example:

```text
ABCDEF1234567890ABCDEF1234567890ABCDEF12
```

## 4. Restart `gpg-agent`

```bash
gpgconf --kill gpg-agent
gpgconf --launch gpg-agent
```

## 5. Point SSH to `gpg-agent`

```bash
export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
```

## 6. Verify the key

```bash
ssh-add -L
```

The OpenPGP authentication subkey should now appear as an SSH public key.

You can also export the same public key with:

```bash
gpg --export-ssh-key <KEY_ID>
```

## 7. Add it to GitHub

Copy the exported `ssh-ed25519 ...` public key and add it under:

**GitHub → Settings → SSH and GPG keys → New SSH key**

Then test:

```bash
ssh -T git@github.com
```

The authentication flow is:

```text
OpenSSH
   ↓
gpg-agent
   ↓
OpenPGP [A] subkey
   ↓
GitHub
```

The private key remains inside the GnuPG key store and is not exported as an OpenSSH private key.
