# OSINT Tools Catalog & Search-Operator Reference

> Loaded on demand. This file is the **tool/source directory** and the **query / dorking
> reference** for the `osint-researcher` skill. It answers: *which tool for which identifier,
> is it passive or active, does it need an API key, and what is the exact query syntax?*
>
> **Framing - read before using anything here.** Every tool and every query below is for
> **legitimate, authorized, defensive, consent-based** work: mapping and hardening *your own*
> attack surface, bug-bounty strictly within a published scope, red-team / pentest engagements
> under a signed authorization, sanctioned threat intelligence, investigative journalism,
> academic research, authorized missing-persons work, and privacy self-assessment. Pure
> exploit / attack / malware / intrusion tooling is deliberately **excluded** - this catalog is
> recon + defensive awareness only. The boundary is **intent and authorization, not the tool**:
> the same search engine serves a defender and an attacker, so the gate is set elsewhere.
>
> **Two hard rules that override everything in this file:**
> 1. **Run internet-facing recon through the connected managed tools, never from the local
>    machine.** No `curl`/`wget`/`whois`/`dig`/`nmap`/CLI scanner is ever pointed at a third-party
>    target from this host. CLI syntax below is documented for *understanding and portability*
>    and for use inside an authorized isolated environment - not for local execution. See
>    **`mcp-tooling.md`** for how to actually run these lookups (Firecrawl, SerpApi, Tavily, Exa,
>    Apify Actors, etc.).
> 2. **"Publicly accessible" ≠ "permitted to use," and "public" ≠ "true."** Accessibility confers
>    no authorization; availability confers no accuracy. Legal basis, PII/GDPR, ToS, sock-puppet
>    policy, evidence handling and investigator OPSEC live in **`legal-ethics-opsec.md`** - that
>    file is the gate; this file is only the toolbox. Analytic method (cycle, Admiralty grading,
>    verification, reporting) is in **`methodology.md`**.

---

## 0. How to read the tables

| Column | Meaning |
|--------|---------|
| **Tool** | Canonical name (and home domain / repo where useful). |
| **What it does** | One-line capability. |
| **P/A** | **Passive** = queries a third party's index/dataset; no packet reaches the target - stealthy, needs no target-side authorization to *run* (acting on findings still does). **Active** = touches the target directly (live DNS, port checks, crawling, rendering) - requires explicit in-scope authorization. **Mixed** = has both modes; **default to the passive mode**. |
| **Access** | **Free** / **API key** (needs a registered key; flagged so you provision it) / **Paid** (meaningful capability is behind payment) / **Freemium** (usable free, gated at volume). |
| **Notes** | Defensive framing, gotchas, where it fits the workflow. |

**"Passive" caveat for engines like URLScan / Shodan:** the *scanner* touched the target, historically
and on shared infrastructure - **you** query only its stored index, so from the investigator's IP the
lookup is passive. That is exactly why they are preferred over you scanning anything yourself.

---

## A. TOOL CATALOG

### A.1 Specialized / "hacker" search engines (internet-wide device, service & certificate indexing)

