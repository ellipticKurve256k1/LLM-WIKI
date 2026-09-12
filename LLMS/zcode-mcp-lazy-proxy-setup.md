---
title: ZCode용 mcp-lazy 설정 가이드
tags:
  - llm
  - mcp
  - zcode
  - context-window
  - macos
aliases:
  - mcp-lazy setup
  - ZCode lazy MCP setup
  - ZCode MCP proxy
created_date: 2026-09-07
---

# mcp-lazy Setup Guide (ZCode, macOS Intel)

Written 2026-09-07, based on a completed real installation on this machine (macOS 12 Monterey, Intel, Node 22).
Companion analysis document: [[LLMS/zcode-codex-harness-context-analysis|하네스 컨텍스트 분석 — ZCode vs Codex CLI]] (§7.1 covers why this proxy was chosen over mcpproxy-go).

> [!important] 2026-09-08 업데이트 — npx 기동 지연 수정
> `npx -y` 방식으로는 세션마다 프록시 기동이 **12초** 걸리는 문제가 발견됐고, **설치본을 직접 `node`로 실행**하는 방식으로 전환해 **0.6초**로 줄였다. 현재 유효한 설정은 §3의 [Step 7](#Step-7—npx-→-직접-node-실행으로-전환-2026-09-08) 참조. 원인과 절차 전체를 그 단계에 기록했다.

## 1. What mcp-lazy Is

[mcp-lazy](https://www.npmjs.com/package/mcp-lazy) (npm, MIT, v0.1.7) is a lazy-loading MCP proxy. Instead of your agent loading every MCP tool schema into its context window at session start, the agent sees only **2 lightweight meta-tools (~300 tokens)** and discovers/invokes real tools on demand.

```
Without mcp-lazy:
  Agent → Todoist (46 tools, ~13k tokens) + exa (3 tools, ~950 tokens) + ...
  = full schemas loaded every session, even when unused

With mcp-lazy:
  Agent → mcp-lazy proxy (mcp_search_tools + mcp_execute_tool, ~300 tokens)
              ↓ on demand (lazy-spawned)
          todoist (via mcp-remote OAuth bridge) / exa (via mcp-remote) / ...
```

Measured result on this machine: **~15k → ~1.5k MCP tokens per session (~90% reduction)**.

Key properties that made it the right choice here (vs mcpproxy-go, which requires macOS 13+):

- Pure Node — macOS 12 Intel에서 정상 동작 (2026-09-08부터는 npx 대신 설치본을 직접 `node`로 실행, Step 7 참조)
- **stdio transport** — ZCode spawns it per session; no daemon, no port, no launchd
- Handles OAuth-requiring remote servers (Todoist) by bridging through `mcp-remote` automatically

## 2. How the Agent Uses It (After Setup)

The agent no longer sees `find-tasks-by-date`, `web_search_exa`, etc. directly. Instead there are two tools:

| Tool               | Args                                             | Purpose                                                                         |
| ------------------ | ------------------------------------------------ | ------------------------------------------------------------------------------- |
| `mcp_search_tools` | `query` (string), `limit` (number, default 5)    | Find tools by keyword. Returns `server_name` + `tool_name` + descriptions.      |
| `mcp_execute_tool` | `tool_name`, `server_name`, `arguments` (object) | Execute a tool found via search. Lazily spawns the backend server on first use. |

Workflow for every MCP task — always search first:

1. `mcp_search_tools(query="today tasks due date")` → `todoist.find-tasks-by-date`
2. `mcp_execute_tool(tool_name="find-tasks-by-date", server_name="todoist", arguments={startDate:"today", overdueOption:"include-overdue"})`

**Critical caveat — search queries must be in English.** The search is weighted substring matching (tool name / description / server description); tool descriptions are English, so a Korean natural-language query like `"오늘 할 일 보여줘"` returns 0 results while `"today tasks due date"` returns `find-tasks-by-date` ranked first. The model must translate user intent into English keywords before searching. (This note is also in the project memory so future sessions do it correctly.)

Search scoring (from source): exact name match +1.0, partial name match +0.8, description keyword +0.6, server description +0.4.

First call to a backend server in a session has a few seconds of spawn overhead (mcp-remote for remote servers); subsequent calls reuse the connection.

## 3. Fresh Installation From Scratch

Everything below was executed and verified on this machine. Total time: ~15-20 minutes, of which OAuth is ~1 minute.

### Step 0 — Prerequisites & backups

```bash
node --version          # must be 18+ (this machine: v22.17.0)
cp ~/.zcode/cli/config.json ~/.zcode/cli/config.json.bak-$(date +%Y%m%d)
```

Note: this machine's default npm cache (`~/.npm/_cacache`) has an EACCES/permission problem from past sudo usage. If `npx` throws `EACCES ... rename ... _cacache`, don't fight it — use a dedicated cache dir (Step 2) instead of fixing the global cache.

### Step 1 — Smoke-test the package

```bash
export npm_config_cache=~/.mcp-lazy/npm-cache   # dedicated cache, avoids the broken ~/.npm
npx -y mcp-lazy --help
# Commands: add, doctor, init, serve
```

`serve` is the stdio MCP proxy entrypoint (this is what ZCode will run).

### Step 2 — Register upstream servers in `~/.mcp-lazy/servers.json`

**Gotcha (cost us one failed `init`): the file schema is `{"servers": {...}}`, NOT `{"mcpServers": {...}}`.** The `mcpServers` key only appears in the *agent config files that `mcp-lazy add` reads* — the stored format differs. Entry fields: `command`, `args` (default `[]`), `env`, `headers` (only used by `add`'s URL conversion), `description` (optional, feeds server-description search scoring).

```json
{
  "servers": {
    "todoist": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://ai.todoist.net/mcp"],
      "env": { "npm_config_cache": "/Users/DongMyeongKang/.mcp-lazy/npm-cache" },
      "description": "Todoist task manager: tasks, projects, sections, labels, comments, reminders, productivity stats"
    },
    "exa": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.exa.ai/mcp?client=agent-plugin", "--header", "x-exa-source: agent-plugin"],
      "env": { "npm_config_cache": "/Users/DongMyeongKang/.mcp-lazy/npm-cache" },
      "description": "Exa AI web search: web search, webpage fetching, deep research agent"
    }
  }
}
```

Notes:

- Remote OAuth servers (Todoist) and plain HTTP servers (exa) both become **stdio entries via `npx -y mcp-remote <url>`** — that's how mcp-lazy proxies URL servers.
- mcp-remote header syntax: `"--header", "key: value"` as separate args (this mirrors what mcp-lazy's own `add` command generates).
- The `env.npm_config_cache` entry makes spawned npx processes use the dedicated cache — this matters because ZCode later spawns `mcp-lazy serve` from its own environment, and any cache permission issue would break session startup.

We wrote this file by hand because `mcp-lazy add` only auto-configures Cursor/Codex/OpenCode/Antigravity, not ZCode.

### Step 3 — Build the tool cache (this is where OAuth happens)

```bash
npx -y mcp-lazy init
```

- Connects to every registered server and snapshots all tool definitions to `~/.mcp-lazy/tool-cache.json`.
- For Todoist, the first run triggers the OAuth flow: `mcp-remote` **opens the browser automatically** and prints the authorize URL. Approve it once. The token is stored locally (mcp-remote's own auth storage) — no re-auth on later runs; this was confirmed by restarting the proxy afterward and connecting without a prompt.
- Expected output: `✓ todoist 47 tools ... ✓ exa 3 tools ... Cache saved: 50 tools from 2 servers`.
- Without `init`, the first agent session instead does discovery at startup (10-30s slower once).

### Step 4 — Wire it into ZCode

Edit `~/.zcode/cli/config.json`:

- Add the proxy as a stdio MCP server (ZCode supports `{"type":"stdio","command",...,"env"}` — verified in the app bundle; `enabled` boolean is honored, `connectServer` skips servers with `enabled: false`):

```json
"mcp": {
  "servers": {
    "Todoist": {
      "type": "http",
      "url": "https://ai.todoist.net/mcp",
      "enabled": false
    },
    "mcp-lazy": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-lazy", "serve"],
      "env": { "npm_config_cache": "/Users/DongMyeongKang/.mcp-lazy/npm-cache" }
    }
  }
}
```

- Set `"enabled": false` on the now-redundant direct servers (Todoist), and disable plugins whose tools moved behind the proxy (exa: `"exa@claude-plugins-official": false` in `plugins.enabledPlugins`).

⚠️ Disabling the exa plugin also removes its bundled skills (exa:search, exa:exa-agent) from the skill list — acceptable trade-off here.

### Step 5 — Verify standalone (optional but recommended, no new session needed)

A minimal Node script speaks JSON-RPC over stdio to the proxy:

```js
// test.mjs — spawn `npx -y mcp-lazy serve`, then:
// 1) initialize → tools/list: expect exactly ["mcp_search_tools","mcp_execute_tool"]
// 2) tools/call mcp_search_tools {query:"today tasks due date"} → find-tasks-by-date first
// 3) tools/call mcp_execute_tool {tool_name:"find-tasks-by-date", server_name:"todoist",
//      arguments:{startDate:"today",overdueOption:"include-overdue"}} → real task data
// 4) same for exa: search "web search internet" → web_search_exa, then call it
```

Measured on this machine: `tools/list` schema = 1,211 bytes (~303 tokens, **97.7% reduction** vs direct Todoist); searches hit the right tools; real calls returned live Todoist tasks and live Exa search results.

### Step 6 — Open a new ZCode session

Config changes only apply to **new sessions**. Expected loadout: mcp-lazy (2) + context7 (2) + browser-use (4) ≈ **~1.5k MCP tokens** (was ~15k). Verify the token drop via `db.sqlite` → `model_usage` → `input_tokens` on the session's first turn (~35.7k → ~22k expected).

### Step 7 — npx → 직접 node 실행으로 전환 (2026-09-08)

npx 방식 그대로 며칠 써 보니 **세션마다 프록시 기동(MCP initialize)에 12초** 걸렸다. 원인은 두 가지:

1. **`npx -y`는 실행마다 패키지를 재검사·재설치하려 한다.** 캐시가 완전해도 수 초, 문제가 있으면 더 걸린다.
2. **이 Mac의 `~/.npm` 캐시에 root 소유 파일이 1,339개** 있다(과거 sudo npm 사용 흔적). npx가 캐시에 쓰려다 `EEXIST ... rename ... EACCES`로 매번 실패하며 더 느려진다. (Step 0의 캐시 문제가 재발견된 것 — 서버 env의 전용 캐시로 우회 중이었어도 npx 본체는 host 캐시를 건드린다.)

**sudo 없이 해결한 절차** — 사용자 폴더 안 prefix에 설치해 host 캐시/전역 경로를 완전히 피한다:

```bash
export npm_config_cache=~/.mcp-lazy/npm-cache
npm install --prefix ~/.mcp-lazy/prefix mcp-lazy mcp-remote
# → ~/.mcp-lazy/prefix/node_modules/mcp-lazy/dist/index.js       (bin: mcp-lazy)
# → ~/.mcp-lazy/prefix/node_modules/mcp-remote/dist/proxy.js     (bin: mcp-remote)
```

**설정 교체 — 세 단계 모두 `node` 직접 실행으로:**

`~/.zcode/cli/config.json`의 `mcp-lazy` 항목:

```json
"mcp-lazy": {
  "type": "stdio",
  "command": "node",
  "args": ["/Users/DongMyeongKang/.mcp-lazy/prefix/node_modules/mcp-lazy/dist/index.js", "serve"],
  "env": { "npm_config_cache": "/Users/DongMyeongKang/.mcp-lazy/npm-cache" }
}
```

`~/.mcp-lazy/servers.json`의 각 서버 (`npx -y mcp-remote <url>` →):

```json
"command": "node",
"args": ["/Users/DongMyeongKang/.mcp-lazy/prefix/node_modules/mcp-remote/dist/proxy.js", "https://ai.todoist.net/mcp"]
```

(exa도 동일 패턴; `env.npm_config_cache`는 더 이상 불필요해 제거했다.)

**결과 (측정):** initialize 핸드셰이크 12.2초 → **0.59초**. 메타툴 2개 로드, Todoist `find-tasks-by-date` 실데이터 조회까지 재검증 완료. OAuth 토큰(`~/.mcp-auth`)은 브리지 실행 방식과 무관하게 유지되어 재인증 불필요.

> [!warning] 유지보수 함정
> prefix 설치는 npx처럼 자동 갱신되지 않는다. 업데이트는 수동으로: `npm update --prefix ~/.mcp-lazy/prefix` (전용 캐시 env와 함께).
>
> host `~/.npm`의 root 소유 파일은 별도로 남아 있다. 한 줄로 근본 해결 가능: `sudo chown -R 501:20 ~/.npm` (암호 입력 필요, 미실행).

## 4. Maintenance

- **New MCP server**: add an entry to `~/.mcp-lazy/servers.json`, re-run `npx -y mcp-lazy init`, done (cache refreshes automatically on fingerprint too). **등록/삭제 후에는 인벤토리 노트 [[LLMS/mcp-lazy-server-inventory]]의 서버 표도 함께 갱신한다.** Note: since Step 7, backend entries launch via `node .../mcp-remote/dist/proxy.js` — copy that pattern instead of `npx -y mcp-remote`.
- **Proxy update (Step 7 이후)**: prefix 설치는 자동 갱신 안 됨 → `export npm_config_cache=~/.mcp-lazy/npm-cache && npm update --prefix ~/.mcp-lazy/prefix`
- **Doctor**: `npx mcp-lazy doctor` shows registered servers and estimated savings (일회성 관리 명령은 npx로 실행해도 무방 — 세션 기동 경로가 아니므로).
- **Adding OAuth servers later**: same as Todoist — the first `init` opens the browser once.

## 5. Troubleshooting

| Symptom                                  | Cause / Fix                                                                                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `npm error EACCES ... _cacache`          | Broken global npm cache on this machine (root-owned files, 2026-09-08 재확인). Use `npm_config_cache=~/.mcp-lazy/npm-cache`. 근본 해결은 `sudo chown -R 501:20 ~/.npm` (미실행). |
| `init` says "No MCP servers registered"  | servers.json uses the wrong top-level key — must be `{"servers": ...}`, not `{"mcpServers": ...}`.                                                |
| `mcp_search_tools` returns 0 results     | Query language: use English keywords (see §2). Also check the tool cache exists (`~/.mcp-lazy/tool-cache.json`).                                  |
| First session start is slow              | Cache missing or fingerprint changed — run `npx -y mcp-lazy init`. **기동이 매번 10초+라면 npx 문제: Step 7처럼 직접 node 실행으로 전환 (12s → 0.6s).** |
| Todoist asks for login again             | OAuth token expired/removed. Any `init` or first tool call re-triggers the browser flow; approve once.                                            |
| ZCode session starts without proxy tools | ZCode spawns the configured `node .../mcp-lazy/dist/index.js serve` — check the server status in ZCode's MCP panel; ensure Node is on PATH for GUI-launched apps. |

## 6. Rollback / Cleanup

```bash
cp ~/.zcode/cli/config.json.bak-20260907 ~/.zcode/cli/config.json   # restore direct connections
# then start a new session
rm -rf ~/.mcp-lazy   # full cleanup (npx-installed, nothing else to uninstall)
```

## Related Notes

- [[LLMS/mcp-lazy-server-inventory|mcp-lazy 등록 MCP 서버 현황]] — 현재 등록된 서버·툴 수·OAuth 토큰 상태 인벤토리 (이 노트는 절차, 저 노트는 현황)
- [[LLMS/zcode-codex-harness-context-analysis|하네스 컨텍스트 분석 — ZCode vs Codex CLI]] — measured context cost and proxy-selection rationale
- [[LLMS/mcp-tool-search-context|MCP 툴 정의와 컨텍스트 윈도우]] — Tool Search and progressive-discovery concepts
- [[LLMS/Subagents-Format/agents-guide|.agents, Skills, and MCP]] — general MCP server configuration reference
- Skill: `~/.agents/skills/mcp-lazy-setup/` — this procedure distilled into a reusable ZCode skill (fresh setup, adding servers, troubleshooting, bundled verification script `scripts/test-proxy.mjs`)
