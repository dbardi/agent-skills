# Agent Skills

A collection of agent skills for [Hermes](https://github.com/NousResearch/hermes-agent), [ClawHub](https://clawhub.ai), and other AI agent platforms.

## Structure

```
agent-skills/
├── hermes/          # Skills for Hermes Agent
│   └── warp-oz/     # Warp Oz CLI orchestration
└── openclaw/        # Skills for OpenClaw
```

## Install

### Hermes

```bash
hermes skills install dbardi/agent-skills/hermes/warp-oz
```

### Manual

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
| [warp-oz](hermes/warp-oz/) | Orchestrate Warp Oz as an autonomous coding agent | 3.2.0 |

## License

MIT
