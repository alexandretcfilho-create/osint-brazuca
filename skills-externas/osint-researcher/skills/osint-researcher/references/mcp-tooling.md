# MCP Tooling - Routing OSINT Collection Through Managed Tools

> Loaded on demand. This file governs **how** the `osint-researcher` skill actually reaches
> the open web: every search, scrape, crawl, extraction, and evidence capture is routed
> through the **connected MCP tools and managed scraping APIs available to you** - never from
> the local machine. It is the operational counterpart to `methodology.md` (the analytic
> backbone) and `legal-ethics-opsec.md` (the gate). For the OPSEC rule this file implements,
> see `legal-ethics-opsec.md §6` (OPSEC & attribution management); for evidence handling see
> `legal-ethics-opsec.md §7` and `methodology.md §7.2`.
>
> One rule sits above all tooling: **no live retrieval ever egresses from this workstation.**
> If a request cannot be satisfied through a managed tool, it does not get satisfied by
> falling back to a local HTTP call - it gets re-planned or refused.

---

## 0. Why route everything through managed tools

Four independent reasons converge on the same rule, and any one of them alone would be
sufficient.

1. **Attribution / OPSEC (the load-bearing reason).** The investigator's real IP and identity
   must never be exposed to a target. `legal-ethics-opsec.md §6.1` states it as both an OPSEC
   rule and a firm collection policy: *route all collection through the connected scraping /
   search MCP tools and managed APIs - never from the local machine.* A managed tool egresses
   from **its** infrastructure and proxy pool, not from this host, so a target's logs, its
   analytics, and any counter-investigation see the provider's footprint, not yours.
   A target may have equal or greater recon skill and may counter-investigate; do not hand
   them your IP.
2. **A clean no-local-scraping posture.** Independently of OSINT, it is sound tradecraft
   never to issue direct HTTP to a third-party target from your own machine - no `requests`, `httpx`,
   `curl`/`wget`, `puppeteer`/`playwright.connect()`, or local `crawlee`. Managed tools are
   the sanctioned egress. The two rules coincide exactly.
3. **Anti-bot capability.** Hard targets sit behind Cloudflare, DataDome, PerimeterX, Akamai,
   and AWS WAF. Managed providers carry residential/mobile proxy pools, real browser
   fingerprints, JS rendering, and CAPTCHA handling that a local script cannot match - and
   that this skill must not build locally.
4. **Cost, caching, and reproducibility.** Managed tools cache (Firecrawl `maxAge`), return
   clean main-content markdown, and give a stable, quotable provenance string for the evidence
   log. Re-querying the same page from a raw client wastes quota and muddies provenance.

> **The passive-first ladder still applies (`methodology.md §2`).** Managed tooling does not
> upgrade you past the intrusiveness ceiling. A crawl is still semi-passive-to-active toward
> the target even though it egresses from a provider; active enumeration against a third party
> still needs written authorization. The tool changes *who the target sees*, not *what rung
> you are on*.

---

## 1. The connected providers and their OSINT role

All of these are configured as MCP servers or managed APIs, keyed from a
secret store - see §7). This is the routing catalogue; it is not an invitation to climb the
intrusiveness ladder.

