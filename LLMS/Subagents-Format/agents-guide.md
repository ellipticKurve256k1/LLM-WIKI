# `.agents`, Skills, and MCP

## What is `.agents`?

`.agents` is a hidden directory used for files that can be shared by compatible AI coding agents. A name beginning with `.` is hidden by default on macOS and Linux.

Its main convention today is storing reusable **skills**:

```text
~/.agents/skills/               # Personal skills available across projects
<project>/.agents/skills/       # Skills specific to one project or repository
```

Each skill has its own folder with a required `SKILL.md` file. It may also include scripts, references, templates, and other supporting files.

Example:

```text
~/.agents/skills/key-takeaways/
├── SKILL.md
└── agents/
    └── openai.yaml
```

Use `~/.agents/skills/` for your personal reusable skills. Use a project's `.agents/skills/` directory when the skill belongs to that codebase and should be shared with the team.

For a workflow that uses the vault's root `AGENTS.md` as a semantic routing map, see [[LLMS/Subagents-Format/llm-wiki-router-usage|LLM Wiki Router usage]].

## Managing Agent Skills with `npx skills`

[`npx skills`](https://skills.sh/) is a Vercel Labs CLI for discovering, installing, updating, removing, and distributing `SKILL.md`-based Agent Skills. It supports agents such as Codex, Claude Code, Cursor, and OpenCode.

### Install skills

Install skills from a GitHub repository:

```bash
npx skills add vercel-labs/agent-skills
npx skills add https://github.com/kepano/obsidian-skills
```

Limit an installation to a specific agent, install globally, or select one skill:

```bash
npx skills add vercel-labs/agent-skills -a codex
npx skills add vercel-labs/agent-skills -g
npx skills add vercel-labs/agent-skills --skill skill-creator
```

Use a skill temporarily without installing it:

```bash
npx skills use vercel-labs/agent-skills@web-design-guidelines
```

### Manage skills

| Task | Command |
| --- | --- |
| Search for skills | `npx skills find <query>` |
| List installed skills | `npx skills list` |
| Update installed skills | `npx skills update` |
| Remove a skill | `npx skills remove <skill>` |
| Create a `SKILL.md` template | `npx skills init <name>` |

Useful installation options:

| Option | Purpose |
| --- | --- |
| `-g`, `--global` | Install globally |
| `-a`, `--agent` | Target specific agents |
| `-s`, `--skill` | Select specific skills |
| `-l`, `--list` | List skills available in a repository |
| `-y`, `--yes` | Skip confirmation prompts |
| `--all` | Install all detected skills |

### Distribute a custom skill repository

A GitHub repository containing Agent Skills can be distributed with the same `add` command:

```bash
npx skills add https://github.com/<owner>/<skill-repository>
```

The CLI therefore acts like a package manager for reusable Agent Skills: repositories are the distribution source, while each skill remains defined by its own `SKILL.md` and supporting files.

## What `.agents` does not handle

`.agents` does not normally store MCP server connections. MCP configuration is client-specific because each application decides how it launches servers, stores credentials, and applies permissions.

For Codex, MCP configuration is stored in:

```text
~/.codex/config.toml            # Personal MCP configuration
<project>/.codex/config.toml    # Project-scoped MCP configuration
```

## Managing MCP servers in Codex

List configured MCP servers:

```bash
codex mcp list
```

Add a local STDIO MCP server:

```bash
codex mcp add <server-name> -- <server-command>
```

Example:

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

Authenticate an MCP server that supports OAuth:

```bash
codex mcp login <server-name>
```

See all available MCP commands:

```bash
codex mcp --help
```

Inside the Codex terminal UI, use `/mcp` to view active MCP servers.

You can also configure a server directly in `~/.codex/config.toml`:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

## Quick reference

| Purpose | Location or command |
| --- | --- |
| Personal skills | `~/.agents/skills/` |
| Project skills | `<project>/.agents/skills/` |
| Find and install skills | `npx skills find`, `npx skills add` |
| Manage installed skills | `npx skills list`, `npx skills update`, `npx skills remove` |
| Personal Codex MCP configuration | `~/.codex/config.toml` |
| Project Codex MCP configuration | `<project>/.codex/config.toml` |
| List MCP servers | `codex mcp list` |
| View active MCP servers in Codex | `/mcp` |

## References

- [OpenAI documentation: Build skills](https://developers.openai.com/codex/skills)
- [OpenAI documentation: Model Context Protocol](https://developers.openai.com/codex/mcp)
- [Vercel Labs skills repository](https://github.com/vercel-labs/skills)
- [skills.sh](https://skills.sh/)
