# News Agent 1.2.1 — Protocol

Claude Code executes every step directly. No external API key, no Node.js runtime, no Anthropic SDK.

## Pipeline

### 0. Load config and cache

Read `News Agent Config.md` for topics, `stories_per_topic`, `summary_length`.
Read `.news-cache.json` for cached headlines per topic.
TTL: 6 hours. Cache key: topic name.

### 1. Pass 1 — Gather (WebSearch, one per topic)

For each topic NOT in cache or expired:
- Call `WebSearch` with query: `"<topic> news today <Month Year>"`
- Parse results into headlines with: topic, headline, snippet, url, source
- Update cache entry with `{ timestamp: Date.now(), headlines: [...] }`

For cached + fresh topics:
- Merge all headlines
- Trim each topic to `stories_per_topic` items
- Save updated `.news-cache.json`

**Rule:** one WebSearch call per uncached topic. No extra searches.

### 2. Filter — Drop low quality

Review all gathered headlines. Drop:
- Listicles ("top 10…", "5 ways…")
- Promotional or sponsored content
- Duplicates
- Clickbait with no substance
- Off-topic items

Keep only genuinely newsworthy items. If err, keep it.

### 3. Pass 2 — Rank and summarize

Rank all remaining items by global importance and newsworthiness. Write exactly `summary_length` sentence(s) per item — tight, informative, no filler. Preserve url and source.

### 4. Output

Write to **`MM-DD-YYYY HH.MM.SS.md`** in the News folder. This format is strict — month-day-year with hyphens, then a space, then hours.minutes.seconds. Example: `05-29-2026 09.07.27.md`.

**Every story must carry its own source link.** Format:

```markdown
# News Brief — <Date>

---

## <Topic Name>

**1. Headline**
Summary text.
*Source: [Publication Name](url)*
```

- Source line: italic, format `*Source: [Name](URL)*`
- One source link per story, no exceptions

## Config reference

Available topics: `AI & tech`, `World affairs`, `Science`, `Business`, `Politics`, `Climate`
`stories_per_topic`: 1–5
`summary_length`: 1–4 sentences

## Cache schema (`.news-cache.json`)

```json
{
  "topics": {
    "<topic>": {
      "timestamp": <epoch ms>,
      "headlines": [
        { "topic": "...", "headline": "...", "snippet": "...", "url": "...", "source": "..." }
      ]
    }
  }
}
```

## v1.2.1 changelog

- **Source links mandatory:** every story in the output file must include a `*Source: [Name](URL)*` line
- **Filename format enforced:** `MM-DD-YYYY HH.MM.SS.md` is strict — no other format accepted
- **Output format clarified:** topic-grouped sections with numbered stories, each ending with its source link