These index **hosts, ports, service banners, TLS certificates, and technologies** - not web pages.
Defensive value: *if you can discover your exposed assets before an attacker does, you secure them
first.* Point them at **your own** IP ranges / domains / org name for attack-surface management.

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **Shodan** (shodan.io) | The canonical device search engine: servers, routers, IoT, cameras, databases, ICS/SCADA, cloud, indexed by service banner, with history. | Passive | Freemium + **API key** | Finds exposed RDP, open Elasticsearch/Redis/MongoDB, Jenkins, Grafana, MQTT, SCADA. Query language in §B.4. CLI (`shodan search`) exists - run via managed tooling, not locally. |
| **Censys** (search.censys.io) | Internet-wide asset + **certificate** intelligence; deep TLS/CT focus, asset attribution, cloud exposure, subdomain discovery. | Passive | Freemium + **API key** | Complements Shodan (certs vs banners). Best for mapping an org's infrastructure and tracking phishing infra. |
| **FOFA** (en.fofa.info) | Cyber-asset discovery via **web/service fingerprints** (domains, IPs, tech). Strong in Asia. | Passive | Freemium + **API key** | Excellent tech fingerprinting; surfaces admin panels. Query syntax in §B.5. |
| **ZoomEye** (zoomeye.ai) | Shodan analog: host discovery, service ID, IoT/ICS analysis, global mapping. | Passive | Freemium + **API key** | Fast infrastructure enumeration; good coverage where Shodan is thin. |
| **Netlas** (netlas.io) | Internet-scan engine unifying hosts, certificates, DNS, WHOIS in one query. | Passive | Freemium + **API key** | Shodan/Censys alternative; handy combined host+cert+DNS view. |
| **BinaryEdge** (binaryedge.io) | Internet-scan data platform: exposed services, open ports, risk scoring. | Passive | Paid + **API key** | Commercial scan data for continuous ASM. |
| **GreyNoise** (greynoise.io) | Internet-background-noise intelligence - distinguishes *targeted* activity from mass/random scanning. | Passive | Freemium + **API key** | Answers "is this IP targeting us or spraying everyone?" - cuts SOC alert fatigue. Also a threat-intel source (§A.8). |
| **LeakIX** (leakix.net) | Indexes *exposed* data: open DBs, misconfigured services, open panels, ransom notes. | Passive | Freemium + **API key** | Check whether **your** infra shows up as exposed (open Redis/Elasticsearch/NAS, backup leaks). |
| **FullHunt** (fullhunt.io) | Attack-surface intelligence: external exposure monitoring, subdomain discovery, cloud-asset visibility. | Passive | Freemium + **API key** | Continuous external ASM for your own perimeter. |
| **Netcraft** (sitereport.netcraft.com) | Site report: hosting history, IP, backend/web-server estimate from Netcraft's global scans. | Passive | Free | Quick hosting-history + tech snapshot for a domain. |
| **Onyphe** (onyphe.io) | Cyber-defense search engine: scan data, threat intel, passive DNS, exposure. | Passive | Freemium + **API key** | European alternative; good for exposure + pivoting. |
| **CriminalIP** (criminalip.io) | Attack-surface / IP-reputation search engine (Shodan-class). | Passive | Freemium + **API key** | Asset discovery + risk scoring. |
| **Intelligence X** (intelx.io) | Deep-data search: historical data, leaks, pastes, public records, documents, dark-web references; preserves snapshots. | Passive | Freemium + **API key** | Cross-cuts breach + archive + dark web. Use for **your own** exposure monitoring. |
| **WiGLE** (wigle.net) | Crowdsourced wireless map: WiFi SSIDs, Bluetooth, cell towers with geolocation. | Passive | Free (account) + **API key** | Wireless audits, physical-security assessment, geolocation context. |

> Traditional **Google / Bing dorking** engines belong to this family too - see **§B.1-B.3**.

### A.2 Domain / DNS / infrastructure recon

Discover subdomains, DNS records, historical infrastructure, and the full attack surface of a domain.

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **WHOIS / RDAP** (who.is; RDAP endpoints) | Registration record: registrant (often redacted), registrar, name servers, dates, **abuse contact**. | Passive | Free | First question: "who owns this?" RDAP is the structured, JSON successor to legacy WHOIS. Capture the registrar **abuse contact** for takedowns. |
| **host / dig** (CLI) | Basic DNS lookups: A/AAAA, MX, TXT (SPF/DKIM/DMARC/verification tokens), NS, SOA; reverse `dig -x`; zone-transfer attempt `dig axfr`. | **Active** (live DNS) - low-noise, ubiquitous | Free | Direct DNS queries touch resolvers → run via managed tooling, not this host. `dig _dmarc.target TXT`; `p=none/quarantine/reject` reading. |
| **DNSDumpster** (dnsdumpster.com) | Visual DNS map: subdomains, mail servers, network infra. | Passive | Free | Great free first-pass subdomain + infra picture. |
| **Amass** (OWASP; github.com/owasp-amass/amass) | Most powerful subdomain-discovery engine. **Passive mode** scrapes CT logs, search engines, passive DNS. | Mixed - **use `amass enum -passive -d target`** | Free (some data sources need **API keys**) | Active mode adds brute-force/resolution and touches DNS; stay passive to leave no trace. |
| **subfinder** (ProjectDiscovery) | Fast passive subdomain enumeration from 50+ public sources/APIs. | Passive | Free (richer with **API keys**) | Pairs with Amass; canonical pipeline `subfinder -silent | httpx`. |
| **Sublist3r** (github.com/aboul3la/Sublist3r) | Python subdomain finder scraping search engines + public datasets. | Passive - **avoid `-b` brute-force to stay passive** | Free | Lightweight complement to Amass. |
| **assetfinder / findomain / ctfr** | Additional passive subdomain sources (assetfinder `--subs-only`, CT-log based). | Passive | Free | Combine and dedupe with the above; more sources = better coverage. |
| **crt.sh** (crt.sh) | Search Certificate Transparency logs. `%.example.com` surfaces subdomains, internal naming, dev/staging. | Passive | Free (public; also SQL) | Certs leak `dev.`/`api.`/`admin.`/`staging.` that appear nowhere else. Query in §B.6. Also finds *related* domains via shared certs. |
| **certspotter / CertStream** | CT-log APIs / real-time CT stream (certstream.calidog.io) for continuous new-cert alerting. | Passive | Free / **API key** | Monitor issuance for **your** brand and lookalikes in real time. |
| **SecurityTrails** (securitytrails.com) | DNS history, WHOIS history, subdomain enumeration, passive DNS. | Passive | Freemium + **API key** | Historical DNS tracks infra changes, forgotten assets, phishing domains. |
| **PassiveTotal / RiskIQ, RapidDNS** | Passive-DNS history and resolution pivoting. | Passive | Freemium + **API key** | Historical resolution → related infrastructure (attribution pivoting). |
| **ViewDNS.info** (viewdns.info) | UI for registration/DNS lookups; infer shared vs dedicated IP. | Passive | Free | Handy for domain-history investigation. |
| **BGP.HE.net / BGPView** | ASN discovery; map an org's IP ranges/CIDRs from its ASN. | Passive | Free / **API key** | `AS`→CIDR ranges → feed to Shodan `net:` for range-wide exposure. |
| **BuiltWith** (builtwith.com) | Detailed website tech-stack profile: analytics, frameworks, ad/partner tags. | Passive | Freemium + **API key** | Shared analytics/ad IDs link related sites (network attribution). |
| **Wappalyzer** (browser ext / API) | Passively identifies a site's technologies as rendered. | Passive | Freemium + **API key** | Real-time per-page tech fingerprint; API for bulk. |
| **URLScan.io** (urlscan.io) | Renders a URL in a sandbox and records DNS/HTTP, JS, screenshots, third-party calls, behavior. | Active - **but URLScan does the fetch, not you** | Freemium + **API key** | Ideal for phishing/malware/domain analysis; the fetch happens on urlscan's infra, protecting your IP. Note: public scans are searchable by others. |

