# Cron — Daily 7:57 AM

Claude Code durable cron. Fires daily, survives restarts.

## Setup

Run once in Claude Code:

```
/loop 1440m run the news agent 1.2.1 pipeline
```

Or schedule via CronCreate:

- **Cron:** `57 7 * * *` (7:57 AM local, off-peak minute)
- **Durable:** true
- **Prompt:** "Run the news agent 1.2.1 pipeline: read config from news agent 1.2.1/News Agent Config.md, check .news-cache.json for cache, WebSearch any uncached topics (one per topic, no extras), filter, rank, write summaries with source links, and save output to MM-DD-YYYY HH.MM.SS.md."

## Status checks

```bash
# List active cron jobs
/loop status

# Check latest briefing
ls -lt sync/Market/News/*.md | head -3
```

## Logs

Pipeline logs written inline during Claude Code execution. Output files timestamped in the News folder.
