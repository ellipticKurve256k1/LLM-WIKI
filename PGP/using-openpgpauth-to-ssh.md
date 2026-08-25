# Using an OpenPGP Authentication Subkey for SSH

OpenPGP authentication subkeys (`[A]`) can authenticate SSH connections through `gpg-agent`. This also allows Git to push to GitHub over SSH without exporting the OpenPGP private key as an OpenSSH private key.

```text
Git / OpenSSH → gpg-agent → OpenPGP [A] subkey → GitHub
```

> [!important]
> SSH authentication uses the public key registered to the GitHub account. The Git commit email and GPG commit-signing email do not need to match the SSH key.

## Initial Setup

### 1. Enable SSH support in `gpg-agent`

Add the following setting to `~/.gnupg/gpg-agent.conf`:

```text
enable-ssh-support
```

### 2. Find the authentication subkey keygrip

```bash
gpg -K --with-keygrip
```

Find the `Keygrip` belonging to the authentication subkey marked `[A]`.

### 3. Allow the subkey for SSH

Add the `[A]` subkey's keygrip to `~/.gnupg/sshcontrol`:

```text
ABCDEF1234567890ABCDEF1234567890ABCDEF12
```

### 4. Start `gpg-agent` and connect the current shell

```bash
gpgconf --launch gpg-agent
export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
```

To apply the socket automatically in new Zsh sessions, add this line to `~/.zshrc`:

```bash
export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
```

## Register the Key with GitHub

Export the authentication subkey as an SSH public key:

```bash
gpg --export-ssh-key <KEY_ID>
```

Copy the resulting `ssh-ed25519 ...` public key and register it under:

**GitHub → Settings → SSH and GPG keys → New SSH key**

Register it as an **SSH key**, not only as a GPG key.

## Verify SSH Authentication

Confirm that `gpg-agent` exposes the public key:

```bash
ssh-add -L
```

Then test GitHub authentication:

```bash
ssh -T git@github.com
```

For detailed authentication logs:

```bash
ssh -vT git@github.com
```

## Push to GitHub over SSH

Check whether the repository remote uses SSH:

```bash
git remote -v
```

The remote should have this form:

```text
git@github.com:USERNAME/REPOSITORY.git
```

If it uses `https://github.com/...`, change it to SSH:

```bash
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```

Push the current branch:

```bash
git branch --show-current
git push origin <BRANCH_NAME>
```

For example:

```bash
git push origin master
```

An HTTPS remote may ask for a username or token. The OpenPGP authentication subkey is used only when the remote uses SSH.

## Troubleshoot `Permission denied (publickey)`

If `ssh-add -L` shows the key but `git push` still fails, reset `gpg-agent` and reconnect the current shell to its new SSH socket:

```bash
gpgconf --kill gpg-agent
gpgconf --launch gpg-agent
export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
ssh-add -L
ssh -T git@github.com
```

This can recover from a stale `gpg-agent`, an outdated agent socket, or an OpenPGP card/key state that is visible as a public key but cannot complete an authentication signature.

> [!note] Public-key listing versus actual authentication
> `ssh-add -L` only confirms that an agent exposes a public key. `ssh -T` or `git push` additionally requires the agent or OpenPGP card to perform a real signature with the private key.

Also verify:

- The exported SSH public key is registered in the GitHub account's **SSH keys**.
- That GitHub account has access to the repository.
- `git remote -v` points to the intended repository using an SSH URL.
- If the subkey is stored on an OpenPGP card, the card is connected and detected with `gpg --card-status`; see [[move-gpg-subkey-to-card]].

The private key remains in the GnuPG key store or on the OpenPGP card throughout this process.