### A.3 SOCMINT / people & username footprinting

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **Sherlock** (github) | Hunts a **username** across 400+ social networks. | Passive | Free | Fast username-enumeration standard; `--csv` output. Verify hits manually (false positives happen). |
| **Maigret** (github) | Sherlock successor - username across ~2,500 sites + profile-data extraction; `--pdf --html`. | Passive | Free | Richer output, broader coverage than Sherlock. |
| **WhatsMyName** (whatsmyname.app) | Username enumeration via a maintained web app + JSON site list. | Passive | Free | Consistency-check a handle across platforms; the JSON list is the shared backbone many tools use. |
| **Blackbird / Social-Analyzer** | Additional username coverage + profile analysis. | Passive | Free | Cross-check to raise confidence. |
| **LinkedIn (public / dorks)** | Enumerate employees, roles, email-naming convention, tech stack (via listed skills). | Passive (manual / dork) | Free (Sales Navigator paid) | Richest single org-chart source. Dork: `site:linkedin.com/in "Company" "role"`. Also the social-engineering pretext vector - defensive awareness. |
| **Epieos** (epieos.com) | Reverse **email/phone** lookup → linked Google account, reviews, socials. | Passive | Freemium | Strong pivot from an email to an identity graph. |
| **GHunt** (github) | Enumerate public info tied to a Google account. | Passive | Free (**cookies/session** needed) | Google-account footprinting; handle the session credential carefully. |
| **Public social platforms** (X, Facebook, Instagram, Reddit, TikTok, Telegram, GitHub, GitLab) | Public posts, connections, geotags, employer/role data. | Passive | Free | Raw SOCMINT sources; combine handles + breach checks into a profile. X/Facebook often strip EXIF on upload. |

> No-login viewers (Picuki/Imginn etc.) exist but their ToS/legality shift constantly - check
> `legal-ethics-opsec.md` before relying on them, and never use a **false identity to reach
> non-public content** (impersonation gate).

### A.4 Email, breach & credential-exposure intelligence

