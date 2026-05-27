# News Agent

Daily news briefing pipeline. Haiku gathers and filters headlines, Sonnet ranks and writes summaries.

| Version | How it runs | Cache | Output |
|---------|-------------|-------|--------|
| [v1.0](v1.0/) | Browser HTML app | Composite key (topics + count), localStorage | In-browser |
| [v1.1](v1.1/) | Terminal CLI (Node.js) | Per-topic, JSON file | Obsidian vault Markdown |

## Quick start (v1.1)

```bash
export ANTHROPIC_API_KEY="sk-ant-…"
node v1.1/news-agent.js
```

Config via `News Agent Config.md` frontmatter in your Obsidian vault. Runs daily via launchd at 8:00 AM.
