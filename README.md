# News Agent 1.0

Daily news briefing agent with a two-pass pipeline — Haiku gathers and filters headlines, Sonnet ranks and writes summaries. Runs entirely in the browser with no backend.

## How it works

| Pass | Model | What it does |
|------|-------|---------------|
| Pass 1 | Haiku + web search | Searches for the most recent headlines across your chosen topics |
| Filter | Haiku | Scores items for quality, drops clickbait, listicles, and off-topic noise |
| Pass 2 | Sonnet | Ranks by global importance and writes concise summaries |

## Usage

1. Get an API key from [console.anthropic.com](https://console.anthropic.com)
2. Open `news-agent.html` in a browser
3. Paste your key, pick topics, set sliders, and run

Your key never leaves the browser — all API calls go directly to `api.anthropic.com`.

## Configuration

- **Topics:** AI & tech, World affairs, Science, Business, Politics, Climate (up to 4)
- **Stories per topic:** 1–5
- **Summary length:** 1–4 sentences

## Cache

Results from Pass 1 are cached in localStorage for 6 hours. Changing topics or story count invalidates the cache for that combination. A cache hit skips the search call entirely, saving both time and tokens.

## Obsidian integration

The `news agent.md` file documents how the pipeline integrates with Obsidian — config via frontmatter, per-topic JSON cache, and markdown output files auto-saved to your vault.

## Models

- **Haiku:** `claude-haiku-4-5-20251001`
- **Sonnet:** `claude-sonnet-4-20250514`
