# 🚀 Warp Oz Skill

![Skill](https://img.shields.io/badge/skill-warp__oz-blue)
![Version](https://img.shields.io/badge/version-3.2.0-green)
![License](https://img.shields.io/badge/license-MIT-brightgreen)

A comprehensive skill for orchestrating [Warp Oz](https://docs.warp.dev/reference/cli/cli) as an autonomous coding agent — local or cloud execution, scheduling, API control, MCP integration, and background monitoring.

Compatible with [Hermes Agent](https://github.com/NousResearch/hermes-agent) and [ClawHub](https://clawhub.ai).

## Features

- **Local agent runs** — bounded one-shot tasks with `oz agent run`
- **Cloud agent runs** — isolated execution on Warp's infrastructure
- **Background monitoring** — `notify_on_complete` + `watch_patterns` for non-streaming CLI
- **Parallel work** — multiple independent Oz tasks simultaneously
- **Scheduled agents** — recurring cron-based tasks via `oz schedule`
- **REST API** — programmatic control when CLI isn't sufficient
- **MCP integration** — connect Oz to external tools (GitHub, Linear, etc.)
- **Skills as agents** — reusable instruction sets for agent behavior

## Installation

### Hermes Agent

```bash
hermes skills install dbardi/hermes-skill-warp-oz
```

Or clone into your skills directory:

```bash
git clone https://github.com/dbardi/hermes-skill-warp-oz ~/.hermes/skills/autonomous-ai-agents/warp-oz
```

### ClawHub

Install from the [ClawHub marketplace](https://clawhub.ai).

## Prerequisites

- [Oz CLI](https://docs.warp.dev/reference/cli/cli) installed (`oz`)
- Warp account with API access
- `WARP_API_KEY` environment variable set

## Quick Start

```bash
# Verify installation
oz whoami

# One-shot local task
oz agent run --prompt "Add retry logic to API calls" --cwd /path/to/project

# Cloud run with notification (Hermes)
terminal(command="oz agent run-cloud --environment ENV_ID --prompt 'Fix auth bug'", background=true, notify_on_complete=true)
```

## Platforms

| Platform | Install |
|----------|---------|
| macOS | `brew tap warpdotdev/warp && brew install --cask oz` |
| Windows | `winget install Warp.Warp` |
| Linux (Debian/Ubuntu) | See SKILL.md for repo setup |
| Linux (RHEL/Fedora) | `sudo rpm --import https://releases.warp.dev/linux/keys/warp.asc && sudo dnf install oz-stable` |
| Linux (Arch) | `sudo pacman -Sy oz-stable` |

## External Endpoints

This skill communicates exclusively with Warp's API:

| Endpoint | Purpose |
|----------|---------|
| `https://app.warp.dev/api/v1/agent/runs` | Create/poll agent runs |
| `https://app.warp.dev/api/v1/agent/models` | List models |
| `https://app.warp.dev/api/v1/agent/runs/{runId}/cancel` | Cancel runs |

No data is sent to any third party beyond Warp's API.

## Security & Privacy

- Task prompts and code are sent to Warp.dev for LLM processing
- `WARP_API_KEY` stored in environment variables — never hardcoded
- Cloud runs execute on Warp's infrastructure
- Review Warp's privacy policy: https://warp.dev/privacy

## License

MIT — see [LICENSE](LICENSE) for details.