**Defensive framing:** monitor **your own organization's** exposure - confirm whether corporate
emails/passwords appear in breach corpora, then force resets. Unauthorized use against others can
breach privacy law; **never** use recovered credentials to access anything (that leaves OSINT and
becomes intrusion). See `legal-ethics-opsec.md`.

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **Hunter.io** (hunter.io) | Discovers professional emails for a domain; reveals the email-format pattern; verifier. | Passive | Freemium + **API key** | Maps exec/support/security contacts; format → permutations (`first.last@`, `f.last@`, `firstl@`). |
| **Have I Been Pwned (HIBP)** (haveibeenpwned.com) | Checks whether an email/password appeared in known breaches, pastes, stuffing lists; domain search. | Passive | Free (search) + **API key** (domain/API) | By Troy Hunt. Domain monitoring of employee exposure; credential reuse is a top risk. |
| **DeHashed** (dehashed.com) | Indexes leaked credentials/breach data: email, username, domain monitoring. | Passive | Paid + **API key** | **Ethics flag:** authorized/defensive/threat-intel only. Do not act on the passwords - monitor and rotate. |
| **holehe** (github) | Checks whether an email is registered on 120+ sites via forgot-password/registration flows, **without alerting the owner**. | Passive / semi-active | Free | Account-discovery standard; `holehe target@email --only-used`. |
| **h8mail** (github) | Domain-wide breach/leak hunting across multiple sources. | Passive | Free (some sources need **API keys**) | Bulk email→breach mapping for your own domain audit. |
| **Leak-Lookup / LeakCheck / LeakPeek / Snusbase** | Compromised-credential databases (search by email/username/domain). | Passive | Paid + **API key** | Defensive exposure monitoring only; same ethics flag as DeHashed. |
| **SpyCloud / Flare.io** | Commercial breach + darknet exposure monitoring for your org. | Passive | Paid + **API key** | Enterprise ASM/credential-monitoring; feeds SOC. |
| **MXToolbox Email Header Analyzer** | Parses `Received:` chain, SPF/DKIM/DMARC alignment, spoofing indicators. | Passive | Free | Email-header investigation (originating IP = bottom-most `Received`). |
| **Breach-corpora awareness (COMB, pastes)** | Large aggregated credential dumps referenced by the above. | n/a (awareness) | - | Know they exist; touch them **only** via legitimate services above, for authorized exposure monitoring. |

### A.5 Phone-number intelligence

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **PhoneInfoga** (github) | Phone-number OSINT: carrier, line type, region/country, VoIP detection, search-engine footprint. | Passive | Free (some lookups need **API keys**) | `phoneinfoga scan -n "+…"`. VoIP flag (Google Voice/TextNow/Twilio/burner) is a scam indicator. |
| **`phonenumbers` (libphonenumber)** | Library: validation, carrier (`carrier.name_for_number`), geocoding. | Passive | Free | Offline metadata; no network calls to the number's owner. |
| **Epieos (phone)** | Reverse phone → linked accounts. | Passive | Freemium | Pivot from number to identity. |
| **Truecaller / reverse-lookup directories** | Caller-ID / reverse lookup cross-reference. | Passive | Freemium | Corroborate name↔number; treat as low-reliability until confirmed. |

### A.6 Image, metadata & GEOINT

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **ExifTool** (exiftool.org) | Reads EXIF/metadata from images & docs: author/username, software version, sometimes **GPS**. | Passive (on downloaded files) | Free | `exiftool image.jpg`. Run only on lawfully-obtained public media. |
| **Jeffrey's Exif Viewer / Metadata2Go** | Web-based EXIF/metadata viewers. | Passive | Free | Quick browser check when a local tool isn't available (mind the "service, not tool" data-handling caveat - `legal-ethics-opsec.md`). |
| **Metagoofil** (github) | Harvests **public documents** (PDF/DOC/XLS/PPT) from a domain and extracts their metadata. | Passive | Free | Surfaces authors/usernames/software/internal paths leaked in doc metadata. `-t pdf,doc,xls`. |
| **FOCA** | Windows GUI: finds & analyzes document metadata from a target's public files. | Passive | Free | Classic desktop counterpart to Metagoofil. |
| **Reverse image search** - Google Images, **Yandex**, TinEye, Bing | Find where an image appears / originated; identify people, places, objects. | Passive | Free | **Yandex** is strongest on faces/places; TinEye best for exact-copy provenance. Core verification technique. |
| **PimEyes** | Face-recognition reverse search. | Passive | Freemium + Paid | **Legal/ethical use only** - face data is sensitive PII; several jurisdictions restrict it (Clearview precedent). Gate via `legal-ethics-opsec.md`. |
| **Geolocation toolkit** - Google Earth Pro, Street View, Bing/Yandex/Baidu Maps, SunCalc, PeakFinder, Overpass Turbo (OSM) | Chronolocation & geolocation from shadows/sun, landmarks, signage, terrain, OSM features. | Passive (methodology + public imagery) | Free | Global vs local indicators framework; SunCalc = time-of-day from shadows; Overpass Turbo = query OSM for matching features. |
| **Satellite/aerial imagery** - Sentinel/Copernicus, NASA EarthData, Maxar Open Data | Public satellite/aerial imagery for GEOINT. | Passive | Free (some **API keys**) | Change detection, corroboration of ground photos. |
| **InVID / WeVerify, FotoForensics, Forensically** | Video/image verification & manipulation forensics; keyframe extraction. | Passive | Free | Detect edited/AI-generated media; corroborate authenticity. |
| **Octopii** (github.com/redhuntlabs/Octopii) | PII scanner (OCR+regex+NLP) for Gov-IDs/addresses/emails in **public-facing** images/PDFs. | Passive | Free | Defensive: scan **your own** exposed storage for leaked PII. |

