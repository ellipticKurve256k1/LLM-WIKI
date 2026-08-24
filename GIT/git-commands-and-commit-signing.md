---
title: git commands and commit signing
type: note
reference: note
finished: true
created_date: 2026-04-19
tags: [git, gpg, terminal, commands]
---

# Git Commands And Commit Signing

## Definition

Quick reference for common Git remote, author, reset, and GPG signing commands.

## Key Ideas

- `git remote` commands manage the repository URL used for push and fetch operations.
- `git config` sets the author identity and signing behavior used when creating commits.
- `git reset --hard` rewrites the working tree to match a target commit and discards local changes.
- Commit signing requires a GPG key, a configured signing key, and a matching email identity.

## Examples

- Use `git remote set-url origin https://access-token@github.com/username/repository.git` when switching an existing remote to token-based HTTPS authentication.
- Use `git commit --amend --author "username <email>" -m "commit"` when changing the author on the most recent commit.
- Use `git log --show-signature --pretty=fuller` to verify whether a commit was signed and which identity was attached.

## Commands

### Remote Setup

```bash
git remote add origin
```

Adds a remote named `origin`.

```bash
git remote set-url origin https://access-token@github.com/username/repository.git
```

Updates the `origin` remote URL to use HTTPS with a personal access token.

### Author Identity

```bash
git config user.name "Your name"
git config user.email "Your email"
git config --get user.name
```

Sets the local Git author identity and shows the current configured username.

### Commit Author Updates

```bash
git commit --author "username <email>" -m "commit"
git commit --amend --author "username <email>" -m "commit"
```

### Commit Staging and Tracking

#### The Three-Tier System

| State | Status | Location | Action to Move |
| :--- | :--- | :--- | :--- |
| **Tracked / Unstaged** | Modified | Working Directory | `git add` $\rightarrow$ Staged |
| **Tracked / Staged** | Ready | Staging Area (Index) | `git commit` $\rightarrow$ History |
| **Untracked** | New | Working Directory | `git add` $\rightarrow$ Tracked & Staged |

#### The "Undo" Logic

-   `git restore --staged <file>`: Moves from **Staged** $\rightarrow$ **Unstaged**.
    - *Result:* Git still tracks it. Your changes are safe. It just won't be in the next commit.
-   `git rm --cached <file>`: Moves from **Tracked** $\rightarrow$ **Untracked**.
    - *Result:* Git stops watching the file entirely. Useful for files like `.DS_Store` or `tags`.

### Commit with tracked staged files

 - stages and commits in one step, but only for tracked files.

```bash
git commit -a -m "msg"
```

Creates a commit with an explicit author, or rewrites the most recent commit to change its author.

### Reset Commands

```bash
git reset --hard HEAD
git reset --hard HEAD~1
```

Resets the working tree to the current commit or the previous commit and discards local changes.

### GPG Key Inspection

```bash
gpg --list-secret-keys --keyid-format long
```

Lists secret keys and shows the long key ID used for Git signing.

```bash
where gpg.exe
git config gpg.program
git config --global gpg.program "$PATH"
```

Finds the installed GPG executable, checks the configured Git GPG program path, and sets it globally.

### Commit Signing

```bash
git config --global user.signingkey "key id"
git config --global commit.gpgsign true
git commit -S -m "msg"
```

Sets the signing key, enables automatic commit signing, and signs a single commit explicitly.

### Signature Verification

```bash
git log --show-signature --pretty=fuller
git log --format=full
```

Shows commit signature details and displays full author and committer information.

### git blame

Annotates each line of a file with who last modified it, when, and in which commit.

Basic command: 

```bash
git blame filename
```

Options:

- setup line range

```bash
git blame filename -L from,to
```

## Thoughts

- The original text mixed setup, destructive reset commands, and signing checks in one dump, so this version separates them by task.
- Commit verification depends on three values matching: the uploaded public key, the configured signing key, and the commit author email.
