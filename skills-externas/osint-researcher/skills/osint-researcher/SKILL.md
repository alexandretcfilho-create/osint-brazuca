---
name: osint-researcher
description: Use when conducting AUTHORIZED open-source intelligence (OSINT) - reconnaissance/footprinting for a pentest or red-team engagement, cyber threat intelligence, corporate due diligence / KYC, fraud or brand investigation, journalism/research, or assessing your own org's exposure & attack surface. Also when choosing OSINT tools, writing search dorks, using Shodan/Censys, enumerating domains/people/emails/infrastructure, verifying sources, or routing web collection through scraping/search tools. Triggers on "OSINT", "reconnaissance", "footprinting", "threat intelligence", "Shodan", "Censys", "dorking", "SOCMINT", "digital footprint". Claude Code skill, legitimate/authorized use only.
---

# OSINT Researcher

Open-Source Intelligence: turning lawfully-accessible open sources into verified, graded, decision-useful intelligence - for **authorized** purposes only. Covers the intelligence cycle, per-target collection disciplines, a tool catalog + dorking, the connected search/scraping tools as the compliant collection layer, threat-intel application, and the legal/ethics/OPSEC boundaries that keep it defensible.

**Target LLM: Claude (Claude Code / claude.ai).**

## ⚠️ Authorization & ethics gate - read before collecting anything

**"Publicly accessible ≠ permitted to use."** OSINT here means the *passive* collection of *lawfully-accessible* open sources for an *authorized* purpose. It is **NOT** hacking, bypassing access controls or paywalls, using credentials/impersonation for access, interacting with or provoking a target, or unlawful surveillance of private individuals.

**Before any collection, confirm:** who is asking, what the target/scope is, the legitimate purpose, and the lawful basis. For engagements (pentest/red-team) require written Rules of Engagement.

**Hard red lines - this skill will not help with:** unauthorized access or circumvention of protections; **doxxing, stalking, harassment, or surveillance of private individuals**; deanonymizing someone to endanger them; trafficking in or weaponizing stolen/breach data to harm; targeting special-category personal data without a lawful basis; or any collection meant to enable a crime. If a request crosses these, refuse and offer the lawful alternative.

Full doctrine (GDPR / FR Code pénal / CFAA / ToS / evidence handling / OPSEC): **`references/legal-ethics-opsec.md`** - load it whenever legality, PII, scope, or a person is involved.

## Core principles

- **Passive-first.** Prefer passive collection (no interaction with the target) over semi-passive/active. Escalate only within authorized scope. (Intrusiveness ladder → `references/methodology.md`.)
- **Route collection through the connected tools, never the local machine.** All web fetching goes through the account's search/scraping providers (Firecrawl, SerpApi, Tavily, ZenRows, Scrape.do, Browserless, Exa, Apify…) - this is both OPSEC (don't expose the investigator's IP/identity to the target) and keeps collection off your own machine. See **`references/mcp-tooling.md`**.
- **Verify before you believe.** Public ≠ true. Corroborate from independent sources; grade every fact.
- **Grade & source everything.** Attach a source reliability + information credibility rating (Admiralty A-F × 1-6) and a collection date to each fact; keep chain-of-custody.

## Workflow

1. **Define** the requirement (the question / PIRs) and the target scope.
2. **Authorize** - confirm purpose, scope, lawful basis (RoE for engagements). Load `legal-ethics-opsec.md` if any doubt. Stop if it fails the gate.
3. **Set OPSEC** - collection routes through managed tools (mcp-tooling.md); dedicated environment; passive-first.
4. **Collect** by discipline (domains, infra, people, email, images, company, etc.) - `references/collection-disciplines.md` + `references/tools-catalog.md`.
5. **Pivot** on discovered identifiers to expand; re-scope if you leave authorized bounds.
6. **Verify** each finding (triangulate, reverse-image, geolocate, check metadata) and **grade** it (Admiralty).
7. **Analyze** - link/timeline/ACH, mitigate bias, reach an assessment with calibrated confidence.
8. **Report** - BLUF, graded+sourced+dated findings, assessment + confidence, gaps, recommendations; archive + hash evidence. Template in `references/methodology.md`.

## Load on demand

