---
name: warp-oz
description: Delegate coding tasks to Warp's Oz CLI agent for feature implementation, refactoring, PR review, and cloud execution. Requires the oz CLI installed and authenticated.
version: 3.2.0
author: Hermes Agent
license: MIT
homepage: https://github.com/dbardi/agent-skills/tree/main/hermes/warp-oz
metadata:
  hermes:
    tags: [Coding-Agent, Warp, Oz, Autonomous, Refactoring, Cloud-Agent, Scheduling]
    related_skills: [opencode, claude-code, codex]
---

# Warp Oz CLI

Use [Warp Oz](https://docs.warp.dev/reference/cli/cli) as an autonomous coding worker orchestrated by Hermes terminal/process tools. Oz is Warp.dev's agent orchestration platform — local or cloud execution, scheduling, API control, and MCP integration.

> **Note:** `warp-cli` is deprecated. All commands use `oz`.

## When to Use

- User explicitly asks to use Oz
- You want an external coding agent to implement/refactor/review code
- Cloud execution preferred (isolated runners, no local resource drain)
- Scheduled/recurring agent tasks
- Integration with MCP servers, triggers (Slack, Linear, GitHub)

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
| **Linux (RHEL/Fedora)** | `sudo dnf install oz-stable` |
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

### Linux Repo Setup (RHEL/Fedora)

```bash
sudo rpm --import https://releases.warp.dev/linux/keys/warp.asc
sudo dnf install oz-stable
```

## Authentication

```bash
# Headless/VM (preferred for Hermes)
export WARP_API_KEY="***"

# Interactive (local machines only — opens browser)
oz login
```

## Verify Installation

```
terminal(command="which oz")
terminal(command="oz whoami")
terminal(command="oz model list")
```

---

## One-Shot Local Runs

Use `oz agent run` for bounded, non-interactive tasks:

```
terminal(command="oz agent run --prompt 'Add retry logic to API calls and update tests' --cwd /path/to/project", timeout=300)
```

### Common Flags

| Flag | Description |
|------|-------------|
| `--prompt <TEXT>` (`-p`) | The task prompt |
| `--cwd <PATH>` (`-C`) | Working directory |
| `--name <NAME>` (`-n`) | Label for grouping/traceability |
| `--model <MODEL_ID>` | Override default model |
| `--skill <SPEC>` | Use skill as base prompt (`repo:skill_name`) |
| `--mcp <SPEC>` | MCP server config (UUID, JSON path, or inline JSON); repeatable |
| `--environment <ID>` (`-e`) | Run in specific cloud environment |
| `--conversation <ID>` | Continue existing conversation |
| `--share [<RECIPIENTS>]` | Share session (`team:view`, `team:edit`, `user@email:view`) |

---

## Background Execution & Monitoring

**Oz does NOT support streaming.** It prints the Run ID immediately, then blocks until completion. Use Hermes background process tools to handle this.

### Local runs — fire and get notified

```
terminal(command="oz agent run --name 'fix-auth' --prompt 'Implement full auth flow with tests' --cwd /path/to/project", background=true, notify_on_complete=true)
```

When notified, fetch the output:

```
process(action="log", session_id="<id>")
```

### Local runs — watch for errors mid-execution

```
terminal(command="oz agent run --name 'refactor-api' --prompt 'Refactor the API module for better error handling' --cwd /path/to/project", background=true, watch_patterns=["ERROR", "FAILED", "Traceback", "error:"], notify_on_complete=true)
```

If a watch pattern fires, you'll be notified immediately and can inspect:

```
process(action="log", session_id="<id>")
```

### Cloud runs — fire and poll via API

Cloud runs (`oz agent run-cloud`) may take longer and are better monitored via the API:

```
# Start the run — capture the Run ID from output
terminal(command="oz agent run-cloud --environment ENV_ID --prompt 'Review auth module for security issues'", background=true, notify_on_complete=true)
```

After completion, check the result:

```
process(action="log", session_id="<id>")
```

For detailed status polling during long cloud runs:

```
terminal(command="oz run list --limit 5")
```

### Cancel a run

```
# Local — kill the CLI process
process(action="kill", session_id="<id>")

# Remote — cancel via API (preferred, actually stops the agent)
terminal(command="curl -s -X POST \"https://app.warp.dev/api/v1/agent/runs/$RUN_ID/cancel\" -H \"Authorization: Bearer $WARP_API_KEY\"")
```

> ⚠️ Killing the local CLI process does NOT stop the remote agent. Use the API cancel endpoint for cloud runs.

---

## Cloud Agent Runs

Use `oz agent run-cloud` to run on Warp's remote runners (isolated, no local resource drain):

```
terminal(command="oz agent run-cloud --environment ENV_ID --prompt 'Summarize this repo and list risky areas'", background=true, notify_on_complete=true)
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

> ⚠️ Cloud runs do NOT support `--cwd`, `--share`, or `--profile`. Always use `--environment` for code tasks — bare cloud runs have no repo access.

---

## Parallel Work Pattern

Use separate `terminal()` calls for independent tasks:

```
terminal(command="oz agent run --name 'fix-auth' --prompt 'Fix auth bug in login flow' --cwd /path/to/project", background=true, notify_on_complete=true)
terminal(command="oz agent run --name 'add-tests' --prompt 'Add unit tests for parser module' --cwd /path/to/project", background=true, notify_on_complete=true)
process(action="list")
```

For cloud runs, same pattern — they execute independently on Warp's infrastructure.

---

## Scheduled Agents (Cron)

```
terminal(command="oz schedule create --name 'Weekly Cleanup' --cron '0 10 * * 1' --prompt 'Scan for stale feature flags and remove unreferenced ones. Open a PR.' --environment ENV_ID")
terminal(command="oz schedule list")
terminal(command="oz schedule get SCHEDULE_ID")
terminal(command="oz schedule pause SCHEDULE_ID")
terminal(command="oz schedule unpause SCHEDULE_ID")
terminal(command="oz schedule delete SCHEDULE_ID")
```

---

## MCP Server Integration

```
terminal(command="oz agent run --prompt 'Query Linear for open bugs' --mcp mcp-config.json --cwd /path/to/project", timeout=300)
```

See `references/mcp-config-example.json` for a full example config.

---

## REST API

Full HTTP API at `https://app.warp.dev/api/v1` with Bearer auth. Useful for programmatic control when CLI isn't sufficient.

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

## Additional CLI Commands

```
terminal(command="oz whoami")              # Auth status
terminal(command="oz model list")          # Available models
terminal(command="oz run list")            # Recent runs + status
terminal(command="oz environment list")    # Cloud environments
terminal(command="oz integration list")    # Connected integrations
terminal(command="oz secret list")         # Stored secrets
terminal(command="oz mcp list")            # MCP server configs
terminal(command="oz schedule list")       # Scheduled agents
```

> `--output-format json` may wrap output in UI frames rather than raw JSON. Test before piping.

---

## Skills as Agents

```
# Local
terminal(command="oz agent run --skill 'owner/repo:skill-name' --prompt 'additional context' --cwd /path/to/project", timeout=300)

# Cloud
terminal(command="oz agent run-cloud --environment ENV_ID --skill 'owner/repo:skill-name' --prompt 'additional context'", background=true, notify_on_complete=true)

# Scheduled
terminal(command="oz schedule create --name 'Weekly Cleanup' --cron '0 10 * * 1' --skill 'owner/repo:cleanup' --environment ENV_ID")
```

---

## Procedure

1. Verify tool readiness:
   - `terminal(command="oz whoami")`
   - `terminal(command="oz model list")`
2. For bounded local tasks, use `oz agent run` with appropriate timeout.
3. For longer tasks, use `background=true` + `notify_on_complete=true`.
4. For error-sensitive tasks, add `watch_patterns=["ERROR", "FAILED", "Traceback"]`.
5. After completion, fetch output with `process(action="log")`.
6. Summarize file changes, test results, and next steps back to user.

## Pitfalls

- Oz does NOT stream output — it blocks until completion. Always use `background=true` + `notify_on_complete=true` for anything that might take more than a few seconds.
- Killing the local CLI process does NOT cancel remote/cloud runs. Use the API cancel endpoint.
- `--output-format json` may not produce parseable JSON — test before relying on it.
- Cloud runs without `--environment` execute in a bare sandbox with no repo access.
- `oz login` requires a browser — doesn't work on headless/VM. Use `WARP_API_KEY`.
- There is no `--version` flag. Use `which oz` to verify installation.
- Credits can exhaust silently — check `oz run list` for recent failures.

## Verification

Smoke test (requires credits):

```
terminal(command="oz agent run --prompt 'Respond with exactly: OZ_SMOKE_OK'", timeout=60)
```

Non-destructive (no credits):

```
terminal(command="oz whoami")
terminal(command="oz model list")
terminal(command="oz run list")
```

Success criteria:
- `oz whoami` shows user/team/email
- `oz model list` returns model list
- For code tasks: expected files changed and tests pass

## Rules

1. Prefer `oz agent run` for local one-shot tasks — simplest path.
2. Always use `background=true` + `notify_on_complete=true` for tasks that may take more than a few seconds.
3. Add `watch_patterns` for error-sensitive tasks to catch failures early.
4. Always scope Oz runs to a specific `--cwd` (local) or `--environment` (cloud).
5. Cancel remote runs via API, not by killing the CLI process.
6. Report concrete outcomes (files changed, test results, remaining risks).
7. For recurring work, use `oz schedule create` instead of manual cron wrappers.


