# News Agent 1.2.1 — Pipeline Runner

When invoked, Claude Code executes the following steps directly:

## Step 0: Load config and cache

Read `news agent 1.2.1/News Agent Config.md` for:
- `topics` (1–4 from: AI & tech, World affairs, Science, Business, Politics, Climate)
- `stories_per_topic` (1–5)
- `summary_length` (1–4 sentences)

Read `.news-cache.json` from the News folder. TTL is 6 hours.

## Step 1: Gather headlines (Pass 1)

For each topic NOT in cache or expired:
- Call `WebSearch` with query: `"<topic> news today <Month Year>"`
- Parse results into `{ topic, headline, snippet, url, source }`
- Save to cache with `{ timestamp: Date.now(), headlines: [...] }`

For cached topics: reuse headlines, trim to `stories_per_topic`.

**One WebSearch per uncached topic. No extra calls.**

Save updated `.news-cache.json`.

## Step 2: Filter

Drop items that are:
- Listicles ("top 10…", "5 ways…")
- Promotional or sponsored
- Duplicates
- Clickbait with no substance
- Clearly off-topic

Keep genuinely newsworthy items. If unsure, keep it.

## Step 3: Rank and summarize

Rank all remaining items by global importance. Write exactly `summary_length` sentence(s) per item — tight, informative, no filler. Preserve url and source.

## Step 4: Output

Write to `MM-DD-YYYY HH.MM.SS.md` in the News folder (e.g., `05-29-2026 09.07.27.md`).

**CRITICAL: Every story MUST include a source link.** Format:

```markdown
# News Brief — <Date>

---

## <Topic Name>

**1. Headline**
Summary text.
*Source: [Publication Name](url)*

**2. Headline**
Summary text.
*Source: [Publication Name](url)*
```

- The source line uses italic, in the format `*Source: [Name](URL)*`
- Every single story gets its own source link. Never skip this.
- The filename is always `MM-DD-YYYY HH.MM.SS.md` — month-day-year with hyphens, then a space, then hours.minutes.seconds.