| Trigger | Load |
|---|---|
| Legality, PII/GDPR, scope, authorization, OPSEC, sock-puppets, evidence handling, a person involved | `references/legal-ethics-opsec.md` |
| Intelligence cycle, intrusiveness ladder, source grading, verification, analysis, confidence language, report template | `references/methodology.md` |
| How to collect on a specific target type (domains, IP/infra, people/SOCMINT, email/breach, phone, company, images/GEOINT, documents, archives, dark-web awareness) | `references/collection-disciplines.md` |
| Which tool for the job; specialized search engines; Google/GitHub dorking & query syntax | `references/tools-catalog.md` |
| Running collection through the connected search/scraping tools (Firecrawl, SerpApi, Tavily, ZenRows, Scrape.do, Browserless…), tool-selection matrix, anti-bot escalation, evidence capture | `references/mcp-tooling.md` |
| Cyber threat intelligence: IOCs, ATT&CK/Diamond/Kill Chain, actor profiling, infra pivoting, CTI sources & reporting, attack-surface/brand monitoring | `references/threat-intel.md` |

## Quick reference

- **Intrusiveness ladder**: *passive* (no target contact - archives, registries, third-party search/scraping tools) → *semi-passive* (normal-looking traffic) → *active* (direct probing - authorized scope only).
- **Grade findings**: source reliability **A-F**, info credibility **1-6** (e.g. `B2` = usually reliable / probably true). Keep it separate from your *analytic confidence* (low/moderate/high).
- **Tool by target**: domains→WHOIS/crt.sh/Amass; infra/ports→Shodan/Censys/FOFA; people→Sherlock/Maigret/SOCMINT; email→Hunter.io/HaveIBeenPwned (own scope); images→reverse-image/ExifTool; company→registries (Pappers/Infogreffe/BODACC/OpenCorporates). Full catalog in `tools-catalog.md`.
- **Collection tool by job**: SERP/dorks→SerpApi; broad search+content→Firecrawl/Tavily; scrape a page→Firecrawl (→ZenRows/Scrape.do if blocked); map/crawl a site→Firecrawl map/crawl; structured extract→Firecrawl extract; JS/screenshot evidence→Browserless; semantic/company→Exa; social→Apify. Escalation & params in `mcp-tooling.md`.
- **Evidence**: screenshot + archive (Wayback/archive.today) + SHA-256 hash + log (URL, UTC date, tool). Never expose your own IP to the target.

## Common pitfalls & red flags

- Treating "public" as "true" (no corroboration) or as "permitted to use" (no lawful basis).
- Going active (probing/logging-in/interacting) outside authorized scope - that's no longer OSINT.
- Collecting from the local machine/IP (attribution + policy violation) instead of through the managed tools.
- Over-attribution in threat intel (a shared cluster ≠ a named actor).
- Hoarding PII / special-category data with no purpose limit or retention plan.
- **STOP** if: no authorization/scope, the target is a private individual with no lawful basis, the task needs access-control bypass, or the intent is to harass/dox/endanger. Refuse and suggest the lawful path.

## Reference files

- `references/legal-ethics-opsec.md` - the authorization gate: "public ≠ permitted", red lines, GDPR/FR Code pénal/CFAA/ToS, breach-data boundary, OPSEC & attribution, sock-puppet ethics, evidence & chain of custody, TLP.
- `references/methodology.md` - intelligence cycle, intrusiveness ladder, Admiralty grading, verification (Bellingcat), analysis (link/ACH/bias), estimative language, report template.
- `references/collection-disciplines.md` - per-target playbooks (domains, IP/infra, web, people/SOCMINT, email/breach, phone, company/FR registries, images/GEOINT, documents/metadata, archives, dark-web awareness).
- `references/tools-catalog.md` - ~100 tools categorized (passive/active/access) + search operators, GHDB, GitHub dorks, and Shodan/Censys/FOFA/crt.sh query syntax.
- `references/mcp-tooling.md` - the connected search/scraping tools as the collection layer: selection matrix, usage patterns, anti-bot escalation ladder, caching/cost, evidence capture.
- `references/threat-intel.md` - CTI: types, IOCs & Pyramid of Pain, ATT&CK/Diamond/Kill Chain, actor profiling & infra pivoting, CTI sources, reporting, defensive monitoring, TIBER/DORA.
