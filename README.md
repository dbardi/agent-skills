# Agent Skills

A collection of agent skills for multiple AI agent platforms.

## Structure

```
agent-skills/
├── hermes/          # Skills for Hermes Agent
│   ├── hermes-tweet/ # X/Twitter automation through Hermes Tweet
│   └── warp-oz/      # Warp Oz CLI orchestration (Hermes-native patterns)
└── openclaw/        # Skills for OpenClaw / ClawHub
    └── warp-oz/     # Warp Oz CLI orchestration (generic patterns)
```

## Install

### Hermes

```bash
hermes skills install dbardi/agent-skills/hermes/hermes-tweet
hermes skills install dbardi/agent-skills/hermes/warp-oz
```

Or manually:

```bash
git clone https://github.com/dbardi/agent-skills.git
cp -r agent-skills/hermes/<skill-name> ~/.hermes/skills/<category>/<skill-name>
```

### ClawHub

Available on the [ClawHub marketplace](https://clawhub.ai).

## Skills

### Hermes

| Skill | Description | Version |
|-------|-------------|---------|
| [hermes-tweet](hermes/hermes-tweet/) | Search tweets, read replies, monitor X trends, and run approval-gated X actions through Hermes Tweet | 1.0.0 |
| [warp-oz](hermes/warp-oz/) | Orchestrate Warp Oz as an autonomous coding agent (Hermes-native) | 3.2.0 |

### OpenClaw / ClawHub

| Skill | Description | Version |
|-------|-------------|---------|
| [warp-oz](openclaw/warp-oz/) | Orchestrate Warp Oz as an autonomous coding agent (generic) | 1.0.0 |

## License

MIT
