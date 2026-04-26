---
name: warp-oz
description: Orchestrate Warp's Oz CLI as an autonomous coding agent — local and cloud execution, scheduling, REST API, MCP integration, and background monitoring.
version: 1.0.0
author: Dascoba
license: MIT
homepage: https://github.com/dbardi/agent-skills/tree/main/openclaw/warp-oz
metadata:
  clawdbot:
    emoji: "🚀"
    requires:
      env: ["WARP_API_KEY"]
    primaryEnv: "WARP_API_KEY"
---

# Warp Oz CLI

Use [Warp Oz](https://docs.warp.dev/reference/cli/cli) as an autonomous coding agent. Oz is Warp.dev's agent orchestration platform — it executes AI coding agents locally or on Warp cloud runners, with scheduling, API control, skills-as-agents, environments, triggers, and self-hosting.

> **Note:** `warp-cli` is deprecated and replaced by `oz`. All commands use `oz`.

## When to Use

- You want an external coding agent to implement, refactor, or review code
- Cloud execution is preferred over local (isolated runners, no resource drain)
- Scheduled/recurring agent tasks (dependency updates, code cleanup, issue triage)
- Integration with MCP servers, triggers (Slack, Linear, GitHub), or agent ecosystems
- Self-hosted execution on your own infrastructure

## Prerequisites

- Oz CLI installed (see Installation below)
- Auth configured (API key or interactive login)
- Warp account with API access
- `WARP_API_KEY` set for headless/VM use

## Installation

| Platform | Method |
|----------|--------|
| **macOS** | `brew tap warpdotdev/warp && brew install --cask oz` |
| **Windows** | `winget install Warp.Warp` — check "Add to PATH" |
| **Linux (Debian/Ubuntu)** | Add Warp apt repo, then `sudo apt install oz-stable` |
| **Linux (RHEL/Fedora)** | `sudo rpm --import https://releases.warp.dev/linux/keys/warp.asc && sudo dnf install oz-stable` |
| **Linux (Arch)** | `sudo pacman -Sy oz-stable` |

### Linux Repo Setup (Debian/Ubuntu)

```bash
sudo apt-get install wget gpg
wget -qO- https://releases.warp.dev/linux/keys/warp.asc | gpg --dearmor > warpdotdev.gpg
sudo install -D -o root -g root -m 644 warpdotdev.gpg /etc/apt/keyrings/warpdotdev.gpg
sudo sh -c 'echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/warpdotdev.gpg] https://releases.warp.dev/linux/deb stable main" > /etc/apt/sources.list.d/warpdotdev.list'
rm warpdotdev.gpg
sudo apt update && sudo apt install oz-stable
```

## Authentication

```bash
# Headless/VM (preferred)
export WARP_API_KEY="***"

# Interactive (local machines — opens browser)
oz login
```

## Verify Installation

```bash
which oz              # locate the binary
oz whoami             # confirm auth
oz model list         # confirm API connectivity
```

---

## Local Agent Runs

```bash
# Basic one-shot
oz agent run --prompt 'Add retry logic to API calls and update tests' --cwd /path/to/project

# With model override
oz agent run --prompt 'Refactor auth module' --model claude-4-6-sonnet-high --cwd /path/to/project

# With skill as base prompt
oz agent run --skill 'owner/repo:skill-name' --prompt 'additional context' --cwd /path/to/project

# With MCP server
oz agent run --prompt 'Check GitHub issues' --mcp mcp-config.json --cwd /path/to/project
```

### Common Flags

| Flag | Description |
|------|-------------|
| `--prompt <TEXT>` (`-p`) | The task prompt |
| `--cwd <PATH>` (`-C`) | Working directory |
| `--name <NAME>` (`-n`) | Label for grouping/traceability |
| `--model <MODEL_ID>` | Override default model |
| `--skill <SPEC>` | Use skill as base prompt |
| `--mcp <SPEC>` | MCP server config; repeatable |
| `--environment <ID>` (`-e`) | Run in specific cloud environment |
| `--conversation <ID>` | Continue existing conversation |
| `--share [<RECIPIENTS>]` | Share session (`team:view`, `team:edit`, etc.) |

---

## Background Execution & Monitoring

**Oz does NOT support streaming.** It prints the Run ID immediately, then blocks until completion.

### Run in background

```bash
oz agent run --name 'fix-auth' --prompt 'Implement full auth flow with tests' --cwd /path/to/project &
OZ_PID=$!
```

### Poll for completion

```bash
wait $OZ_PID
echo "Exit code: $?"
```

### Monitor via API

```bash
RUN_ID="019dc700-abcd-1234-5678-abcdef123456"

while true; do
  STATE=$(curl -s "https://app.warp.dev/api/v1/agent/runs/$RUN_ID" \
    -H "Authorization: Bearer $WARP_API_KEY" | jq -r '.state')
  echo "$(date +%H:%M:%S) State: $STATE"
  case "$STATE" in
    SUCCEEDED|FAILED|ERROR|CANCELLED) break ;;
  esac
  sleep 10
done
echo "Final state: $STATE"
```

### Cancel a run

```bash
# Via API (preferred — actually stops the remote agent)
curl -X POST "https://app.warp.dev/api/v1/agent/runs/$RUN_ID/cancel" \
  -H "Authorization: Bearer $WARP_API_KEY"

# Via CLI (kills local process only, remote may continue)
kill $OZ_PID
```

---

## Cloud Agent Runs

```bash
# Basic cloud run
oz agent run-cloud --environment ENV_ID --prompt 'Summarize this repo and list risky areas'

# Cloud run with computer use
oz agent run-cloud --environment ENV_ID --computer-use --prompt 'Visually test the login flow end-to-end'
```

### Cloud run flags

| Flag | Description |
|------|-------------|
| `--environment <ID>` (`-e`) | Cloud environment to run in |
| `--host <WORKER_ID>` | Run on specific self-hosted worker |
| `--attach <PATH>` | Attach file (max 5, repeatable) |
| `--computer-use` | Enable visual GUI interaction |
| `--open` | View session in Warp desktop |
| *(also supports `--name`, `--mcp`, `--model`, `--skill`, `--file`, `--prompt`)* | |

### Cloud vs Local Decision

| Scenario | Recommendation |
|----------|---------------|
| Quick bounded task, files already local | `oz agent run` |
| Heavy computation, don't tax the machine | `oz agent run-cloud` |
| Need repo isolation, consistent toolchain | `oz agent run-cloud` with environment |
| Recurring scheduled tasks | `oz schedule create` |
| Event-driven (Slack, Linear, GitHub) | Cloud triggers (Oz web app) |

> ⚠️ Cloud runs do NOT support `--cwd`, `--share`, or `--profile`. Always use `--environment` for code tasks.

---

## Parallel Work Pattern

```bash
oz agent run --name 'fix-auth' --prompt 'Fix auth bug in login flow' --cwd /path/to/project &
PID1=$!

oz agent run --name 'add-tests' --prompt 'Add unit tests for parser module' --cwd /path/to/project &
PID2=$!

wait $PID1 $PID2
echo "All runs complete"
```

---

## Scheduled Agents (Cron)

```bash
oz schedule create \
  --name "Weekly Cleanup" \
  --cron "0 10 * * 1" \
  --prompt "Scan for stale feature flags and remove any that are no longer referenced. Open a PR." \
  --environment "ENV_ID"

oz schedule list
oz schedule get SCHEDULE_ID
oz schedule pause SCHEDULE_ID
oz schedule unpause SCHEDULE_ID
oz schedule update SCHEDULE_ID --prompt "new prompt"
oz schedule delete SCHEDULE_ID
```

---

## REST API

Full HTTP API at `https://app.warp.dev/api/v1` with Bearer auth.

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/agent/runs` | Create a run |
| `GET` | `/agent/runs/{runId}` | Get run details + state |
| `POST` | `/agent/runs/{runId}/cancel` | Cancel a run |
| `GET` | `/agent/runs` | List runs (paginated, filterable) |
| `GET` | `/agent/models` | List available models |

### Run States

`QUEUED` → `PENDING` → `CLAIMED` → `INPROGRESS` → `SUCCEEDED` / `FAILED` / `ERROR` / `CANCELLED` / `BLOCKED`

---

## MCP Server Integration

```bash
oz agent run --prompt 'Query Linear for open bugs' --mcp mcp-config.json --cwd /path/to/project
```

See `references/mcp-config-example.json` for a full example config.

---

## Additional CLI Commands

| Command | Description |
|---------|-------------|
| `oz whoami` | Show authenticated user, team, email |
| `oz model list` | List available LLM models |
| `oz run list` | List recent agent runs |
| `oz environment list` | List cloud environments |
| `oz integration list` | List configured integrations |
| `oz secret list` | List stored secrets |
| `oz mcp list` | List MCP server configs |
| `oz schedule list` | List scheduled agents |

> `--output-format json` may wrap output in UI frames rather than raw JSON. Test before piping.

---

## Procedure

1. Verify: `oz whoami` and `oz model list`
2. For bounded local tasks, use `oz agent run` directly
3. For longer tasks, run in background with `&` and monitor via API
4. For cloud tasks, always specify `--environment ENV_ID`
5. Summarize outcomes (files changed, test results, remaining risks)

## Pitfalls

- Oz does NOT stream output — it blocks until completion. Use background execution for anything non-trivial.
- Killing the local CLI process does NOT cancel remote/cloud runs. Use the API cancel endpoint.
- `--output-format json` may not produce parseable JSON.
- Cloud runs without `--environment` have no repo access.
- `oz login` requires a browser — doesn't work on headless/VM. Use `WARP_API_KEY`.
- There is no `--version` flag. Use `which oz` to verify.
- Credits can exhaust silently — check `oz run list` for recent failures.

## Verification

Smoke test (requires credits):

```bash
oz agent run --prompt 'Respond with exactly: OZ_SMOKE_OK'
```

Non-destructive (no credits):

```bash
oz whoami
oz model list
oz run list
```

## Rules

1. Prefer `oz agent run` for local one-shot tasks.
2. Always scope Oz runs to a specific `--cwd` (local) or `--environment` (cloud).
3. Cancel remote runs via API, not by killing the CLI process.
4. Report concrete outcomes (files changed, test results, remaining risks).
5. For recurring work, use `oz schedule create`.

## External Endpoints

| Endpoint | Method | Data Sent | Purpose |
|----------|--------|-----------|---------|
| `https://app.warp.dev/api/v1/agent/runs` | POST | Prompt, model, environment config | Create agent runs |
| `https://app.warp.dev/api/v1/agent/runs/{runId}` | GET | API key (Bearer header) | Poll run status |
| `https://app.warp.dev/api/v1/agent/runs/{runId}/cancel` | POST | API key (Bearer header) | Cancel a run |
| `https://app.warp.dev/api/v1/agent/models` | GET | API key (Bearer header) | List available models |
| `https://app.warp.dev/api/v1/agent/runs` | GET | API key (Bearer header) | List past runs |

All API calls use Bearer token auth via `WARP_API_KEY`. No data is sent to any third party beyond Warp's API.

## Security & Privacy

- **What leaves the machine:** Task prompts, file contents (via `--attach`), and repo code (via cloud environments) are sent to Warp's API for processing by LLM models.
- **What stays local:** Local `oz agent run` executes on your machine. No telemetry or usage data is sent beyond the API calls shown above.
- **Credential handling:** `WARP_API_KEY` is stored in environment variables — never hardcoded in skill files.
- **Cloud environments:** Repo code and environment secrets are accessible to the cloud agent during execution. Ensure environment secrets are scoped appropriately.

### Trust Statement

By using this skill, your prompts and code are sent to Warp.dev's API for processing. Only install if you trust Warp. Cloud runs execute your code on Warp's infrastructure. Review Warp's privacy policy at https://warp.dev/privacy.