### A.7 Web & code intelligence (source-code recon, secret discovery - on YOUR OWN / authorized assets)

**Defensive framing:** audit **your own** repos and buckets for leaked secrets before anyone else finds them.

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **GitHub / GitLab code search** | Search public repos for committed credentials, tokens, internal names, endpoints. | Passive | Free (**account**; API key for volume) | Dorks in §B.7. Run against **your org** or in authorized scope. |
| **grep.app** (grep.app) | Full-text search across massive public source-code corpus. | Passive | Free | Hunt exposed secret patterns (`AWS_SECRET_ACCESS_KEY`, `jwt_secret`) across all of GitHub at once. |
| **TruffleHog** (github) | Scans git history / filesystem / S3 / GitHub org for verified secrets. | Passive (on your repos) | Free (+ paid) | `trufflehog github --org=yourorg --only-verified`. Defensive secret-leak detection. |
| **Gitleaks / Gitrob** | Secret scanning across repos and history. | Passive (on your repos) | Free | CI-friendly; run on **your own** repositories. |
| **waybackurls / gau** | Pull all historical URLs a domain ever exposed (Wayback + sources). | Passive | Free | Mine for old admin panels, `.env`, `.git`, backups, decommissioned endpoints. |
| **httpx** (ProjectDiscovery) | Probe a host list for live web servers, titles, tech, status, ASN. | **Active** (fetches targets) | Free | End of the subdomain pipeline; run only in authorized scope / via managed infra. |
| **WhatWeb** | Web tech fingerprinting (`whatweb -a 3`). | Active | Free | Identify CMS/framework/server; active fetch. |
| **BucketLoot** (github) | Enumerates & reads exposed S3/cloud-bucket textual files, extracting secrets/URLs; JSON output. | Passive | Free | Check **your own** buckets for exposure. |
| **Browser DevTools → View Source** | Read HTML/JS for dev comments, analytics/AdSense IDs, emails, endpoints. | Passive | Free | Shared analytics/AdSense IDs link related sites (network attribution). |

### A.8 Vulnerability & threat intelligence

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **CVE / NVD / CWE** | Canonical public vulnerability & weakness references. | Passive | Free | Baseline: product/version → known CVEs; prioritize remediation. |
| **Vulners** (vulners.com) | Aggregates CVEs, advisories, exploit references, malware. | Passive | Freemium + **API key** | e.g. "Apache Struts" → CVEs + advisories in one view. |
| **VirusTotal** (virustotal.com) | File/URL/IP/domain reputation + relationship pivoting. | Passive | Freemium + **API key** | `vt` CLI; pivot `entity:domain name:*.evil.com`, `behaviour_network:` - infrastructure attribution. Never upload sensitive/confidential files (they become searchable). |
| **AlienVault OTX / AbuseIPDB / Pulsedive** | IOC pulses, IP-reputation, IOC enrichment. | Passive | Free / **API key** | Enrich and corroborate indicators. |
| **GreyNoise** | (see §A.1) targeted-vs-noise context on scanning IPs. | Passive | Freemium + **API key** | Cross-listed: threat-intel + exposure. |
| **ANY.RUN / Hybrid Analysis** | Interactive malware sandboxes (behavioral analysis). | Passive (analysis of samples) | Freemium + **API key** | For CTI on samples you're authorized to analyze; public submissions are shared. |
| **OpenCTI / MISP** | Self-hosted CTI platform / threat-sharing platform (IOC storage, correlation, STIX/TAXII). | Passive | Free (self-host) | Store, correlate, and share IOCs; feed SIEM. TLP-aware sharing (`legal-ethics-opsec.md`). |
| **Recorded Future / Mandiant / Intel 471 / Flashpoint** | Commercial CTI & dark-web monitoring. | Passive | Paid + **API key** | Enterprise threat/exposure intelligence. |

### A.9 Frameworks & automation (aggregators / orchestration)

