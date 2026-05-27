# News Agent

## HTML App (`news-agent.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Daily News Agent</title>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #ffffff;
    --bg-secondary: #f5f5f3;
    --bg-info: #e6f1fb;
    --bg-success: #eaf3de;
    --bg-warning: #faeeda;
    --bg-danger: #fcebeb;
    --text: #1a1a1a;
    --text-secondary: #6b6b68;
    --text-info: #0c447c;
    --text-success: #27500a;
    --text-warning: #633806;
    --text-danger: #791f1f;
    --border: rgba(0,0,0,0.12);
    --border-strong: rgba(0,0,0,0.22);
    --radius: 8px;
    --radius-lg: 12px;
    --green: #1D9E75;
    --amber: #EF9F27;
    --red: #E24B4A;
    --teal-light: #9FE1CB;
    --orange-light: #F0997B;
  }

  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #1c1c1a;
      --bg-secondary: #252523;
      --bg-info: #042c53;
      --bg-success: #173404;
      --bg-warning: #412402;
      --bg-danger: #501313;
      --text: #f0ede8;
      --text-secondary: #9b9990;
      --text-info: #85b7eb;
      --text-success: #c0dd97;
      --text-warning: #fac775;
      --text-danger: #f09595;
      --border: rgba(255,255,255,0.1);
      --border-strong: rgba(255,255,255,0.2);
    }
  }

  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    background: var(--bg-secondary);
    color: var(--text);
    min-height: 100vh;
    padding: 2rem 1rem;
    line-height: 1.6;
  }

  .container { max-width: 720px; margin: 0 auto; }

  h1 {
    font-size: 20px;
    font-weight: 500;
    margin-bottom: 4px;
    color: var(--text);
  }

  .subtitle {
    font-size: 13px;
    color: var(--text-secondary);
    margin-bottom: 1.5rem;
  }

  .card {
    background: var(--bg);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-lg);
    padding: 1rem 1.25rem;
    margin-bottom: 12px;
  }

  .card label {
    font-size: 11px;
    color: var(--text-secondary);
    display: block;
    margin-bottom: 6px;
    letter-spacing: 0.05em;
    text-transform: uppercase;
  }

  /* API Key */
  .key-row { display: flex; gap: 8px; align-items: center; }
  .key-row input[type=password],
  .key-row input[type=text] {
    flex: 1;
    font-size: 13px;
    padding: 7px 10px;
    border: 0.5px solid var(--border-strong);
    border-radius: var(--radius);
    background: var(--bg-secondary);
    color: var(--text);
    font-family: monospace;
    outline: none;
  }
  .key-row input:focus { border-color: #378add; }
  .toggle-btn {
    font-size: 12px;
    padding: 7px 10px;
    border: 0.5px solid var(--border-strong);
    border-radius: var(--radius);
    background: transparent;
    color: var(--text-secondary);
    cursor: pointer;
    white-space: nowrap;
  }
  .toggle-btn:hover { background: var(--bg-secondary); }
  .key-note { font-size: 11px; color: var(--text-secondary); margin-top: 6px; }
  .key-note a { color: var(--text-info); }

  /* Config grid */
  .config-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 12px; }
  @media (max-width: 540px) { .config-grid { grid-template-columns: 1fr; } }

  /* Tags */
  .tag-row { display: flex; flex-wrap: wrap; gap: 6px; }
  .tag {
    font-size: 13px;
    padding: 4px 10px;
    border-radius: 20px;
    cursor: pointer;
    border: 0.5px solid var(--border-strong);
    color: var(--text-secondary);
    background: transparent;
    transition: all 0.15s;
    user-select: none;
  }
  .tag.on {
    background: var(--bg-info);
    color: var(--text-info);
    border-color: #378add;
  }

  /* Sliders */
  .slider-row { display: flex; align-items: center; gap: 10px; }
  .slider-row input[type=range] { flex: 1; accent-color: #378add; }
  .slider-val { font-size: 13px; font-weight: 500; min-width: 80px; color: var(--text); }

  /* Cache bar */
  .cache-bar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: var(--bg-secondary);
    border: 0.5px solid var(--border);
    border-radius: var(--radius);
    padding: 8px 12px;
    margin-bottom: 12px;
    font-size: 12px;
    color: var(--text-secondary);
  }
  .cache-badge {
    font-size: 11px;
    padding: 2px 8px;
    border-radius: 20px;
    font-weight: 500;
  }
  .cache-hit { background: var(--bg-success); color: var(--text-success); }
  .cache-miss { background: var(--bg-warning); color: var(--text-warning); }
  .cache-clear {
    font-size: 11px;
    color: var(--text-secondary);
    cursor: pointer;
    border: none;
    background: none;
    padding: 0;
    text-decoration: underline;
  }

  /* Pipeline */
  .pipeline { display: flex; margin-bottom: 12px; }
  .pipe-step {
    flex: 1;
    padding: 8px 10px;
    border: 0.5px solid var(--border);
    background: var(--bg);
    font-size: 11px;
    text-align: center;
  }
  .pipe-step:first-child { border-radius: var(--radius) 0 0 var(--radius); }
  .pipe-step:last-child { border-radius: 0 var(--radius) var(--radius) 0; }
  .pipe-step + .pipe-step { border-left: none; }
  .pipe-label { color: var(--text-secondary); margin-bottom: 2px; }
  .pipe-model { font-weight: 500; color: var(--text); font-size: 12px; }
  .pipe-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    background: var(--border);
    margin: 4px auto 0;
    transition: background 0.2s;
  }
  .pipe-dot.active { background: var(--amber); }
  .pipe-dot.done { background: var(--green); }
  .pipe-dot.skip { background: var(--teal-light); }
  .pipe-dot.filtered { background: var(--orange-light); }

  /* Run button */
  .run-btn {
    width: 100%;
    padding: 11px;
    font-size: 15px;
    font-weight: 500;
    cursor: pointer;
    border-radius: var(--radius);
    border: 0.5px solid var(--border-strong);
    background: var(--bg);
    color: var(--text);
    transition: background 0.15s;
    margin-bottom: 12px;
  }
  .run-btn:hover:not(:disabled) { background: var(--bg-secondary); }
  .run-btn:disabled { opacity: 0.45; cursor: not-allowed; }

  /* Status + error */
  .status { font-size: 13px; color: var(--text-secondary); min-height: 20px; margin-bottom: 8px; }
  .error-box {
    font-size: 12px;
    color: var(--text-danger);
    background: var(--bg-danger);
    border: 0.5px solid #f09595;
    border-radius: var(--radius);
    padding: 8px 12px;
    margin-bottom: 10px;
    display: none;
    white-space: pre-wrap;
    word-break: break-all;
  }

  /* News items */
  .news-list { display: flex; flex-direction: column; gap: 10px; }
  .news-item {
    background: var(--bg);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-lg);
    padding: 1rem 1.25rem;
  }
  .item-header { display: flex; align-items: flex-start; gap: 10px; margin-bottom: 6px; }
  .rank-badge {
    font-size: 11px;
    font-weight: 500;
    padding: 2px 7px;
    border-radius: 20px;
    background: var(--bg-secondary);
    color: var(--text-secondary);
    white-space: nowrap;
    flex-shrink: 0;
    margin-top: 2px;
  }
  .item-title { font-size: 14px; font-weight: 500; color: var(--text); line-height: 1.4; }
  .item-title a { color: inherit; text-decoration: none; }
  .item-title a:hover { text-decoration: underline; }
  .item-summary { font-size: 13px; color: var(--text-secondary); line-height: 1.6; margin-bottom: 8px; }
  .item-meta { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
  .topic-pill { font-size: 11px; padding: 2px 8px; border-radius: 20px; }
  .cat-ai       { background: #E1F5EE; color: #085041; }
  .cat-world    { background: #EEEDFE; color: #3C3489; }
  .cat-science  { background: #EAF3DE; color: #27500A; }
  .cat-business { background: #FAEEDA; color: #633806; }
  .cat-politics { background: #FBEAF0; color: #72243E; }
  .cat-climate  { background: #E6F1FB; color: #0C447C; }
  .source-link a { color: var(--text-info); text-decoration: none; font-size: 11px; }
  .source-link a:hover { text-decoration: underline; }

  /* Token panel */
  .token-panel {
    background: var(--bg-secondary);
    border: 0.5px solid var(--border);
    border-radius: var(--radius);
    padding: 10px 14px;
    margin-top: 1rem;
  }
  .token-row {
    display: flex;
    justify-content: space-between;
    font-size: 12px;
    color: var(--text-secondary);
    margin-bottom: 3px;
  }
  .token-row.total {
    border-top: 0.5px solid var(--border);
    margin-top: 5px;
    padding-top: 5px;
    color: var(--text);
    font-weight: 500;
  }
  .token-row span:last-child { color: var(--text); font-weight: 500; }
  .savings-note {
    font-size: 11px;
    color: var(--text-success);
    background: var(--bg-success);
    border-radius: var(--radius);
    padding: 4px 10px;
    margin-top: 8px;
  }
</style>
</head>
<body>
<div class="container">
  <h1>Daily News Agent</h1>
  <p class="subtitle">Two-pass pipeline — Haiku gathers & filters, Sonnet ranks & writes. Cache skips Pass 1 for 6 hours.</p>

  <!-- API Key -->
  <div class="card">
    <label>Anthropic API key</label>
    <div class="key-row">
      <input type="password" id="api-key" placeholder="sk-ant-…" autocomplete="off">
      <button class="toggle-btn" onclick="toggleKey()">Show</button>
    </div>
    <p class="key-note">Never leaves your browser. Get a key at <a href="https://console.anthropic.com" target="_blank">console.anthropic.com</a> — Haiku + Sonnet usage applies.</p>
  </div>

  <!-- Config -->
  <div class="config-grid">
    <div class="card">
      <label>Topics (pick up to 4)</label>
      <div class="tag-row" id="topic-tags">
        <span class="tag on" data-t="AI & tech">AI &amp; tech</span>
        <span class="tag on" data-t="World affairs">World affairs</span>
        <span class="tag" data-t="Science">Science</span>
        <span class="tag" data-t="Business">Business</span>
        <span class="tag" data-t="Politics">Politics</span>
        <span class="tag" data-t="Climate">Climate</span>
      </div>
    </div>
    <div class="card">
      <label>Stories per topic</label>
      <div class="slider-row" style="margin-bottom:12px;">
        <input type="range" min="1" max="5" value="3" step="1" id="stories-slider">
        <span class="slider-val" id="stories-val">3</span>
      </div>
      <label>Summary length</label>
      <div class="slider-row">
        <input type="range" min="1" max="4" value="2" step="1" id="len-slider">
        <span class="slider-val" id="len-val">2 sentences</span>
      </div>
    </div>
  </div>

  <!-- Cache bar -->
  <div class="cache-bar" id="cache-bar" style="display:none;">
    <div style="display:flex;align-items:center;gap:8px;">
      🗄 <span id="cache-status-text">Checking cache…</span>
    </div>
    <div style="display:flex;align-items:center;gap:8px;">
      <span class="cache-badge" id="cache-badge"></span>
      <button class="cache-clear" onclick="clearCache()">Clear cache</button>
    </div>
  </div>

  <!-- Pipeline indicator -->
  <div class="pipeline">
    <div class="pipe-step">
      <div class="pipe-label">Pass 1</div>
      <div class="pipe-model">Haiku + search</div>
      <div class="pipe-dot" id="dot1"></div>
    </div>
    <div class="pipe-step">
      <div class="pipe-label">Filter</div>
      <div class="pipe-model">Haiku</div>
      <div class="pipe-dot" id="dot2"></div>
    </div>
    <div class="pipe-step">
      <div class="pipe-label">Pass 2</div>
      <div class="pipe-model">Sonnet</div>
      <div class="pipe-dot" id="dot3"></div>
    </div>
  </div>

  <button class="run-btn" id="run-btn" onclick="runAgent()">▶ Generate today's briefing</button>

  <div class="status" id="status-msg"></div>
  <div class="error-box" id="error-box"></div>
  <div id="results" class="news-list"></div>
  <div id="token-panel" style="display:none;"></div>
</div>

<script>
  const CACHE_TTL_MS = 6 * 60 * 60 * 1000;
  const HAIKU  = "claude-haiku-4-5-20251001";
  const SONNET = "claude-sonnet-4-20250514";
  const LEN_LABELS = ["1 sentence","2 sentences","3 sentences","4 sentences"];
  const CAT_CLASS = {
    "AI & tech":    "cat-ai",
    "World affairs":"cat-world",
    "Science":      "cat-science",
    "Business":     "cat-business",
    "Politics":     "cat-politics",
    "Climate":      "cat-climate"
  };

  const selectedTopics = new Set(["AI & tech","World affairs"]);

  // ── UI wiring ──────────────────────────────────────────────────────────────

  document.querySelectorAll("#topic-tags .tag").forEach(t => {
    t.addEventListener("click", () => {
      const v = t.dataset.t;
      if (selectedTopics.has(v)) {
        selectedTopics.delete(v); t.classList.remove("on");
      } else if (selectedTopics.size < 4) {
        selectedTopics.add(v); t.classList.add("on");
      }
      updateCacheBar();
    });
  });

  document.getElementById("stories-slider").addEventListener("input", e => {
    document.getElementById("stories-val").textContent = e.target.value;
    updateCacheBar();
  });

  document.getElementById("len-slider").addEventListener("input", e => {
    document.getElementById("len-val").textContent = LEN_LABELS[+e.target.value - 1];
  });

  function toggleKey() {
    const inp = document.getElementById("api-key");
    const btn = document.querySelector(".toggle-btn");
    if (inp.type === "password") { inp.type = "text"; btn.textContent = "Hide"; }
    else { inp.type = "password"; btn.textContent = "Show"; }
  }

  function getKey() { return document.getElementById("api-key").value.trim(); }

  // ── Cache (localStorage) ───────────────────────────────────────────────────

  function cacheKey() {
    const topics = [...selectedTopics].sort().join("-").replace(/[^a-zA-Z0-9-]/g, "+");
    const stories = document.getElementById("stories-slider").value;
    return `news-agent-${topics}-${stories}`;
  }

  function getCached() {
    try {
      const raw = localStorage.getItem(cacheKey());
      if (!raw) return null;
      const p = JSON.parse(raw);
      const age = Date.now() - p.timestamp;
      if (age > CACHE_TTL_MS) return null;
      return { data: p.headlines, age };
    } catch { return null; }
  }

  function setCache(headlines) {
    try {
      localStorage.setItem(cacheKey(), JSON.stringify({ timestamp: Date.now(), headlines }));
    } catch(e) { console.warn("Cache write failed", e); }
  }

  function clearCache() {
    Object.keys(localStorage)
      .filter(k => k.startsWith("news-agent-"))
      .forEach(k => localStorage.removeItem(k));
    updateCacheBar();
  }

  function updateCacheBar() {
    const bar = document.getElementById("cache-bar");
    bar.style.display = "flex";
    const cached = getCached();
    const badge = document.getElementById("cache-badge");
    const text  = document.getElementById("cache-status-text");
    if (cached) {
      const mins = Math.round(cached.age / 60000);
      const hrs  = Math.floor(mins / 60);
      const rem  = mins % 60;
      const age  = hrs > 0 ? `${hrs}h ${rem}m ago` : `${mins}m ago`;
      const left = Math.round((CACHE_TTL_MS - cached.age) / 60000);
      text.textContent  = `Cached — expires in ${left}m`;
      badge.textContent = `HIT · ${age}`;
      badge.className   = "cache-badge cache-hit";
    } else {
      text.textContent  = "No cache for this config";
      badge.textContent = "MISS";
      badge.className   = "cache-badge cache-miss";
    }
  }

  // ── Helpers ────────────────────────────────────────────────────────────────

  function dot(id, state) {
    document.getElementById(id).className = "pipe-dot" + (state ? " " + state : "");
  }
  function setStatus(msg)  { document.getElementById("status-msg").textContent = msg; }
  function showError(msg)  { const b = document.getElementById("error-box"); b.textContent = msg; b.style.display = "block"; }
  function hideError()     { document.getElementById("error-box").style.display = "none"; }

  async function apiCall({ model, prompt, useSearch = false }) {
    const key = getKey();
    if (!key) throw new Error("No API key — enter your key above.");

    const headers = { "Content-Type": "application/json", "x-api-key": key, "anthropic-version": "2023-06-01" };
    if (useSearch) headers["anthropic-beta"] = "web-search-2025-03-05";

    const body = { model, max_tokens: 1000, messages: [{ role: "user", content: prompt }] };
    if (useSearch) body.tools = [{ type: "web_search_20250305", name: "web_search" }];

    const resp = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST", headers, body: JSON.stringify(body)
    });

    const data = await resp.json();
    if (!resp.ok) throw new Error(`API ${resp.status}: ${data?.error?.message || JSON.stringify(data)}`);

    const text   = (data.content || []).filter(b => b.type === "text").map(b => b.text).join("");
    const tokens = (data.usage?.input_tokens || 0) + (data.usage?.output_tokens || 0);
    return { text, tokens };
  }

  function parseJSON(raw) {
    const cleaned = raw.replace(/```json|```/g, "").trim();
    const start = cleaned.indexOf("[");
    const end   = cleaned.lastIndexOf("]");
    if (start === -1 || end === -1) throw new Error("No JSON array in response:\n" + raw.slice(0, 300));
    return JSON.parse(cleaned.slice(start, end + 1));
  }

  // ── Main pipeline ──────────────────────────────────────────────────────────

  async function runAgent() {
    const topics = [...selectedTopics];
    if (!topics.length) { setStatus("Pick at least one topic."); return; }
    if (!getKey())      { showError("Enter your Anthropic API key above."); return; }

    const stories   = +document.getElementById("stories-slider").value;
    const sentences = +document.getElementById("len-slider").value;
    const btn       = document.getElementById("run-btn");
    const results   = document.getElementById("results");
    const tokenPanel = document.getElementById("token-panel");

    btn.disabled = true;
    results.innerHTML = "";
    tokenPanel.style.display = "none";
    hideError();
    dot("dot1",""); dot("dot2",""); dot("dot3","");

    const tokenLog = { pass1: 0, filter: 0, pass2: 0, cacheHit: false };
    let headlines = [];

    // ── Pass 1 or cache ──────────────────────────────────────────────────────
    const cached = getCached();
    if (cached) {
      tokenLog.cacheHit = true;
      headlines = cached.data;
      dot("dot1","skip");
      setStatus(`Cache hit — ${headlines.length} headlines loaded, skipping Pass 1`);
      updateCacheBar();
    } else {
      dot("dot1","active");
      setStatus("Pass 1 — Haiku searching headlines…");

      const p1 = `Today is ${new Date().toDateString()}. Use web search to find the ${stories} most recent significant news stories for each of these topics: ${topics.join(", ")}.

Return ONLY a JSON array. Each object must have exactly these fields:
- "topic": one of the input topic strings
- "headline": string
- "snippet": string (one sentence from the search result)
- "url": string (source URL)
- "source": string (publication name)

No markdown, no explanation. Start your response with [ and end with ].`;

      try {
        const { text, tokens } = await apiCall({ model: HAIKU, prompt: p1, useSearch: true });
        tokenLog.pass1 = tokens;
        headlines = parseJSON(text);
        setCache(headlines);
        dot("dot1","done");
        updateCacheBar();
      } catch(e) {
        dot("dot1","");
        showError("Pass 1 failed: " + e.message);
        setStatus("");
        btn.disabled = false;
        return;
      }
    }

    // ── Filter pass ──────────────────────────────────────────────────────────
    dot("dot2","active");
    const preFilterCount = headlines.length;
    setStatus(`Filter — scoring ${preFilterCount} headlines for quality…`);

    const pFilter = `You are a news editor. Review these headlines and drop any that are off-topic, listicles ("top 10…"), promotional, duplicate, or low-quality clickbait. Keep only genuinely newsworthy items (score ≥ 3 out of 5).

${JSON.stringify(headlines, null, 2)}

Return ONLY the kept items as a JSON array with the exact same fields. Start with [ and end with ]. No explanation text.`;

    try {
      const { text, tokens } = await apiCall({ model: HAIKU, prompt: pFilter });
      tokenLog.filter = tokens;
      const filtered = parseJSON(text);
      const dropped  = preFilterCount - filtered.length;
      headlines      = filtered;
      dot("dot2", dropped > 0 ? "filtered" : "done");
      setStatus(`Filter done — kept ${headlines.length}/${preFilterCount}${dropped > 0 ? ` (dropped ${dropped} low-quality)` : ""}`);
    } catch(e) {
      dot("dot2","done");
      setStatus(`Filter skipped (${e.message}) — continuing with all ${headlines.length} headlines`);
    }

    // ── Pass 2 — rank & summarize ────────────────────────────────────────────
    dot("dot3","active");
    setStatus("Pass 2 — Sonnet ranking and summarizing…");

    const p2 = `You are a senior news editor. Rank these ${headlines.length} items by global importance and newsworthiness. Write exactly ${sentences} sentence(s) per summary — tight, informative, no filler. Preserve the url and source fields exactly.

${JSON.stringify(headlines, null, 2)}

Return ONLY a JSON array. Each object: { "rank": number (1 = most important), "topic": string, "headline": string, "summary": string, "url": string, "source": string }. Start with [ and end with ]. No explanation.`;

    try {
      const { text, tokens } = await apiCall({ model: SONNET, prompt: p2 });
      tokenLog.pass2 = tokens;
      const ranked = parseJSON(text).sort((a,b) => a.rank - b.rank);
      dot("dot3","done");
      setStatus(`Briefing ready — ${ranked.length} stories`);

      results.innerHTML = ranked.map(item => {
        const cc     = CAT_CLASS[item.topic] || "cat-world";
        const domain = (() => { try { return new URL(item.url).hostname.replace("www.",""); } catch { return item.source || ""; } })();
        return `<div class="news-item">
          <div class="item-header">
            <span class="rank-badge">#${item.rank}</span>
            <span class="item-title"><a href="${item.url}" target="_blank" rel="noopener">${item.headline}</a></span>
          </div>
          <p class="item-summary">${item.summary}</p>
          <div class="item-meta">
            <span class="topic-pill ${cc}">${item.topic}</span>
            <span class="source-link"><a href="${item.url}" target="_blank" rel="noopener">↗ ${domain}</a></span>
          </div>
        </div>`;
      }).join("");

      const total = tokenLog.pass1 + tokenLog.filter + tokenLog.pass2;
      tokenPanel.innerHTML = `<div class="token-panel">
        <div class="token-row"><span>Pass 1 — Haiku + search</span><span>${tokenLog.cacheHit ? "0 (cache hit)" : tokenLog.pass1.toLocaleString()}</span></div>
        <div class="token-row"><span>Filter — Haiku</span><span>${tokenLog.filter.toLocaleString()}</span></div>
        <div class="token-row"><span>Pass 2 — Sonnet</span><span>${tokenLog.pass2.toLocaleString()}</span></div>
        <div class="token-row total"><span>Total this run</span><span>${total.toLocaleString()}</span></div>
        ${tokenLog.cacheHit ? `<div class="savings-note">✓ Pass 1 served from cache — search tokens saved</div>` : ""}
      </div>`;
      tokenPanel.style.display = "block";

    } catch(e) {
      dot("dot3","");
      showError("Pass 2 failed: " + e.message);
      setStatus("");
    }

    btn.disabled = false;
  }

  // ── Init ───────────────────────────────────────────────────────────────────
  updateCacheBar();
</script>
</body>
</html>
```

---

## Obsidian Integration

### Config (`News Agent Config.md`)

```yaml
---
topics:
  - AI & tech
  - World affairs
stories_per_topic: 3
summary_length: 2
---
```

**Available topics:** `AI & tech`, `World affairs`, `Science`, `Business`, `Politics`, `Climate`
**stories_per_topic:** 1–5
**summary_length:** 1–4 (sentences)

### Cache (`.news-cache.json`)

```json
{
  "topics": {
    "AI & tech": {
      "timestamp": 1768334400000,
      "headlines": [
        {
          "topic": "AI & tech",
          "headline": "...",
          "snippet": "...",
          "url": "https://...",
          "source": "..."
        }
      ]
    }
  }
}
```

TTL: 6 hours (21,600,000 ms). Cache is per-topic — adding a new topic only searches the new one.

### Manual Pipeline Protocol

1. Read `News Agent Config.md` for topics, `stories_per_topic`, `summary_length`
2. Read `.news-cache.json`
3. For each topic: if cached and < 6h old, reuse. Otherwise, one WebSearch per topic.
4. Update cache with fresh results.
5. Pass 1 (gather): merge cached + fresh headlines, trim to `stories_per_topic` per topic.
6. Filter: drop low-quality, off-topic, listicles, clickbait.
7. Pass 2 (rank): rank all remaining by global importance, write summaries.
8. Output to `MM-DD-YYYY HH.MM.SS.md`.
