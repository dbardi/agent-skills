# 🚀 Warp Oz Skill

![Skill](https://img.shields.io/badge/skill-warp__oz-blue)
![Version](https://img.shields.io/badge/version-1.0.0-green)
![License](https://img.shields.io/badge/license-MIT-brightgreen)

Orchestrate [Warp Oz](https://docs.warp.dev/reference/cli/cli) as an autonomous coding agent — local and cloud execution, scheduling, REST API, MCP integration, and background monitoring.

## Features

- **Local agent runs** — bounded one-shot tasks with `oz agent run`
- **Cloud agent runs** — isolated execution on Warp's infrastructure
- **Background monitoring** — non-streaming CLI with API polling or agent notification patterns
- **Parallel work** — multiple independent Oz tasks simultaneously
- **Scheduled agents** — recurring cron-based tasks via `oz schedule`
- **REST API** — programmatic control when CLI isn't sufficient
- **MCP integration** — connect Oz to external tools (GitHub, Linear, etc.)
- **Skills as agents** — reusable instruction sets for agent behavior

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

# Cloud run
oz agent run-cloud --environment ENV_ID --prompt "Fix auth bug"
```

## Platforms

| Platform | Install |
|----------|---------|
| macOS | `brew tap warpdotdev/warp && brew install --cask oz` |
| Windows | `winget install Warp.Warp` |
| Linux (Debian/Ubuntu) | See SKILL.md for repo setup |
| Linux (RHEL/Fedora) | `sudo rpm --import https://releases.warp.dev/linux/keys/warp.asc && sudo dnf install oz-stable` |
| Linux (Arch) | `sudo pacman -Sy oz-stable` |

## License

MIT — see [LICENSE](../../LICENSE) for details.
