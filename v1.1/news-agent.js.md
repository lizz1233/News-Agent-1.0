# news-agent.js

```js
#!/usr/bin/env node
"use strict";

// ── Paths ──────────────────────────────────────────────────────────────────────
const os = require("os");
const fs = require("fs");
const path = require("path");

const OBSIDIAN_BASE =
  process.env.NEWS_AGENT_PATH ||
  path.join(os.homedir(), "Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian001/sync/Market/News");

const CONFIG_FILE = path.join(OBSIDIAN_BASE, "News Agent Config.md");
const CACHE_FILE = path.join(OBSIDIAN_BASE, ".news-cache.json");

// ── Constants ──────────────────────────────────────────────────────────────────
const CACHE_TTL_MS = 6 * 60 * 60 * 1000;
const HAIKU  = "claude-haiku-4-5-20251001";
const SONNET = "claude-sonnet-4-20250514";
const API_KEY = process.env.ANTHROPIC_API_KEY;
const API_URL = "https://api.anthropic.com/v1/messages";

const VALID_TOPICS = [
  "AI & tech",
  "World affairs",
  "Science",
  "Business",
  "Politics",
  "Climate",
];

// ── CLI ────────────────────────────────────────────────────────────────────────
const args = process.argv.slice(2);
if (args.includes("--clear-cache")) {
  fs.existsSync(CACHE_FILE) && fs.unlinkSync(CACHE_FILE);
  console.log("Cache cleared.");
  process.exit(0);
}
if (args.includes("--help") || args.includes("-h")) {
  console.log(`news-agent — Daily news briefing pipeline

Usage:  node news-agent.js [options]

Options:
  --clear-cache    Clear the per-topic cache
  --dry-run        Print output to stdout instead of writing a file
  -h, --help       Show this help

Config:  ${CONFIG_FILE}
Cache:   ${CACHE_FILE}

API key: set ANTHROPIC_API_KEY in your environment`);
  process.exit(0);
}

// ── Helpers ────────────────────────────────────────────────────────────────────

function parseFrontmatter(text) {
  const m = text.match(/^---\n([\s\S]*?)\n---/);
  if (!m) throw new Error("No frontmatter found in config");
  // Minimal YAML — handles the known config schema only
  const raw = m[1];
  const out = { topics: [], stories_per_topic: 3, summary_length: 2 };
  let inTopics = false;
  for (let line of raw.split("\n")) {
    const t = line.trim();
    if (t.startsWith("topics:")) { inTopics = true; continue; }
    if (inTopics) {
      const item = t.match(/^\s*-\s*(.+)/);
      if (item) out.topics.push(item[1].trim());
      else if (!t.startsWith("-") && t !== "") inTopics = false;
    }
    const sp = t.match(/^stories_per_topic:\s*(\d+)/);
    if (sp) out.stories_per_topic = parseInt(sp[1]);
    const sl = t.match(/^summary_length:\s*(\d+)/);
    if (sl) out.summary_length = parseInt(sl[1]);
  }
  return out;
}

function loadJSON(file) {
  try { return JSON.parse(fs.readFileSync(file, "utf8")); }
  catch { return null; }
}

function saveJSON(file, data) {
  fs.writeFileSync(file, JSON.stringify(data, null, 2));
}

function now() {
  return new Date().toISOString().replace(/T/, " ").replace(/\..+/, "");
}

function timestamp() {
  const d = new Date();
  const pad = n => String(n).padStart(2, "0");
  return `${pad(d.getMonth()+1)}-${pad(d.getDate())}-${d.getFullYear()} ${pad(d.getHours())}.${pad(d.getMinutes())}.${pad(d.getSeconds())}`;
}

function isCacheFresh(entry) {
  return entry && (Date.now() - entry.timestamp) < CACHE_TTL_MS;
}

// ── API call ───────────────────────────────────────────────────────────────────

async function apiCall({ model, prompt, useSearch = false }) {
  if (!API_KEY) throw new Error("ANTHROPIC_API_KEY not set");

  const headers = {
    "Content-Type": "application/json",
    "x-api-key": API_KEY,
    "anthropic-version": "2023-06-01",
  };
  if (useSearch) headers["anthropic-beta"] = "web-search-2025-03-05";

  const body = {
    model,
    max_tokens: 2000,
    messages: [{ role: "user", content: prompt }],
  };
  if (useSearch) body.tools = [{ type: "web_search_20250305", name: "web_search" }];

  const resp = await fetch(API_URL, {
    method: "POST", headers, body: JSON.stringify(body),
  });

  const data = await resp.json();
  if (!resp.ok) throw new Error(`API ${resp.status}: ${data?.error?.message || JSON.stringify(data)}`);

  const text = (data.content || [])
    .filter(b => b.type === "text")
    .map(b => b.text)
    .join("");

  return {
    text,
    tokens: (data.usage?.input_tokens || 0) + (data.usage?.output_tokens || 0),
  };
}

function parseJSON(raw) {
  const cleaned = raw.replace(/```json|```/g, "").trim();
  const start = cleaned.indexOf("[");
  const end   = cleaned.lastIndexOf("]");
  if (start === -1 || end === -1) throw new Error("No JSON array in response:\n" + raw.slice(0, 300));
  return JSON.parse(cleaned.slice(start, end + 1));
}

// ── Pass 1 — search headlines ─────────────────────────────────────────────────

async function pass1_search(topic, stories) {
  const prompt = `Today is ${new Date().toDateString()}. Use web search to find the ${stories} most recent significant news stories about: ${topic}.

