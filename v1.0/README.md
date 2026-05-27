# News Agent 1.1

Terminal-based daily news briefing pipeline. Searches headlines via the Anthropic API, filters for quality, ranks by global importance, and writes a Markdown briefing to your Obsidian vault.

## How it works

| Stage | Model | What it does |
|-------|-------|---------------|
| Pass 1 | Haiku + web search | Searches one API call per topic for the most recent headlines |
| Filter | Haiku | Drops clickbait, listicles, off-topic, and low-quality items |
| Pass 2 | Sonnet | Ranks by global importance and writes concise summaries |

Each topic is cached independently for 6 hours — adding a topic only searches the new one.

## Setup

```bash
export ANTHROPIC_API_KEY="sk-ant-…"
```

Optional — set a custom Obsidian path:

```bash
export NEWS_AGENT_PATH="/path/to/your/vault/sync/Market/News"
```

## Usage

```bash
node news-agent.js           # Full run → writes MM-DD-YYYY HH.MM.SS.md
node news-agent.js --dry-run # Print to stdout, don't write a file
node news-agent.js --clear-cache  # Clear the per-topic cache
```

## Configuration

Edit `News Agent Config.md` in your Obsidian vault:

```yaml
---
topics:
  - AI & tech
  - World affairs
  - Business
stories_per_topic: 3
summary_length: 2
---
```

### Available topics

`AI & tech`, `World affairs`, `Science`, `Business`, `Politics`, `Climate` (up to 4)

### Settings

- **stories_per_topic:** 1–5
- **summary_length:** 1–4 sentences

## Cache

Per-topic results are cached in `.news-cache.json` alongside your config. TTL is 6 hours. Cache hits skip the API call entirely — you only pay for topics you add or that have expired.

## Output

Each run writes a timestamped Markdown file to your Obsidian News folder:

```
sync/Market/News/
  News Agent Config.md    ← your config
  .news-cache.json        ← per-topic cache
  05-27-2026 10.32.32.md  ← briefing output
```

## Models

- **Haiku:** `claude-haiku-4-5-20251001`
- **Sonnet:** `claude-sonnet-4-20250514`

## v1.0 → v1.1

v1.0 was a browser-based HTML app. v1.1 moves to the terminal — config lives in Obsidian frontmatter, cache is per-topic (not per config-combo), and output is written directly into the vault as Markdown.
