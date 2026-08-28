# References LLM Wiki

## Purpose

This folder is an existing Obsidian reference wiki. Use this file as a semantic routing map into the notes, not as a catalog or replacement for their content.

## Core Principles

- Preserve the current folders, filenames, note conventions, and human-authored content.
- Start with the smallest relevant route and expand only when needed.
- Prefer existing notes over creating duplicates.
- Follow meaningful Obsidian wikilinks; do not add keyword-based link spam.
- Update this routing map only when a domain, primary location, or entry note materially changes.

## Knowledge Routing

### AWS and Terraform Infrastructure

Use for:
- AWS CLI operations, EC2 storage, AWS MCP, IAM access, Terraform configuration, imports, variables, and AWS networking resources

Primary locations:
- `AWS/`
- `Terraform/AWS/`

Entry notes:
- [[AWS/aws-cli-references]]
- [[AWS/aws-mcp-server-codex-setup]]
- [[Terraform/AWS/basic-blocks]]
- [[Terraform/AWS/resource-reference]]
- [[Terraform/AWS/import-resources-references]]

Bridge notes:
- Terraform EC2 and security-group work may route to [[bash-ssh/ssh-key-management-commands]].
- AWS MCP identity and resource inspection may route to [[AWS/aws-cli-references]].

### Containers and Docker

Use for:
- Docker commands, images, containers, Dockerfiles, Compose services, volumes, entrypoints, and installation

Primary location:
- `Docker/`

Entry notes:
- [[Docker/docker-basic-commands]]
- [[Docker/docker-compose]]
- [[Docker/dockerfile]]
- [[Docker/docker-compose-entrypoint-command]]

### Shell, SSH, Linux, Networking, and Terminal Tools

Use for:
- Bash commands and scripts, permissions, SSH keys and jump hosts, tar, Makefiles, Netplan, static IPs, and terminal multiplexing

Primary locations:
- `bash-ssh/`
- `TMUX/`

Entry notes:
- [[bash-ssh/bash-commands]]
- [[bash-ssh/bash-script]]
- [[bash-ssh/ssh-key-management-commands]]
- [[bash-ssh/ubuntu-netplan-ip-configuration]]
- [[TMUX/init-and-basic-setup]]

### Git, GPG, and File Integrity

Use for:
- Git workflows, commit signing, GPG keys and subkeys, OpenPGP cards, SSH authentication through GPG, and SHA-256 verification

Primary locations:
- `GIT/`
- `PGP/`

Entry notes:
- [[GIT/git-commands-and-commit-signing]]
- [[PGP/simple-gpg-command-note]]
- [[PGP/using-openpgpauth-to-ssh]]
- [[PGP/move-gpg-subkey-to-card]]
- [[shahash-commands-by-os]]

### LLMs, Agents, Skills, and MCP

Use for:
- Local LLM engines, GGUF, quantization, embeddings, benchmarks, Codex skills, MCP configuration, and subagent formats

Primary locations:
- `LLMS/`
- `LLMS/Subagents-Format/`

Entry notes:
- [[LLMS/local-llm-engine]]
- [[LLMS/quantization-note]]
- [[LLMS/ai-embedding-model]]
- [[LLMS/Benchmarks]]
- [[LLMS/Subagents-Format/agents-guide]]
- [[LLMS/mcp-tool-search-context]] (MCP tool definitions & context window / Tool Search)

For agent templates, continue to:
- [[LLMS/Subagents-Format/codex]]
- [[LLMS/Subagents-Format/opencode]]

### Bitcoin

Use for:
- Bitcoin study, custody and wallet transfers, BIP39 seeds, chainwork, Taproot, and quantum-risk analysis

Primary location:
- `Bitcoin/`

Hub:
- [[Bitcoin/README]]

Entry notes:
- [[Bitcoin/bip39/how-seed-works]]
- [[Bitcoin/concepts/chainwork]]
- [[Bitcoin/quantum/taproot-is-quantum-vulnerable-while-at-rest]]

For practical custody guidance, search within `Bitcoin/BTCHelp/` after reading the hub.

### SQL and PostgreSQL

Use for:
- SQL tables, CRUD queries, PostgreSQL permissions, and row-level security

Primary location:
- `SQL/`

Entry notes:
- [[SQL/basic-query]]
- [[SQL/postgresql-access]]

### Vim Editing

Use for:
- Vim keys, motions, operators, Ex commands, plugins, and command references

Primary location:
- `VIM/`

Entry notes:
- [[VIM/VIM-KEY]]
- [[VIM/vim-references/Cheetsheet]]
- [[VIM/vim-references/VIM-command-note]]
- [[VIM/vim-references/VIM-Plugin]]

There are multiple notes named `Cheetsheet`; use the full path when routing to avoid ambiguity.

### Obsidian and Markdown Authoring

Use for:
- Dataview queries, frontmatter-driven note views, inline code, Markdown code blocks, and useful HTML in Markdown

Primary location:
- Root of this wiki

Entry notes:
- [[Dataview-Note]]
- [[useful-html-tags-for-markdown]]
- [[Base-inline-code]]

### Device Setup

Use for:
- Standalone device-specific reference material

Entry note:
- [[iphone-font-settings]]

## Navigation Strategy

1. Match the request to the narrowest route above.
2. Read the listed hub or entry notes first.
3. Follow only relevant outgoing wikilinks and bridge notes.
4. Search filenames, note titles, and aliases when the route is insufficient.
5. Search note contents only after title-level navigation fails.
6. Use semantic or vector search only as a fallback when available.

Do not recursively rescan the full wiki for ordinary questions. A broad scan is appropriate for an explicit audit, initial bootstrap, or when routed navigation demonstrably fails.

## Operations

- **ROUTE:** identify the smallest relevant folders and entry notes.
- **QUERY:** answer from routed notes and cite them with contextual Obsidian links; do not create a note for every answer.
- **ADD:** search for equivalent notes and aliases before updating the best existing owner or creating a note.
- **SYNTHESIZE:** create a durable synthesis only for reusable conclusions spanning multiple notes or sources.
- **REFRESH:** inspect changed areas first and update this map only when navigation materially changes.
- **ORGANIZE:** prefer links and aliases over moves or renames; preview and obtain approval for structural changes.
- **LINT:** report broken links, stale routes, orphans, duplicates, missing hub coverage, and convention drift without auto-fixing ambiguity.

## Note Creation and Linking Rules

- Use the `obsidian-markdown` skill whenever creating or substantially editing a vault note.
- Search exact titles, aliases, filename variants, and likely phrases before creating a note.
- Place new knowledge in the most appropriate existing folder and link it to a relevant hub or concept note.
- Use contextual inline wikilinks when they improve future navigation.
- Do not create links solely because notes share a word.

## Existing Note Safety

- Prefer additive, localized, and reversible changes.
- Preserve existing wording, structure, properties, language, and intent.
- Do not bulk normalize frontmatter or headings.
- Do not move, rename, merge, split, or delete notes without explicit authorization and an impact preview.

## Completion Criteria

A wiki task is complete when relevant existing knowledge was checked first, the smallest useful route was used, duplicates were avoided, note edits followed Obsidian conventions, meaningful links were added where useful, this routing map changed only when warranted, and human-authored notes were not unnecessarily rewritten.