Return ONLY a JSON array. Each object must have exactly these fields:
- "topic": "${topic}"
- "headline": string
- "snippet": string (one sentence from the search result)
- "url": string (source URL)
- "source": string (publication name)

No markdown, no explanation. Start your response with [ and end with ].`;

  console.log(`  Searching: ${topic}`);
  const { text, tokens } = await apiCall({ model: HAIKU, prompt, useSearch: true });
  const headlines = parseJSON(text);
  console.log(`  → ${headlines.length} headlines (${tokens.toLocaleString()} tokens)`);
  return { headlines, tokens };
}

// ── Filter pass ────────────────────────────────────────────────────────────────

async function pass2_filter(headlines) {
  console.log(`  Filtering ${headlines.length} items for quality…`);

  const prompt = `You are a news editor. Review these headlines and drop any that are off-topic, listicles ("top 10…"), promotional, duplicate, or low-quality clickbait. Keep only genuinely newsworthy items (score ≥ 3 out of 5).

${JSON.stringify(headlines, null, 2)}

Return ONLY the kept items as a JSON array with the exact same fields. Start with [ and end with ]. No explanation text.`;

  try {
    const { text, tokens } = await apiCall({ model: HAIKU, prompt });
    const filtered = parseJSON(text);
    const dropped = headlines.length - filtered.length;
    console.log(`  → Kept ${filtered.length}/${headlines.length}${dropped > 0 ? ` (dropped ${dropped})` : ""} (${tokens.toLocaleString()} tokens)`);
    return { headlines: filtered, tokens };
  } catch (e) {
    console.log(`  → Filter skipped (${e.message}), continuing with all ${headlines.length}`);
    return { headlines, tokens: 0 };
  }
}

// ── Pass 3 — rank & summarize ──────────────────────────────────────────────────

async function pass3_rank(headlines, sentences) {
  console.log(`  Ranking ${headlines.length} items…`);

  const prompt = `You are a senior news editor. Rank these ${headlines.length} items by global importance and newsworthiness. Write exactly ${sentences} sentence(s) per summary — tight, informative, no filler. Preserve the url and source fields exactly.

${JSON.stringify(headlines, null, 2)}

Return ONLY a JSON array. Each object: { "rank": number (1 = most important), "topic": string, "headline": string, "summary": string, "url": string, "source": string }. Start with [ and end with ]. No explanation.`;

  const { text, tokens } = await apiCall({ model: SONNET, prompt });
  const ranked = parseJSON(text).sort((a, b) => a.rank - b.rank);
  console.log(`  → Ranked (${tokens.toLocaleString()} tokens)`);
  return { headlines: ranked, tokens };
}

// ── Output ─────────────────────────────────────────────────────────────────────

function renderMarkdown(ranked, config, tokenLog) {
  const lines = [];
  lines.push(`# Daily Briefing — ${new Date().toDateString()}`);
  lines.push("");
  lines.push(`**Topics:** ${config.topics.join(", ")} | **Stories per topic:** ${config.stories_per_topic} | **Summary length:** ${config.summary_length} sentence(s)`);
  lines.push("");

  for (const item of ranked) {
    let domain = "";
    try { domain = new URL(item.url).hostname.replace("www.", ""); } catch { domain = item.source || ""; }
    lines.push("---");
    lines.push("");
    lines.push(`## #${item.rank} — ${item.headline}`);
    lines.push("");
    lines.push(`**Topic:** ${item.topic} | **Source:** [${domain}](${item.url})`);
    lines.push("");
    lines.push(item.summary);
    lines.push("");
  }

  const total = tokenLog.pass1 + tokenLog.filter + tokenLog.pass2;
  lines.push("---");
  lines.push("");
  lines.push("### Pipeline stats");
  lines.push("");
  lines.push("| Stage | Tokens |");
  lines.push("|---|---|");
  lines.push(`| Pass 1 — Haiku + search | ${tokenLog.cacheHits > 0 ? `${tokenLog.pass1.toLocaleString()} (${tokenLog.cacheHits} topic(s) from cache)` : tokenLog.pass1.toLocaleString()} |`);
  lines.push(`| Filter — Haiku | ${tokenLog.filter.toLocaleString()} |`);
  lines.push(`| Pass 2 — Sonnet | ${tokenLog.pass2.toLocaleString()} |`);
  lines.push(`| **Total** | **${total.toLocaleString()}** |`);
  lines.push("");

  return lines.join("\n");
}

