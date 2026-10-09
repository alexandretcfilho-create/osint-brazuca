# OSINT Skill

[![Early Beta](https://img.shields.io/badge/status-early%20beta-orange)](https://github.com/smixs/osint-skill/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Apify Actors](https://img.shields.io/badge/Apify%20actors-55%2B-green)](https://apify.com)

> **Early Beta** — functional but APIs change, Apify actors rotate, and edge cases are still being discovered.

> **[Версия на русском / Russian version](README.ru.md)**

Systematic intelligence gathering on individuals. From a name or handle to a scored dossier with psychoprofile, career map, and entry points.

### Compatible Agents

Works with any AI coding agent that supports the SKILL.md format:

| Agent | Install |
|-------|---------|
| **[Claude Code](https://claude.com/product/claude-code)** | `cp -r osint/ ~/.claude/skills/osint/` |
| **[OpenClaw](https://github.com/openclaw/openclaw)** | Copy to workspace `skills/` directory |
| **[Codex](https://developers.openai.com/codex/app)** | Point to `osint/SKILL.md` in your agent config |
| **[OpenCode](https://github.com/anomalyco/opencode)** | Copy to project `skills/` directory |
| **Any SKILL.md agent** | Place `osint/` folder where your agent reads skills |

The skill uses standard tools (`bash`, `curl`, `node`, `python3`) and writes instructions in Markdown — no vendor lock-in.

## Features

- **Phased pipeline** (0 → 1 → 1.5 → 2 → 3 → 4 → 5 → 6): from quick search to deep research
- **Swarm Mode**: coordinates 3-5 parallel sub-agents on Sonnet for speed
- **55+ Apify actors** embedded: Instagram (12), Facebook (14), TikTok (14), YouTube (5), Google Maps (4), LinkedIn, and more
- **Psychoprofile**: MBTI / Big Five based on content analysis (YouTube transcripts, Telegram messages, blogs)
- **Confidence Scoring**: every fact gets graded A/B/C/D by number of independent confirmations
- **Internal Intelligence**: checks Telegram history, email, vault contacts BEFORE going external
- **Research Escalation**: 4 levels from free to $0.50, from seconds to minutes
- **Budget tracking**: spends ≤$0.50 without asking, asks permission above that

## Quick Start

```bash
# Clone the repo
git clone https://github.com/smixs/osint-skill.git
cd osint-skill

# Copy to your agent's skills directory (example for Claude Code)
cp -r osint/ ~/.claude/skills/osint/

# Run self-diagnostics to check what's available
bash osint/scripts/diagnose.sh
```

## Requirements

### Required

| Tool | Purpose | Install |
|------|---------|---------|
| **curl** | HTTP requests to APIs | Pre-installed on macOS/Linux |
| **python3** | JSON parsing, MCP client | Pre-installed on macOS/Linux |
| **jq** | JSON processing | `brew install jq` / `apt install jq` |

### For Apify actors (55+ platforms)

| Tool | Purpose | Install |
|------|---------|---------|
| **Node.js 18+** | Runs `run_actor.js` (embedded Apify runner) | [nodejs.org](https://nodejs.org) |

### Optional

| Tool | Purpose | Install |
|------|---------|---------|
| **mcpc** | Dynamic actor discovery in Apify Store | `npm install -g @apify/mcpc` |

## API Keys & Services

The skill uses **graceful degradation** — the more API keys you provide, the deeper it can dig. You need **at least one** search API to get started.

### Free Tier

| Service | Env Variable | What It Does | Get It |
|---------|-------------|--------------|--------|
| **Brave Search** | _(built into Claude Code)_ | 2,000 queries/month, basic web search | Built-in, nothing needed |
| **Jina AI** | `JINA_API_KEY` | URL → markdown reader, search, deepsearch | [jina.ai/api-key](https://jina.ai/api-key) |
| **Apify** | `APIFY_API_TOKEN` | Instagram, TikTok, YouTube, LinkedIn scraping. Free tier ~$5/month | [console.apify.com](https://console.apify.com/account/integrations) |
| **Parallel AI** | `PARALLEL_API_KEY` | AI-powered search with reasoning and citations | [platform.parallel.ai](https://platform.parallel.ai) |

### Paid (recommended)

| Service | Env Variable | What It Does | Cost | Get It |
|---------|-------------|--------------|------|--------|
| **Perplexity API** | `PERPLEXITY_API_KEY` | Sonar (fast AI answers), Deep Research | ~$5/month | [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) |
| **Exa AI** | `EXA_API_KEY` | Semantic search, people/company research | ~$5/month | [dashboard.exa.ai](https://dashboard.exa.ai) |
| **Tavily** | `TAVILY_API_KEY` | Agent-optimized search, $0.005/request basic | ~$5/month | [app.tavily.com](https://app.tavily.com/home) |

### Advanced

| Service | Env Variable | What It Does | Cost | Get It |
|---------|-------------|--------------|------|--------|
| **Bright Data** | `BRIGHTDATA_MCP_URL` | CAPTCHA bypass, authwall bypass, Facebook scraping, Yandex search | ~$10/month+ | [brightdata.com/mcp](https://brightdata.com/products/web-scraper/mcp) |

### Setting Up Keys

**Option 1 — Environment variables (recommended):**
```bash
export PERPLEXITY_API_KEY="pplx-..."
export EXA_API_KEY="exa-..."
export APIFY_API_TOKEN="apify_api_..."
export JINA_API_KEY="jina_..."
export TAVILY_API_KEY="tvly-..."
export PARALLEL_API_KEY="..."
export BRIGHTDATA_MCP_URL="https://mcp.brightdata.com/..."
```

**Option 2 — File fallback** (supported by some scripts):
```
<workspace>/scripts/apify-api-token.txt
<workspace>/scripts/jina-api-key.txt
<workspace>/scripts/parallel-api-key.txt
<workspace>/scripts/brightdata-mcp-url.txt
```

## How It Works

### Research Phases

```
Phase 0: Tooling Self-Check      → diagnose.sh, check available tools
Phase 1: Seed Collection         → parallel search across all engines
Phase 1.5: Internal Intelligence → Telegram, email, vault (BEFORE external sources)
Phase 2: Platform Extraction     → LinkedIn, Instagram, Facebook, TikTok, YouTube...
Phase 3: Cross-Reference         → facts verified, graded A/B/C/D
Phase 4: Psychoprofile           → MBTI, Big Five, communication style
Phase 5: Completeness Check      → 9 mandatory checks + Depth Score 1-10
Phase 6: Dossier Output          → formatted dossier from template
```

### Research Escalation (cheap → expensive)

```
Level 1: Quick Answers      → Perplexity Sonar, Brave, Tavily, Exa    (~$0.00)
Level 2: Source Verification → Jina read, Parallel extract             (~$0.01)
Level 3: Social Media        → Apify scrapers, Bright Data             (~$0.01-0.10)
Level 4: Deep Research       → Perplexity Deep, Exa Deep, Jina Deep   (~$0.05-0.50)
```

### Embedded Scripts

| Script | Purpose |
|--------|---------|
| `diagnose.sh` | Self-diagnostics for all tools and APIs |
| `perplexity.sh` | search / sonar / deep research |
| `tavily.sh` | search / deep / extract |
| `exa.sh` | search / company / people / crawl / deep |
| `first-volley.sh` | Parallel search across all engines at once |
| `merge-volley.sh` | Deduplicate and group search results |
| `apify.sh` | LinkedIn / Instagram / any actor / store search |
| `run-actor.sh` | Universal Apify runner (55+ actors, polling, CSV/JSON export) |
| `run_actor.js` | Node.js engine powering run-actor.sh |
| `jina.sh` | read URL / search / deepsearch |
| `parallel.sh` | search / extract |
| `brightdata.sh` | scrape / search / search-geo / search-yandex |
| `mcp-client.py` | Lightweight MCP client for Bright Data (stdlib only) |

## Project Structure

```
osint/
├── SKILL.md                          # Main skill file (452 lines)
├── references/
│   ├── tools.md                      # Full catalog of 55+ Apify actors + all tools
│   ├── platforms.md                  # Platform-specific extraction guide
│   ├── content-extraction.md         # YouTube/podcast/blog extraction
│   └── psychoprofile.md              # MBTI/Big Five methodology
├── assets/
│   └── dossier-template.md           # Output dossier template
└── scripts/
    ├── diagnose.sh                   # Self-check
    ├── run-actor.sh                  # Universal Apify runner (bash wrapper)
    ├── run_actor.js                  # Apify runner engine (Node.js, embedded)
    ├── package.json                  # ESM support for run_actor.js
    ├── apify.sh                      # Apify shortcuts
    ├── perplexity.sh                 # Perplexity API
    ├── tavily.sh                     # Tavily API
    ├── exa.sh                        # Exa AI API
    ├── jina.sh                       # Jina AI API
    ├── parallel.sh                   # Parallel AI API
    ├── brightdata.sh                 # Bright Data MCP
    ├── mcp-client.py                 # MCP client (Python, stdlib only)
    ├── first-volley.sh               # Parallel first search
    └── merge-volley.sh               # Result merging
```

## Known Issues (Beta)

- **Shell injection**: user input is interpolated into JSON without `jq` escaping. Do not run with untrusted input.
- **macOS**: `first-volley.sh` uses `tail --pid` (Linux-only). Parallel searches work on macOS, but timeout logic may not trigger.
- **Apify actors**: actor IDs can change or get removed without notice. Use `apify.sh store-search` to find alternatives.
- **Inconsistent key loading**: Perplexity, Tavily, and Exa only load from env vars (no file fallback, unlike Apify/Jina/Parallel).

## Credits

- **Apify Actor Runner** (`run_actor.js`) embedded from [apify/agent-skills](https://github.com/apify/agent-skills) (MIT License)
- Actor catalog based on [apify-ultimate-scraper](https://github.com/apify/agent-skills/tree/main/skills/apify-ultimate-scraper) v1.3.0

## License

[MIT](LICENSE)