| Provider | Surface | OSINT role | Notes |
|----------|---------|-----------|-------|
| **Firecrawl MCP** | `firecrawl_search`, `firecrawl_scrape`, `firecrawl_map`, `firecrawl_crawl`, `firecrawl_extract`, `firecrawl_interact` | **Primary** - search + scrape + crawl + structured extract + PDF parse + browser actions | Best default for almost everything; supports `proxy: stealth` for protected pages, `parsers:["pdf"]`, `maxAge` cache |
| **SerpApi** | `search` (`engine: google_light`) | Search-engine result harvesting: dorking at scale, SERP structure (people-also-ask, related searches) | Compact mode; the tool for operator-heavy Google/Bing queries |
| **Tavily MCP** | `tavily_search`, `tavily_extract`, `tavily_crawl`, `tavily_map`, `tavily_research` | Research-grade search + a **second search opinion**; synthesis-oriented | `search_depth`, `country`; good for corroboration diversity |
| **ZenRows** | Scraping API | **Hard-target scrape** - residential proxies, JS render, CAPTCHA | First anti-bot escalation for Cloudflare/DataDome/PerimeterX |
| **Scrape.do** | Scraping API | Fallback hard-target scrape - rotating proxies + headless render | Second anti-bot escalation |
| **Browserless** | Hosted headless Chrome | Full JS render, **interaction, and evidence screenshots** | Capture full-page PNG for chain-of-custody |
| **Exa** | Neural/semantic search | Semantic / entity / company / code discovery when keywords fail | `type: auto`; finds *conceptually* similar pages |
| **Brave** | Fast web search | Quick web search; a third search opinion; SERP fallback | Lightweight |
| **Perplexity** | Deep synthesis | Multi-source synthesis and framing of a topic | Synthesis, not a source of record - always trace to primaries |
| **Apify MCP** (`mcp.apify.com`) | Run Store Actors | Social / platform data (profiles, posts), maps, structured site scrapers | `search-actors` → `fetch-actor-details` → `call-actor`; prefer high-usage Actors |
| **Other keyed APIs** | Scrapfly, ScraperAPI, BrightData, WebScrapingAPI, ScrapingBee, etc. | Additional anti-bot / proxy fallbacks by API key | Use only when the ladder above is exhausted |

---

## 2. Tool-selection matrix - OSINT job → best tool → fallback

Pick by the **job**, then follow the fallback chain only when the primary returns poor or
blocked results. Do not reformulate the same query three times on the same tool before
switching.

| OSINT job | Primary | Fallback(s) | Note |
|-----------|---------|-------------|------|
| **SERP / dork harvesting** (operators, at scale) | **SerpApi** (`engine: google_light`, compact) | Firecrawl `firecrawl_search` → Brave | SerpApi preserves SERP structure (PAA, related); dork operators go here |
| **Broad web search + content** | **Firecrawl** `firecrawl_search` | Tavily `tavily_search` → Brave | Firecrawl returns full-page content inline; Tavily is the second opinion |
| **Second/independent search opinion** (corroboration) | **Tavily** `tavily_search` | Brave → Exa | Independence matters for corroboration (`methodology.md §3.3`) |
| **Single-page scrape** | **Firecrawl** `firecrawl_scrape` (`onlyMainContent`, `maxAge`) | Firecrawl `proxy: stealth` → ZenRows → Scrape.do → Browserless | See the escalation ladder, §5 |
| **Whole-site URL enumeration** | **Firecrawl** `firecrawl_map` | Tavily `tavily_map` | Cheap way to see a site's shape before crawling |
| **Multi-page crawl** | **Firecrawl** `firecrawl_crawl` (tight `limit`) | Tavily `tavily_crawl` | Semi-passive→active toward target - respect the ceiling |
| **Structured / schema extraction** | **Firecrawl** `firecrawl_extract` (JSON schema) | Tavily `tavily_extract` (fields) | One call per entity type; keep the schema tight |
| **JS-heavy render / interaction** | **Firecrawl** `firecrawl_interact` | Browserless | For content behind clicks/scroll (public only - never past auth) |
| **Evidence screenshot** | **Browserless** (full-page PNG) | Firecrawl `firecrawl_scrape` `formats:["screenshot"]` | Capture for chain-of-custody (§6) |
| **Semantic / entity / company / code search** | **Exa** (`type: auto`) | Firecrawl search → Tavily | When you know the *concept* but not the keywords |
| **Deep synthesis / topic framing** | **Perplexity** | Tavily `tavily_research` | Synthesis only - never a source of record; trace to primaries |
| **Social / platform data** | **Apify MCP** (Store Actor) | provider-specific keyed API | ToS- and law-sensitive - gate through `legal-ethics-opsec.md §4.5, §6.3` |
| **PDF parsing** | **Firecrawl** `firecrawl_scrape` `parsers:["pdf"]` | Firecrawl `firecrawl_extract` on the PDF URL | Filings, reports, disclosures |
| **Site-shape + then content** (recon) | `firecrawl_map` → select → `firecrawl_scrape` | - | Map first (cheap), scrape only what you need |

---

