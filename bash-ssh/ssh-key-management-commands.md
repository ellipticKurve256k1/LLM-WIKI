---
title: SSH Key Management
tags: [reference, ssh-key]
type: reference
priority: 1
finished: true
created_date: 2026-05-05
---

# SSH Key Management

## Generate Key

| Command | Description |
|---------|-------------|
| `ssh-keygen -t ed25519` | Create ED25519 key |
| `chmod 600 ~/.ssh/id_ed25519` | Set secure permissions |
| `ssh-keygen -y -f ~/.ssh/key.pem > ~/.ssh/key.pub` | extract pubkey from private key |

## Add to Agent
| Command | Description |
|---------|-------------|
| `ssh-add ~/.ssh/id_ed25519` | Add key to RAM |
| `ssh-add -t 1h ~/.ssh/id_ed25519` | Add with 1h lifetime |

**add supposed to be private key**

## Manage Keys
| Command | Description |
|---------|-------------|
| `ssh-add -l` | List loaded keys |
| `ssh-add -D` | Delete all keys |

## Connect
| Command | Description |
|---------|-------------|
| `ssh -vT git@github.com` | Test SSH connection (verbose, no shell) |

