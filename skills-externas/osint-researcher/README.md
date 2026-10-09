# osint-researcher - Claude Code Agent Skill

An Agent Skill **for Claude** (Claude Code / claude.ai). **Target LLM: Claude.**
Conduct **authorized** open-source intelligence and turn lawfully-accessible open sources into verified, graded, decision-useful intelligence: reconnaissance/footprinting for pentest and red-team engagements, cyber threat intelligence, corporate due diligence / KYC, fraud and brand investigation, journalism/research, and assessing your own org's exposure and attack surface.

**→ The skill lives in [`skills/osint-researcher/`](skills/osint-researcher/).** Copy that folder into your `~/.claude/skills/` directory.

## What it covers

- **Legal / ethics / OPSEC gate** - "public does not mean permitted", red lines, GDPR / FR Code penal / CFAA, the breach-data boundary, attribution hygiene, evidence chain-of-custody, TLP.
- **Methodology** - the intelligence cycle, the intrusiveness ladder (passive / semi-passive / active), Admiralty source grading (A-F x 1-6), Bellingcat-style verification, ACH and bias mitigation, estimative language, a report template.
- **Collection disciplines** - per-target playbooks: domains/DNS, IP/infra, web, people/SOCMINT, email/breach, phone, company (incl. registries), images/GEOINT, documents/metadata, archives, and dark-web awareness.
- **Tools catalog** - ~100 tools categorized (passive / active / access) plus Google and GitHub dorking and Shodan/Censys/FOFA/crt.sh query syntax.
- **Collection layer** - optional routing of every search/scrape/crawl/extract through connected tools (Firecrawl, SerpApi, Tavily, ZenRows, Scrape.do, Browserless, Exa) with a tool-selection matrix and an anti-bot escalation ladder, for OPSEC and reach.
- **Threat intelligence** - IOCs and the Pyramid of Pain, ATT&CK / Diamond / Kill Chain, actor profiling, infrastructure pivoting, CTI sources, defensive monitoring, TIBER-EU / DORA context.

## Authorized, ethical use only

This skill is scoped to **lawful OSINT and defensive security** (authorized security testing, threat intelligence, due diligence, investigations, journalism, and own-footprint assessment). It is passive-first and refuses doxxing, stalking, harassment, unauthorized access or circumvention, and any collection meant to enable harm. Before any collection it requires a confirmed purpose, scope, and lawful basis (written Rules of Engagement for pentest/red-team work). See [`skills/osint-researcher/references/legal-ethics-opsec.md`](skills/osint-researcher/references/legal-ethics-opsec.md).

The connected search/scraping tools are optional: if configured they serve as the compliant collection layer (keeping the investigator's own IP off the target); if not, the methodology, tool catalog, and tradecraft stand on their own.

---

Part of the **[mr-bridge.com](https://mr-bridge.com)** toolkit for scraping, data, and content automation:
[Scrapers](https://mr-bridge.com/scrapers) · [MCP servers](https://mr-bridge.com/mcp-servers) · [AI workflows](https://mr-bridge.com/ai-workflows) · [Studies](https://mr-bridge.com/studies) · [Articles](https://mr-bridge.com/articles) · [Solutions](https://mr-bridge.com/solutions)

---

*Part of the [MrBridge Agent Skills catalog](https://github.com/MrBridgeHQ/skills). Browse them all at [mr-bridge.com/skills](https://mr-bridge.com/skills).*