// ── Main ───────────────────────────────────────────────────────────────────────

async function main() {
  console.log(`News Agent — ${new Date().toDateString()}\n`);

  // 1. Load config
  if (!fs.existsSync(CONFIG_FILE)) {
    console.error("Config not found:", CONFIG_FILE);
    process.exit(1);
  }
  const config = parseFrontmatter(fs.readFileSync(CONFIG_FILE, "utf8"));
  if (!config.topics.length) {
    console.error("No topics configured.");
    process.exit(1);
  }
  const invalid = config.topics.filter(t => !VALID_TOPICS.includes(t));
  if (invalid.length) {
    console.error("Invalid topics:", invalid.join(", "));
    console.error("Valid:", VALID_TOPICS.join(", "));
    process.exit(1);
  }
  console.log(`Config: ${config.topics.join(", ")} | ${config.stories_per_topic} stories/topic | ${config.summary_length} sentence(s)\n`);

  // 2. Load cache
  let cache = loadJSON(CACHE_FILE) || { topics: {} };

  // 3. Pass 1 — search per topic (or use cache)
  console.log("── Pass 1 — gathering headlines ──");
  const tokenLog = { pass1: 0, filter: 0, pass2: 0, cacheHits: 0 };
  let allHeadlines = [];

  for (const topic of config.topics) {
    const cached = cache.topics[topic];
    if (isCacheFresh(cached)) {
      allHeadlines.push(...cached.headlines.slice(0, config.stories_per_topic));
      tokenLog.cacheHits++;
      const ageMin = Math.round((Date.now() - cached.timestamp) / 60000);
      console.log(`  ${topic}: cache hit (${ageMin}m old) → ${Math.min(cached.headlines.length, config.stories_per_topic)} headlines`);
    } else {
      const { headlines, tokens } = await pass1_search(topic, config.stories_per_topic);
      tokenLog.pass1 += tokens;
      allHeadlines.push(...headlines);
      cache.topics[topic] = { timestamp: Date.now(), headlines };
    }
  }

  saveJSON(CACHE_FILE, cache);
  console.log(`\n  Total gathered: ${allHeadlines.length} headlines\n`);

  // 4. Filter pass
  console.log("── Filter pass ──");
  const fResult = await pass2_filter(allHeadlines);
  tokenLog.filter = fResult.tokens;
  let filtered = fResult.headlines;
  console.log("");

  // 5. Rank pass
  console.log("── Pass 2 — rank & summarize ──");
  const rResult = await pass3_rank(filtered, config.summary_length);
  tokenLog.pass2 = rResult.tokens;
  console.log("");

  // 6. Output
  const markdown = renderMarkdown(rResult.headlines, config, tokenLog);
  const dryRun = args.includes("--dry-run");

  if (dryRun) {
    console.log(markdown);
  } else {
    const outFile = path.join(OBSIDIAN_BASE, `${timestamp()}.md`);
    fs.writeFileSync(outFile, markdown);
    console.log(`Output: ${outFile}`);
  }

  const total = tokenLog.pass1 + tokenLog.filter + tokenLog.pass2;
  console.log(`\nTokens: ${total.toLocaleString()} total (Pass 1: ${tokenLog.pass1.toLocaleString()}, Filter: ${tokenLog.filter.toLocaleString()}, Rank: ${tokenLog.pass2.toLocaleString()})`);
  if (tokenLog.cacheHits > 0) console.log(`Cache:  ${tokenLog.cacheHits} topic(s) served from cache`);
}

main().catch(e => {
  console.error("\nFatal:", e.message);
  process.exit(1);
});
```

## Usage

```bash
export ANTHROPIC_API_KEY="sk-ant-…"
node "news agent 1.1/news-agent.js"           # full run
node "news agent 1.1/news-agent.js" --dry-run # stdout only
node "news agent 1.1/news-agent.js" --clear-cache
```
