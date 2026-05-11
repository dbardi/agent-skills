---
name: hermes-tweet
description: Use the native Hermes Tweet plugin for X/Twitter automation through Xquik. Search tweets, read tweet replies, look up users, monitor X trends, and run approval-gated account actions such as post tweets, post replies, DMs, follows, monitors, and webhooks.
version: 1.0.0
author: Xquik
license: MIT
homepage: https://github.com/Xquik-dev/hermes-tweet
metadata:
  hermes:
    tags: [Social-Media, Twitter, X, Tweet-Search, Hermes-Plugin, Xquik]
    related_skills: []
---

# Hermes Tweet

Use [Hermes Tweet](https://github.com/Xquik-dev/hermes-tweet) when a Hermes
Agent workflow needs structured X/Twitter tools for search, reads, monitoring,
or explicitly approved account actions.

## When To Use

- Search tweets from Hermes Agent.
- Read tweet replies and public tweet context.
- Look up users, account status, and X trends.
- Monitor tweets, keywords, and social signals.
- Prepare post tweets, post replies, DMs, follows, monitors, and webhooks behind
  a human approval step.

## Installation

Install the plugin through Hermes:

```bash
hermes plugins install Xquik-dev/hermes-tweet --enable
```

Or install the published package into the Hermes Python environment:

```bash
uv pip install --python ~/.hermes/hermes-agent/venv/bin/python hermes-tweet
hermes plugins enable hermes-tweet
```

Verify:

```bash
hermes tools list
```

## Configuration

Create an API key in Xquik and expose it to the Hermes runtime:

```bash
export XQUIK_API_KEY="xq_..."
export HERMES_TWEET_ENABLE_ACTIONS="false"
```

Keep `HERMES_TWEET_ENABLE_ACTIONS=false` for read-only research, monitoring,
and cron workflows. Set it to `true` only when the workflow has an explicit
human approval step for X account actions.

Do not paste API keys, X passwords, cookies, or login material into prompts or
tool arguments.

## Tool Model

| Tool | Purpose |
|------|---------|
| `tweet_explore` | Search the bundled endpoint catalog. No API call. |
| `tweet_read` | Call catalog-listed read-only endpoints. |
| `tweet_action` | Call write-like or private endpoints. Disabled by default. |

Use `tweet_explore` first. Use `tweet_read` for tweet search, replies, user
lookup, trends, monitors, and audits. Use `tweet_action` only after the user
approves the exact final action.

## Verification

Discovery probe:

```bash
hermes -z "Use tweet_explore to find tweet search endpoints. Do not call tweet_action." --toolsets hermes-tweet
```

Authenticated read probe:

```bash
hermes -z "Use tweet_explore, then read /api/v1/account. Do not call tweet_action." --toolsets hermes-tweet
```

Expected behavior:

- `tweet_explore` works without `XQUIK_API_KEY`.
- `tweet_read` works after `XQUIK_API_KEY` is configured.
- `tweet_action` stays hidden or disabled unless
  `HERMES_TWEET_ENABLE_ACTIONS=true`.

## Agent Workflow

1. Decide whether the request is read-only or action-taking.
2. Use `tweet_explore` to identify the right endpoint path.
3. Use `tweet_read` for searches, replies, user lookup, trends, and monitoring.
4. Summarize searched terms, endpoint paths, timestamps, and permission or
   rate-limit errors.
5. For post tweets, post replies, DMs, follows, monitor changes, or webhook
   changes, draft the exact action first.
6. Call `tweet_action` only after explicit approval.

## Example Prompts

```text
Use tweet_explore to find tweet search. Search X for recent posts about Hermes
Agent plugins and summarize recurring questions. Do not call tweet_action.
```

```text
Use tweet_explore to find the tweet reply read endpoint. Read replies for this
tweet URL and draft a response plan. Do not post anything.
```

```text
Use tweet_read to inspect the thread. Draft a reply for approval. Only call
tweet_action after I approve the exact final text.
```

## Pitfalls

- Long-running Hermes sessions may need reload or restart after `.env` changes.
- Missing API keys should expose only safe discovery behavior.
- Action endpoints are intentionally unavailable unless explicitly enabled.
- Do not use browser automation or cookie scraping as a fallback for this skill.

## References

- Guide: https://docs.xquik.com/guides/hermes-tweet
- Package: https://pypi.org/project/hermes-tweet/
- Repository: https://github.com/Xquik-dev/hermes-tweet