These orchestrate many tools above into one workflow.

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **Maltego** | Link-analysis / graph tool mapping relationships between people, orgs, domains, emails, socials via **transforms**. | Passive (transform-dependent) | Freemium (CE = 12 results/transform) + Paid | Visual-investigation standard. Transform Hub: Shodan, VirusTotal, HIBP, WhoisXML, PassiveTotal, GitHub. Workflow: seed → transforms → expand → follow → export. |
| **SpiderFoot** (now Intel471) | Automates recon across 200+ sources: domain intel, breach, IP reputation, subdomains, correlation. | Passive (configurable) | Free (local) + Paid (HX cloud) | Web UI on `127.0.0.1:5001`; module classes Footprint/Investigate/Passive. Great for a single aggregated first pass. |
| **theHarvester** | CLI scraping public sources (Google/Bing/LinkedIn/Shodan/crt.sh/…) for emails, subdomains, hosts. | Passive | Free (some sources need **API keys**) | `theharvester -d target -b all`. Fast people+infra footprint. |
| **Recon-ng** | Modular, Metasploit-style recon console with marketplace modules; workspaces. | Passive / Mixed | Free (module data needs **API keys**) | `marketplace install all`; e.g. `recon/domains-hosts/hackertarget`, `reporting/html`. |
| **OSINT Framework** (osintframework.com) | Curated web directory of OSINT tools/resources by identifier type. | n/a (index) | Free | **Living directory** - navigation hub, not a scanner. Verify links (some rot). |
| **IntelTechniques** (Michael Bazzell) | Toolset + methodology + search tools for structured investigations. | n/a (methodology + tools) | Free / Paid (books) | Widely referenced investigative doctrine. |
| **Malfrat's OSINT Map / MetaOSINT / Awesome-OSINT** | Meta-directories / curated tool maps (EU-leaning, aggregators). | n/a (index) | Free | Discovery layer; cross-check freshness. |
| **dnstwist** | Typosquat / lookalike-domain permutation generator with `--registered`/`--mxcheck`/`--ssdeep`. | Mixed (registration checks are lookups) | Free | **Defensive**: find phishing-ready lookalikes of **your** brand. Permutation classes: homoglyph, bitsquatting, transposition, omission, addition. |
| **Kali Linux (information-gathering metapackage)** | Bundles theHarvester/Recon-ng/Maltego/SpiderFoot/Shodan-CLI in one distro. | - | Free | Run in an **isolated VM** for OPSEC + reproducibility (`legal-ethics-opsec.md`). |
| **OSINT VMs** - Trace Labs OSINT VM, CSI Linux, Tsurugi | Purpose-built investigation environments (Trace Labs = missing-persons/SAR). | - | Free | Snapshottable, isolated; keep separate from personal identity. |

### A.10 Archives, capture & investigation utilities

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **Wayback Machine** (web.archive.org) | Historical/cached snapshots - removed pages, old endpoints, deleted content, past ownership. | Passive | Free + **API** | Invaluable for domain-history and "what changed" investigations. |
| **archive.today** (archive.ph) | On-demand immutable snapshot (defeats later edits/deletion). | Passive | Free | Capture a live page as third-party-witnessed evidence. |
| **Perma.cc** | Permanent citation-grade archiving (academic/legal). | Passive | Freemium | Court/academic-grade preservation. |
| **Monolith / CutyCapt / WaybackPy** | Self-contained HTML archive / full-page screenshot / programmatic Wayback submission. | Passive | Free | Evidence pipeline: `monolith` → `cutycapt` → `waybackpy --save` → `sha256sum` manifest. See `methodology.md` chain-of-custody. |
| **Hunchly** | Auto-documents/preserves every page visited during an investigation. | Passive | Paid | Investigator-grade capture with evidence integrity. |
| **Company/leak registries** - OpenCorporates, OCCRP Aleph, Offshore Leaks DB; (FR) Societe.com, Pappers, Infogreffe, BODACC, INPI, Legifrance, Data.gouv.fr | Corporate structure, filings, directors, capital links, IP, procurement, case law. | Passive | Free / Freemium | FININT / due-diligence backbone; primary-source, high-reliability. |
| **SEC EDGAR / PACER (US)** | Corporate filings (10-K/DEF 14A) / federal court records. | Passive | Free (PACER per-page fee) | Exec names, salaries, litigation history. |

### A.11 Dark-web awareness (defensive monitoring only)

> Legitimate scope here is **narrow**: monitoring for **your own** leaked data / brand mentions, or
> sanctioned CTI. Access, ToS, and legality are strict - read `legal-ethics-opsec.md` first. This
> skill does **not** facilitate marketplace transactions or criminal-forum participation.

