# News Agent 1.2.1

Claude Code executes the pipeline natively — WebSearch for gathering, built-in reasoning for filtering and ranking. No API key, no external runtime.

## How it works

| Stage | Engine | What it does |
|-------|--------|---------------|
| Pass 1 | Claude WebSearch | One search per uncached topic, gathers headlines |
| Filter | Claude reasoning | Drops clickbait, listicles, off-topic, low-quality |
| Pass 2 | Claude reasoning | Ranks by global importance, writes summaries |

## Why 1.2.1

v1.2 was the initial Claude Code-native rewrite. v1.2.1 hardens two requirements that caused output drift:

- **Source links are mandatory** — every story in the output file must end with `*Source: [Publication](URL)*`. v1.2 specified this in the template but didn't enforce it explicitly enough.
- **Filename format is strict** — output files must be named `MM-DD-YYYY HH.MM.SS.md`. No other format (e.g., `YYYY-MM-DD News.md`) is accepted.

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and configured
- Nothing else — no API keys, no Node.js, no external runtime

## Setup

**1. Copy the agent folder.** Clone the repo or copy the `news agent 1.2.1/` folder to wherever you want it. If you use Obsidian, drop it right into your vault.

**2. Open the config.** Edit `News Agent Config.md` frontmatter:

```yaml
---
topics:                          # pick 1–4 from the available list
  - AI & tech
  - World affairs
  - Business
stories_per_topic: 5             # 1–5
summary_length: 2                # 1–4 sentences per story
output_path: News                # folder for timestamped .md briefings
---
```

- **Obsidian users:** set `output_path` to a vault-relative path, e.g. `Market/News` or `Daily Briefings`.
- **Everyone else:** use any local path, e.g. `/Users/you/news-output` or `./briefings`.

The cache file (`.news-cache.json`) is written inside `output_path` automatically.

**3. Run it.** Open Claude Code in the directory containing the `news agent 1.2.1/` folder, then say:

```
run the news agent 1.2.1 pipeline
```

Claude Code reads the protocol, loads your config, WebSearches any uncached topics, filters, ranks, writes summaries with source links, and saves the briefing as `MM-DD-YYYY HH.MM.SS.md` in your `output_path`.

That's it. One sentence, no scripts.

## Daily automation (optional)

Schedule it once and get a briefing every morning — no external cron, no launchd. From within Claude Code:

```
schedule the news agent 1.2.1 to run every day at 7:57 AM
```

Claude Code sets up a durable `CronCreate` job that survives restarts. Or do it manually:

```
Cron:     57 7 * * *    (7:57 AM local, off-peak minute)
Durable:  true
Prompt:   "Run the news agent 1.2.1 pipeline: read config, check cache,
          WebSearch uncached topics (one per topic, no extras), filter,
          rank, write summaries with source links, and save output to
          MM-DD-YYYY HH.MM.SS.md."
```

Check your latest briefing:
```bash
ls -lt <output_path>/*.md | head -3
```

## Files

```
news agent 1.2.1/
  README.md              ← this file
  protocol.md            ← full pipeline spec that Claude Code follows
  news-agent.prompt.md   ← concise prompt template for the pipeline
  News Agent Config.md   ← your settings (topics, stories, output path)
  cron.md                ← cron setup reference
  com.news-agent.cron.md ← cron job definition
```

## v1.2 → v1.2.1

| | v1.2 | v1.2.1 |
|---|---|---|
| Source links | Specified in template but easily missed | **Mandatory** — enforced in prompt + protocol |
| Filename format | `MM-DD-YYYY HH.MM.SS.md` (stated) | `MM-DD-YYYY HH.MM.SS.md` (**strict**, with example) |
| Output path | Hardcoded to News folder | Configurable via `output_path` in frontmatter |