## 3. Usage patterns - representative invocations

Parameters below are illustrative of the shape of each call; adjust to the live schema. All
examples assume the passive-first ceiling and authorization are already cleared per the gate.

### 3.1 Google dork via SerpApi

Search-engine harvesting with operators. SerpApi runs the query on Google's own engine - this
is passive (you never touch the target's systems), and it is lawful; **using** anything a dork
surfaces (a config file, a credential) is not (`legal-ethics-opsec.md §3, §2.1`).

```
SerpApi.search {
  engine: "google_light",          // lightweight, cheap SERP
  q: 'site:example.com filetype:pdf ("internal" OR "confidential")',
  num: 20,
  // compact response mode to minimize tokens/quota
}
```

Read the organic results, `people_also_ask`, and `related_searches` for pivot leads
(`methodology.md §5.3`). To harvest a target's exposed document surface, iterate operators
(`filetype:`, `inurl:`, `intitle:`, `site:`) - one query per operator combination, not the
same query reworded.

### 3.2 Broad search with inline content (Firecrawl)

```
firecrawl_search {
  query: "<subject> data breach 2026",
  limit: 10,                       // keep tight - quota discipline (§4)
  sources: ["web", "news"],        // "images" available for reverse-image leads
  scrapeOptions: { onlyMainContent: true, maxAge: 172800000 }  // 48h cache
}
```

`firecrawl_search` returns ranked results **with page content already extracted**, so a single
call often replaces a search-then-scrape sequence. After using results, call
`firecrawl_search_feedback` with the search ID (improves quality and refunds 1 credit).

### 3.3 Single-page scrape, clean, then stealth (Firecrawl)

Default scrape first:

```
firecrawl_scrape {
  url: "https://example.com/target-page",
  formats: ["markdown"],
  onlyMainContent: true,
  maxAge: 172800000                // serve from cache if seen in last 48h
}
```

If the response is a challenge page, a 403/429, or empty boilerplate, re-issue **once** with
the stealth proxy before leaving Firecrawl:

```
firecrawl_scrape {
  url: "https://example.com/target-page",
  formats: ["markdown"],
  onlyMainContent: true,
  proxy: "stealth",                // residential/anti-bot egress
  waitFor: 3000                    // let JS settle
}
```

Only if stealth also fails do you climb to ZenRows / Scrape.do / Browserless (§5). Never
respond to a block by circumventing an access control - that is a red line
(`legal-ethics-opsec.md §2.1`); switching to a stronger managed egress for a *public* page is
fine, defeating authentication or a CAPTCHA gate on *non-public* content is not.

### 3.4 ZenRows fallback for a protected public page

When Firecrawl (default + stealth) cannot render a hard target, escalate to the anti-bot API.
Keys come from the secret store (§7), never inline:

```
GET https://api.zenrows.com/v1/
  ?url=https://example.com/target-page
  &js_render=true
  &premium_proxy=true
  &proxy_country=fr
# apikey supplied from the secret store / MCP config - never written in the request literal
```

Scrape.do is the equivalent next rung (rotating proxies + headless render) with the same
key-from-store discipline.

### 3.5 Site map (Firecrawl)

Enumerate a site's URLs cheaply before deciding what to crawl:

```
firecrawl_map {
  url: "https://example.com",
  limit: 500,
  includeSubdomains: true
}
```

Review the URL list, select the handful that answer the EEI, and scrape only those - do not
crawl the whole site by reflex (`methodology.md §1.2`, data minimization
`legal-ethics-opsec.md §4.1`).

### 3.6 Schema-structured extraction (Firecrawl)

Turn a page (or a few) into typed fields in one call:

```
firecrawl_extract {
  urls: ["https://example.com/company/about"],
  schema: {
    type: "object",
    properties: {
      legal_name:     { type: "string" },
      registration_id:{ type: "string" },
      officers:       { type: "array", items: { type: "string" } },
      addresses:      { type: "array", items: { type: "string" } }
    }
  },
  prompt: "Extract the company's legal identity and officers."
}
```

Keep the schema narrow to the EEI. Every extracted field is still a *finding* to be graded and
corroborated (`methodology.md §3`) - extraction is collection, not analysis.

### 3.7 Evidence screenshot (Browserless)

Full-page capture for chain-of-custody at the moment of collection:

```
POST https://<browserless-host>/screenshot
{
  "url": "https://example.com/target-page",
  "options": { "fullPage": true, "type": "png" }
}
# token supplied from the secret store - never inline
```

Then archive + hash the PNG (§6). Firecrawl's `firecrawl_scrape { formats: ["screenshot"] }`
is a lighter alternative when a hosted-Chrome render is overkill.

### 3.8 PDF parsing (Firecrawl)

```
firecrawl_scrape {
  url: "https://example.com/annual-report.pdf",
  parsers: ["pdf"],
  formats: ["markdown"]
}
```

For a structured pull from the same PDF, point `firecrawl_extract` at the PDF URL with a
schema instead.

### 3.9 Social / platform data (Apify MCP)

Platform data is ToS- and law-sensitive; clear the gate first (`legal-ethics-opsec.md §4.5`
ToS, `§6.3` sock-puppet/impersonation limits, `§2.2` no deanonymizing to harm). Then:

```
Apify: search-actors { search: "instagram profile scraper" }
Apify: fetch-actor-details { actor: "<username/slug>" }   // read input schema first
Apify: call-actor { actor: "<username/slug>", input: { ... }, waitSecs: 45 }
```

Prefer high-usage, well-rated Actors. Passive-observation-only - an Actor is never a vehicle
to reach non-public content or to interact with a target.

### 3.10 Semantic discovery (Exa) and synthesis (Perplexity)

- **Exa** when keyword search misses the concept: `Exa.search { query: "<natural-language
  description of the entity/topic>", type: "auto" }` - finds conceptually similar pages,
  companies, or code.
- **Perplexity** to frame or summarize a topic quickly - but its output is **synthesis, not a
  source of record.** Every claim must be traced back to a primary and graded
  (`methodology.md §3.3, §4`). Never quote Perplexity as evidence in a report.

---

## 4. Cost, quota & caching discipline

Quota is finite and provenance is cleaner when you do not re-fetch. Apply on every call:

- **Cache first (Firecrawl `maxAge`).** Set `maxAge` (e.g. `172800000` = 48h) so a page seen
  recently is served from cache instead of re-fetched. Re-fetch only when freshness genuinely
  matters for the time window.
- **`onlyMainContent: true`** on scrapes/searches - strips nav/ads/boilerplate, cuts tokens,
  and yields cleaner text for grading.
- **Tight `limit`.** Search `limit: 10-20`; map/crawl the minimum that answers the EEI. The
  a sensible default for Firecrawl discovery is `limit: 20-25`, `maxAge: 172800000`.
- **SerpApi compact mode** and `engine: google_light` for cheap SERPs.
- **Don't re-query the same page or re-run the same search.** One page = one authoritative
  capture; reference it thereafter. Reformulating the same query on the same tool three times
  is a smell - switch tools or switch strategy.
- **Search feedback refund.** Call `firecrawl_search_feedback` with the search ID after using
  Firecrawl search results (refunds 1 credit, improves ranking).
- **Map before crawl.** `firecrawl_map` (cheap) to see a site's shape, then scrape only the
  URLs you need - far cheaper than a blind `firecrawl_crawl`, and it respects data
  minimization (`legal-ethics-opsec.md §4.1`).
- **Right-size the provider.** Do not open Browserless (hosted Chrome) for a static page a
  plain `firecrawl_scrape` handles; do not escalate to a premium anti-bot API until the free
  rungs have actually failed.

---

## 5. Escalation ladder for blocked targets

When a scrape returns a challenge page, a 403/429, a CAPTCHA wall, or empty boilerplate, climb
one rung at a time - re-issuing the request **once** per rung - and stop the moment you get a
clean render. Escalating egress strength for a **public** page is legitimate; escalating in
order to **defeat an access control on non-public content is a red line** and this skill stops.

```
Rung 0  Firecrawl firecrawl_scrape (default, onlyMainContent, maxAge)
          │  block / challenge / empty
Rung 1  Firecrawl firecrawl_scrape { proxy: "stealth", waitFor: … }
          │  still blocked
Rung 2  ZenRows API      (js_render + premium/residential proxy)
          │  still blocked
Rung 3  Scrape.do API    (rotating proxies + headless render)
          │  still blocked
Rung 4  Browserless      (full hosted-Chrome render / interaction)
          │  still blocked
STOP  →  reconsider - see the stop conditions below
```

Beyond Rung 4, other keyed providers (Scrapfly, ScraperAPI, BrightData, WebScrapingAPI) exist
as lateral alternatives, not as license to keep hammering a hardened wall.

**When to stop climbing:**

- The block is an **authentication / paywall / CAPTCHA on non-public content** - stop.
  Bypassing an access control is unauthorized access (`legal-ethics-opsec.md §2.1`,
  `§0` core rule). Persistent challenge pages are the target saying *no*.
- **robots.txt / ToS** forbids the collection (`legal-ethics-opsec.md §4.5`). A ToS that
  forbids what the task needs is a signal to reconsider, not a hurdle to route around.
- The step would become **active toward a third party without written authorization**
  (`methodology.md §2`, `legal-ethics-opsec.md §3`).
- You are being **rate-limited deliberately** - respect it; grinding through it is
  circumvention.

When a stop condition fires, record the gap honestly (`methodology.md §7.1` intelligence gaps)
rather than laundering a block into a bypass.

---

## 6. Evidence capture through these tools

Chain-of-custody is a collection-time obligation, not a post-hoc cleanup (`legal-ethics-opsec.md
§7`, `methodology.md §7.2`). Capture every significant finding **as you collect it**, using the
managed tools:

1. **Timestamp (UTC)** - record collection date/time on every capture.
2. **Screenshot** - full-page via **Browserless** (`fullPage: true`) or Firecrawl
   `firecrawl_scrape { formats: ["screenshot"] }`.
3. **Self-contained page copy** - `firecrawl_scrape { formats: ["markdown","html"] }` (and/or
   `rawHtml`) as the retained artifact; PDFs via `parsers:["pdf"]`.
4. **Independent third-party archive** - submit the URL to the **Wayback Machine** and/or
   **archive.today** so the record does not depend on you or the source staying online.
5. **Hash** - `sha256sum` every captured file into the evidence manifest at capture time.
6. **Log provenance** - evidence log with columns `UTC timestamp | type | description | SHA256
   | source/tool`. Record **which tool** produced each artifact (e.g. "Firecrawl scrape,
   proxy: stealth" or "Browserless screenshot") - the tool is part of the provenance and the
   reproducibility trail.

The report references the **hash manifest**, not raw attachments, and carries the correct
**TLP marking** (`legal-ethics-opsec.md §7`). Retention follows the storage-limitation rule -
define a retention period and securely delete when it lapses (`legal-ethics-opsec.md §4.1,
§7`).

---

## 7. Credentials - never inline, never committed

All API keys and tokens for these providers live in the **account's secret store / MCP
configuration** (a secret manager, environment variables, or the MCP server config). Rules:

- **Never write a key or token in a request literal, a code snippet, a log, a commit, or a
  report.** The examples in §3 deliberately reference keys as *"supplied from the secret
  store"* and never show a value.
- MCP-connected providers (Firecrawl, SerpApi, Tavily, Apify, Exa, Brave, Perplexity)
  authenticate through their MCP server config - you invoke the tool, the harness supplies the
  credential; you never see or handle it.
- Direct-API providers (ZenRows, Scrape.do, Browserless, and the other keyed scrapers) read
  their key from the environment / secret store at call time. Reference the variable, never
  the value.
- If a key would ever need to appear in output for the task to proceed - it does not proceed.
  Redact and re-plan.

> Cross-references: OPSEC / no-local-egress rule → `legal-ethics-opsec.md §6`; access-control
> and ToS red lines → `§0, §2.1, §4.5`; evidence handling, retention, TLP → `§7`; the analytic
> cycle these tools feed → `methodology.md §1`; passive-first ceiling → `methodology.md §2`;
> grading every collected fact → `methodology.md §3`; reporting + chain-of-evidence →
> `methodology.md §7`.