| Tool | What it does | P/A | Access | Notes |
|------|--------------|-----|--------|-------|
| **Ahmia.fi** (clearnet gateway), OnionSearch, DarkSearch | Search indexed onion content from clearnet / multi-engine. | Passive | Free | Discovery layer; corroborate before trusting. |
| **Intelligence X, DeHashed** | (see §A.1/A.4) cross-cut dark-web/leak references. | Passive | Freemium/Paid + **API key** | Monitor *your own* exposure. |
| **Flare / Recorded Future / Intel 471 / Flashpoint** | Managed dark-web & leak monitoring for your org. | Passive | Paid + **API key** | Outsourced, lower-risk than direct access. |

---

## B. SEARCH OPERATORS & DORKING

> **Frame every dork below as finding exposure of your own / authorized assets.** Replace
> `target.com` / `Your Org` with an asset you are authorized to assess. Dorking itself uses a public
> engine's own features and is lawful; the line is crossed by **exploiting** what you find (e.g.
> using discovered credentials = fraudulent access). See `legal-ethics-opsec.md`.

### B.1 Google / Bing core operators

| Operator | Effect | Example (authorized-asset framing) |
|----------|--------|-----------------------------------|
| `site:` | Restrict to a domain/subdomain | `site:target.com` - everything Google indexes for your domain |
| `-site:` / `-` | Exclude | `site:target.com -site:www.target.com` - non-www subdomains only |
| `filetype:` / `ext:` | Restrict to a file type | `site:target.com filetype:pdf "confidential"` - exposed confidential PDFs |
| `intitle:` / `allintitle:` | Match page title | `site:target.com intitle:"index of"` - open directory listings |
| `inurl:` / `allinurl:` | Match the URL | `site:target.com inurl:login` · `inurl:admin` · `inurl:dashboard` |
| `intext:` / `allintext:` | Match body text | `site:target.com intext:"internal use only"` |
| `"..."` | Exact phrase | `"@target.com" site:linkedin.com` - employees + email pattern |
| `OR` (`|`) | Either term | `site:target.com filetype:env OR filetype:conf` - exposed config files |
| `AND` / space | All terms | `site:target.com password backup` |
| `*` | Wildcard within phrase | `"target * admin panel"` |
| `before:` / `after:` | Date range | `site:target.com after:2024-01-01 before:2025-01-01` |
| `related:` | Similar sites | `related:target.com` |
| `cache:` | Cached copy (deprecated on Google; use Wayback/archive.today) | - |

Bing shares most of these and adds `ip:`, `feed:`, `contains:`. Yandex indexes content Google misses
and has the strongest **reverse-image** search - worth a parallel pass.

### B.2 The Google Hacking Database (GHDB) concept

