# News Agent 1.2

Claude Code executes the pipeline natively — WebSearch for gathering, built-in reasoning for filtering and ranking. No API key, no external runtime.

## How it works

| Stage | Engine | What it does |
|-------|--------|---------------|
| Pass 1 | Claude WebSearch | One search per uncached topic, gathers headlines |
| Filter | Claude reasoning | Drops clickbait, listicles, off-topic, low-quality |
| Pass 2 | Claude reasoning | Ranks by global importance, writes summaries |

## Why 1.2

v1.1 required `ANTHROPIC_API_KEY` in your shell environment and called `api.anthropic.com` from Node.js. v1.2 uses Claude Code's own tools — the same engine that runs this conversation handles the entire pipeline. No key management, no Node.js dependency for the pipeline itself.

## Usage

Just ask Claude Code to run it:

```
run the news agent
```

Claude Code reads `protocol.md`, loads your config and cache, searches uncached topics, filters, ranks, and writes the briefing to your vault.

## Daily automation

A Claude Code cron fires daily at 8:00 AM (durable, survives restarts). It enqueues a prompt that triggers the pipeline.

```bash
# In Claude Code:
/loop 1440m run the news agent protocol from news agent 1.2
```

Or via `CronCreate`:
```
0 57 8 * * * → "run the news agent 1.2 pipeline for today"
```

## Configuration

Same `News Agent Config.md` as v1.1. Same cache at `.news-cache.json`. Same output format.

## Files

```
news agent 1.2/
  README.md     ← this file
  protocol.md   ← step-by-step pipeline for Claude Code
```

## v1.1 → v1.2

| | v1.1 | v1.2 |
|---|---|---|
| Engine | Node.js + Anthropic API | Claude Code native |
| Needs API key | Yes (env var) | No |
| Cron | launchd | Claude Code cron |
| Search | Haiku + web_search tool | Claude WebSearch |
| Filter/Rank | Haiku + Sonnet API calls | Claude reasoning |
