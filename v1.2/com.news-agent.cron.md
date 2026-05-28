# Daily Cron — 7:57 AM

Claude Code durable cron job. Fires every morning and triggers the news agent pipeline.

## Job definition

```
Cron:     57 7 * * *    (7:57 AM local, daily)
Type:     recurring
Durable:  true
Prompt:   "Run the news agent 1.2 pipeline: read config, check cache,
          WebSearch uncached topics (one per topic, no extras), filter,
          rank, and write output to timestamped .md file."
```

## Management

```bash
# In Claude Code:
/cron list                    # see all scheduled jobs
/cron delete <job-id>         # remove this cron

# Check if it fired today:
ls -lt sync/Market/News/*.md | head -1
```

## Note

Cron fires when Claude Code is running. For fully unattended daily runs even when Claude Code is closed, configure durable storage in `.claude/settings.json` with a `CronCreate` hook.