The **GHDB** (on Exploit-DB) is a curated, categorized collection of thousands of proven dork patterns
that surface exposed files, login portals, error messages, and misconfigurations. Use it **to audit your
own / authorized assets**: pick relevant dork categories (e.g. "Files Containing Juicy Info", "Sensitive
Directories", "Web Server Detection"), scope each with `site:yourdomain`, and remediate what appears
(remove indexed files, add `robots`/auth, rotate any leaked secret). It is a **defensive checklist of how
you are exposed**, not an attack script.

Common exposure-audit patterns (scope with your own `site:`):

```
site:target.com intitle:"index of" ("backup" OR "db" OR ".git" OR ".env")
site:target.com filetype:log
site:target.com filetype:sql "INSERT INTO"
site:target.com inurl:(login | signin | admin | portal | dashboard)
site:target.com intext:"SQL syntax" | "Warning: mysql" | "stack trace"
site:target.com ext:(env | conf | ini | yml | bak | old)
```

### B.3 GitHub / GitLab code-search dorks (your own repos / authorized scope)

Run these to find **your** organization's accidentally-committed secrets before anyone else does.

```
org:your-org "AWS_SECRET_ACCESS_KEY"
org:your-org filename:.env
org:your-org "-----BEGIN RSA PRIVATE KEY-----"
"target.com" "api_key"            # web search: site:github.com "target.com" api_key
"target.com" password language:YAML
"target.com" "authorization: bearer"
```

Cross-reference **grep.app** (searches all public code at once) and **TruffleHog**
(`trufflehog github --org=your-org --only-verified`) to confirm live secrets and rotate them.

### B.4 Shodan query language

Point at **your own** org / IP ranges (get ranges from ASN → `net:`).

```
org:"Your Org Ltd"                     # assets attributed to your org
net:203.0.113.0/24                     # your CIDR (from ASN lookup)
hostname:target.com                    # hosts on your domain
port:22 org:"Your Org Ltd"             # exposed SSH on your estate
ssl.cert.subject.cn:target.com         # certs naming your domain
http.title:"Grafana"                   # exposed Grafana dashboards
product:"MongoDB"  org:"Your Org Ltd"  # exposed databases
vuln:CVE-2021-44228                    # (paid) hosts flagged for a known CVE
"default password"                     # banners advertising defaults
```

Combine with `country:`, `city:`, `os:`, `has_screenshot:true`. Facets (`--facets port`) summarize
exposure across a range.

### B.5 Censys query language

```
services.service_name: HTTP and location.country: France
autonomous_system.organization: "Your Org Ltd"
parsed.names: target.com                       # certs mentioning your domain
services.software.product: "nginx"
services.tls.certificates.leaf_data.subject.common_name: target.com
services.port: 3389                             # exposed RDP
```

### B.6 FOFA query language (fingerprint search)

```
domain="target.com"
host="target.com"
app="Apache-Tomcat"
org="Your Org Ltd"
cert="target.com"
title="Admin" && domain="target.com"
```

### B.7 crt.sh (Certificate Transparency → subdomains)

`crt.sh` is a public CT-log search; `%` is the SQL wildcard (URL-encoded `%25`).

```
# Web UI - all certs whose name matches any subdomain of the domain:
%.target.com

# JSON API (run via managed tooling, not this host):
https://crt.sh/?q=%25.target.com&output=json
#   → parse .[].name_value, split on newlines, sort -u  → subdomain list

# Direct SQL (crt.sh exposes a Postgres endpoint) - ILIKE with % wildcards:
SELECT DISTINCT name_value
FROM certificate_and_identities
WHERE name_value ILIKE '%.target.com';
```

CT is also a **related-domain finder**: a shared certificate (SAN list) links otherwise-unconnected
hostnames - useful for attribution and for spotting shadow IT under your own brand.

### B.8 DNS / WHOIS quick reference (run via managed tooling, never local)

```
whois target.com                     # registrant, registrar, NS, abuse contact
dig target.com ANY                   # (many resolvers restrict ANY)
dig target.com MX                    # mail provider fingerprint (google/outlook/proofpoint…)
dig target.com TXT                   # SPF, DKIM selectors, DMARC, SaaS verification tokens
dig _dmarc.target.com TXT            # DMARC policy: p=none | quarantine | reject
dig -x 203.0.113.10                  # reverse DNS
```

MX/TXT records are a cheap tech-stack tell: MX → mail provider; verification TXT → SaaS in use
(e.g. Google Workspace + Slack + Atlassian). SPF/DMARC policy strength (`p=reject` strongest) is a
defensive-posture signal for **your own** domains.

---

## C. Living-directory note - verify before you rely

Two of the entries above are **living directories**, not fixed tool lists, and are the best way to
find a current tool for an identifier type the tables here don't cover:

- **i-intelligence - *OSINT Tools and Resources Handbook*** - a discipline-organized catalogue of
  several thousand named tools/sources (people, social, phone/email/username, business/FININT,
  domains/WEBINT, dark web, images/facial recognition, geospatial, video/metadata, verification,
  link analysis, frameworks/platforms). Its own foreword warns: *"today's wonder tool may be tomorrow's
  security liability"* - **test any new tool in an isolated environment** before trusting it.
- **OSINT Framework** (osintframework.com) - an interactive tree of tools by identifier type, plus
  **Malfrat's OSINT Map**, **MetaOSINT**, and **Awesome-OSINT** as complementary meta-directories.

**Tools change constantly** - projects are abandoned, APIs close, free tiers vanish, and clone/impostor
sites appear that mimic a real tool's branding to harvest your query data or serve malware. Before
relying on anything:

1. **Verify at the source repo** (GitHub/GitLab), never a glossy presentation site.
2. **Prefer local, open-source tools** over web services - a web "tool" sends *your* query to a third
   party who may log it, which engages *your own* PII/GDPR compliance (the "services, not tools" caveat).
3. **Confirm passive vs active** for the current version - modes and defaults drift between releases.
4. **Confirm the access model** - many former-free tools now gate behind an API key or paywall.
5. **Run it through the managed tooling** described in `mcp-tooling.md`, in an isolated environment -
   never from the local host, never against a target you are not authorized to assess.

> Bottom line: this catalog is a **map, not the territory**. It tells you which class of tool fits which
> job and how to query it; `mcp-tooling.md` tells you how to run it safely; `legal-ethics-opsec.md` tells
> you whether you're allowed to; `methodology.md` tells you what to do with the results.
