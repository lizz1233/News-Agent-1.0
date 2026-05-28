# News Agent

Daily news briefing pipeline — gathers headlines, filters for quality, ranks by importance, writes summaries.

| Version | Runtime | Key needed | Web search | Cron |
|---------|---------|------------|------------|------|
| [v1.0](v1.0/) | Browser HTML | Paste in-browser | Haiku + Anthropic tool | None |
| [v1.1](v1.1/) | Node.js CLI | `ANTHROPIC_API_KEY` env | Haiku + Anthropic tool | launchd (8 AM) |
| [v1.2](v1.2/) | Claude Code native | **None** | Claude WebSearch | Claude Code cron (7:57 AM) |

## Quick start

**v1.1 (terminal):**
```bash
export ANTHROPIC_API_KEY="sk-ant-…"
node v1.1/news-agent.js
```

**v1.2 (no key):** Ask Claude Code to run it — reads config from your Obsidian vault, searches the web, and writes the briefing.

## Setup

Config lives in `News Agent Config.md` frontmatter in your Obsidian vault:

```yaml
topics: [AI & tech, World affairs, Business]
stories_per_topic: 3
summary_length: 2
```

## Cache

Per-topic, 6-hour TTL, stored in `.news-cache.json`. All versions share the same cache — adding a topic only searches the new one.
