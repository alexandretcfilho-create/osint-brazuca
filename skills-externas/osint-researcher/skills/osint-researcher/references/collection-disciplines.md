# Collection Disciplines - Reference

> Loaded on demand. This file is the **per-target collection playbook** of the `osint-researcher`
> skill: for each discipline it states *what you are collecting*, the *key sources and tools*
> (with exact names and an example query where useful), *how to pivot* to the next identifier, and
> the *cautions / legal note*. Scope is **legitimate, authorized, ethical OSINT** only - external
> attack-surface self-assessment, consented due diligence, sanctioned threat intelligence,
> investigative journalism, academic research, authorized missing-persons work, and authorized
> pentest/Red-Team reconnaissance.
>
> Read this alongside the two sibling references it depends on:
> - `methodology.md` - the intelligence cycle, the **intrusiveness ladder** (passive → semi-passive →
>   active), source grading (Admiralty code), verification, pivoting logic, confidence language,
>   reporting. Each discipline below is an instance of that ladder.
> - `legal-ethics-opsec.md` - authorization/scope, the hard red lines, GDPR/PII, ToS, breach-data
>   *awareness-vs-use* boundary, sock-puppet ethics, evidence handling, and the no-local-IP rule.
>   **Every "Caution" line here cross-refers it. When in doubt, that file governs.**
>
> Three rules sit above every technique below:
> 1. **"Public" ≠ "permitted" and "public" ≠ "true."** Accessibility never confers authorization
>    (Clearview AI, Bluetouff); availability never confers accuracy. Grade and corroborate everything
>    (`methodology.md` §3-4).
> 2. **Passive-first.** Start on the passive rung of every discipline; climb to active enumeration only
>    when a lower rung cannot answer the requirement *and* written authorization + scope permit it.
> 3. **Route collection through managed tools - never the local IP.** Any live retrieval must egress
>    through the connected scraping/search MCP tools and managed APIs (Firecrawl, SerpApi, Tavily,
>    Exa, Apify Actors, and the configured providers), never via `requests`/`httpx`/`curl`/browser
>    automation from this workstation. See `mcp-tooling.md` for the tool-to-discipline mapping and
>    `legal-ethics-opsec.md` §6.1.

---

## 0. How to use this file

- Investigations are organized **by identifier type** (OSINT Framework taxonomy): Domain, IP, Web,
  Person, Email, Phone, Company, Image, Document, plus the historical and dark-web awareness layers.
- Each discipline is a node in a **pivot graph**: a finding in one discipline yields identifiers that
  seed the next (domain → IP → org → people → email → breach, etc.). The "How to pivot" line in each
  section names the exits. Keep a link chart of pivots (Maltego / a mind-map) per `methodology.md` §5.
- The **passive/active** tag on each tool follows `methodology.md` §2: *passive* = queries a
  third-party index, no packet touches the target; *active* = interacts directly with target systems
  (needs explicit scope/authorization); *mixed* = has both modes - use the passive mode to stay
  non-intrusive.
- **Managed-tool routing** (`mcp-tooling.md`): a Google dork, a `crt.sh` query, a Shodan lookup, a
  Wayback fetch - all of these should be issued through Firecrawl/SerpApi/Tavily/Exa or a dedicated
  Apify Actor / provider API, not from the local host. The named CLI tools (`amass`, `subfinder`,
  `dig`, `nmap`, `sherlock`, `holehe`, `exiftool`, `phoneinfoga`) are given so you understand the
  technique and its output; when they would send traffic to a third-party target they must run in an
  authorized, non-local environment (Apify Actor, sanctioned lab, or the engagement's own tooling) -
  never from this machine against a live target.

---

## 1. Domains & DNS

**What you are collecting.** Ownership and registration of a domain; its full DNS footprint (A/AAAA,
MX, TXT, NS, SOA, CNAME); every subdomain (including forgotten `dev.`/`staging.`/`api.`/`admin.`
hosts); the IPs and name servers it resolves to; historical registration and DNS changes. This maps
the *organizational perimeter* - often the first node of a domain/company investigation.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **WHOIS / RDAP** (`whois`, RDAP endpoints; who.is) | Passive | Registrant (often redacted), registrar, **registrar abuse contact**, name servers, dates | `whois target.com`; regional registries `whois.arin.net` (Americas), `whois.ripe.net` (Europe), `whois.apnic.net` (APAC) |
| **Historical WHOIS/DNS** - SecurityTrails, DomainTools, WhoisXML, ViewDNS.info, RiskIQ/PassiveTotal | Passive | Registration/DNS history; registrant data that survived later GDPR masking | Track when NS or registrant changed → infra migration, ownership change |
| **`dig` / `host`** | Active (low-noise, ubiquitous) | Live DNS records | `dig target.com ANY`; `dig target.com MX`; `dig _dmarc.target.com TXT`; reverse `dig -x <ip>` |
| **Certificate Transparency** - **crt.sh**, certspotter, CertStream (`certstream.calidog.io`), Facebook CT | Passive | Subdomains + *related* domains leaked in issued certs | `crt.sh` query `%.target.com` (via a managed fetch); returns `dev.`, `vpn.`, internal naming that appears nowhere else |
| **Passive subdomain enum** - **Amass** (`amass enum -passive -d target.com`), **subfinder**, **assetfinder --subs-only**, findomain, DNSDumpster, chaos.projectdiscovery.io | Passive/Mixed | Subdomains aggregated from CT, passive DNS, search engines | Combine outputs, dedupe; Amass/Sublist3r **without** brute-force flags stays passive |
| **Active subdomain enum** - `amass enum -brute -w wordlist`, `gobuster dns`, puredns/shuffledns, altdns (permutations), dnsx (resolve/validate) | Active - **authorized scope only** | Brute-forced/validated subdomains | Only after passive is exhausted and ROE permits |
| **Passive DNS** - SecurityTrails, VirusTotal passive DNS, mnemonic PassiveDNS, RapidDNS | Passive | Historical resolution (domain↔IP over time) → related infrastructure | Pivot to shared-hosting / co-hosted domains |
| **Zone transfer** - `dig axfr @ns1.target.com` | Active | Full zone if misconfigured (rare) | Worth one attempt in-scope; usually refused |

**How to pivot.** Registrant / registrant-org name → **Company** (§7) and reverse-WHOIS for other
domains by the same owner. Name servers / MX → hosting & mail provider (§2, §5). Resolved IPs →
**IP / infrastructure** (§2) and ASN. Subdomains → **Web / attack surface** (§3) via live-host probing.
TXT records (SPF/DKIM/DMARC, verification tokens) → SaaS/IdP in use (a Google/Microsoft/Slack/Okta/
Atlassian verification TXT reveals the vendor stack → phishing-scenario and vendor-exposure intel).

**Cautions / legal note.** WHOIS/RDAP, CT logs, and passive DNS are passive and low-risk. **Active
subdomain brute-force, zone-transfer attempts, and mass resolution are active enumeration** - they
touch or query target-controlled infrastructure and require written authorization + scope
(`legal-ethics-opsec.md` §3, `methodology.md` §2). Historical WHOIS can surface personal data that is
now redacted - treat it as PII (GDPR data-minimization / purpose-limitation, `legal-ethics-opsec.md`
§4). Route CT/passive-DNS/WHOIS lookups through managed tools, not the local host.

---

## 2. IP / network / infrastructure

**What you are collecting.** For an IP, range, or host: open ports, running services and their
software/versions, service banners, TLS/SSL certificates, hostnames, the owning ASN/organization and
BGP announcements, hosting/CDN provider, and whether the host is merely part of internet-wide
background scanning noise. This is *external attack-surface mapping* - done against your own or
in-scope ranges.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Shodan** (shodan.io) | Passive (queries Shodan's scan index) | Exposed services, ports, banners, product/version, screenshots, history | `org:"Target Company"`; `hostname:target.com`; `net:203.0.113.0/24`; `port:22 country:"FR"`; `ssl.cert.subject.cn:target.com`; `product:"MongoDB"`; `vuln:CVE-2023-…` |
| **Censys** (search.censys.io) | Passive | Host + certificate intelligence, asset attribution, cloud exposure | `services.service_name: HTTP`; `parsed.names: target.com`; `autonomous_system.organization:"Target"` |
| **FOFA** (en.fofa.info), **ZoomEye** (zoomeye.ai), **Netlas** (netlas.io) | Passive | Fingerprint/asset search; Shodan/Censys alternates | FOFA `app="Apache-Tomcat"`; Netlas unified host+cert+DNS |
| **LeakIX** (leakix.net), **FullHunt** (fullhunt.io), **BinaryEdge**, **CriminalIP** | Passive | Exposed DBs/panels/backups; continuous external ASM | Check whether *your* infra shows up exposed |
| **GreyNoise** (greynoise.io) | Passive | Is an IP mass-scanning everyone, or targeting you? | Cuts SOC alert-fatigue; enrich any suspicious source IP |
| **ASN / BGP** - bgp.he.net, BGPView API, `whois -h whois.radb.net -- '-i origin AS12345'`, nitefood `asn`, `nmap -sL` | Passive | ASN, announced CIDR ranges, org, abuse contact | `asn <ip|domain>` returns ASN + abuse email; expand to the org's full IP space |
| **Netcraft** (sitereport.netcraft.com) | Passive | Hosting history, backend tech, web-server estimate | Quick hosting/tech snapshot for a domain |
| **`nmap`** (`-sS -sV -O`, `-sC`, `-p`, `-Pn`) | **Active** | Port scan, service/version + OS fingerprint, NSE scripts | `nmap -sS -sV -O target` - **authorized scope only** |

**How to pivot.** IP → ASN → the org's entire announced range → more hosts to profile. Service banner
+ version → **Web / attack surface** (§3) and vulnerability lookup (CVE via Vulners). TLS cert
subject/SAN → back to **Domains** (§1) and *related* infra sharing the same cert (a strong attribution
pivot, `methodology.md` §5.3). Hostname in a banner → new subdomains.

**Cautions / legal note.** Shodan/Censys/FOFA/ZoomEye/Netlas/GreyNoise are **passive** - they serve
*their* scan data, so no packet leaves you toward the target; safe and stealthy. **`nmap` and any
direct port/service/OS scanning are active** and can constitute unauthorized access without a signed
ROE (`legal-ethics-opsec.md` §3, §2.1). Never run `nmap` against a third-party target from this host;
it belongs to an authorized engagement's own environment. Do the passive engines first - they usually
answer the question without ever scanning.

---

## 3. Web / attack surface

**What you are collecting.** For an in-scope web property: its technology stack, exposed and
historical endpoints, JavaScript-referenced routes and API paths, misconfigured/forgotten files, and
accidentally public secrets or storage. The goal is to enumerate the *reachable web surface* so it can
be secured (ASM) or tested under authorization.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Wappalyzer**, **BuiltWith** (builtwith.com) | Passive | Tech-stack fingerprint (framework, CMS, analytics, CDN) | Knowing WordPress/React/nginx scopes later work |
| **whatweb**, header inspection | Mixed | Server/x-powered-by, tech detection | `whatweb -a 3 target.com` (active at level 3) |
| **httpx** (ProjectDiscovery) | Active | Live-host probing: status, title, tech, TLS, ASN | Pipeline `subfinder -silent target.com | httpx -td -title -status-code` |
| **URLScan.io** (urlscan.io) | Passive-to-you (URLScan fetches) | Renders a URL, records DNS/HTTP/JS/screenshots/3rd-party calls | Analysis runs on urlscan's infra, not your IP - ideal for phishing/domain tracking |
| **Wayback / archive** - Wayback Machine (web.archive.org), `waybackurls`, `gau`, Memento | Passive | Historical URLs → old admin panels, `.env`, `.git`, backups, dead endpoints | Mine snapshots for decommissioned but still-live paths |
| **robots.txt / sitemap.xml** | Semi-passive | Declared-but-hidden paths, sitemaps of all pages | Read the target's own published hints |
| **JS / endpoint discovery** - Photon, LinkFinder-style extraction | Active (crawls) | URLs, JS files, API endpoints, emails, keys embedded in JS | `photon -u https://target.com --keys --emails` - authorized scope |
| **Google / GitHub dorking** | Passive | Indexed sensitive files, exposed code/secrets | `site:target.com filetype:pdf "confidential"`; `site:target.com intitle:"index of"`; `site:target.com inurl:login`; `site:github.com "target.com" api_key` |
| **grep.app** (grep.app) | Passive | Full-text search across public source code | `AWS_SECRET_ACCESS_KEY`, `jwt_secret`, `password=` |
| **TruffleHog**, Gitrob | Mixed | Verified secrets in repos/git/filesystem/S3 | `trufflehog github --org=<authorized-org> --only-verified` |
| **BucketLoot**, cloud-storage enum | Passive | Exposed S3/GCS/Azure bucket contents → keys/URLs | Check *your own* buckets for exposure |

**How to pivot.** Tech stack + versions → CVE/vuln lookup (Vulners, §on threat intel). Emails/authors
found in JS/comments → **Person / SOCMINT** (§4) and **Email** (§5). Analytics/AdSense IDs in page
source → *related sites* sharing the same ID (the classic private-blog-network correlation).
Subdomains from CT (§1) → this section's live-host probing. Endpoints → the application layer for an
authorized test.

**Cautions / legal note.** Tech fingerprinting via Wappalyzer/BuiltWith, dorking, `grep.app`,
Wayback, and URLScan are **passive**. **Active crawling, `httpx` probing, JS-endpoint discovery, and
secret-scanning of others' repos need authorization + scope.** Dorking is lawful (you use a public
engine's own features) - but *acting on* what it surfaces (using discovered credentials, accessing a
directory-listing you were not meant to see) can be fraudulent access (`legal-ethics-opsec.md` §2.1,
Bluetouff precedent). GitHub/`grep.app` secret hunting is legitimate **against your own org**; scanning
third-party orgs' secrets requires authorization. Route dork queries through SerpApi/Firecrawl, and
repo scans through an authorized environment - never live from the local host.

---

## 4. People / SOCMINT

**What you are collecting.** Public profiles of a *person or role* across platforms; username reuse
across sites; correlated accounts belonging to the same individual; public posts, connections,
posting patterns, time zones, and stated employer/role. SOCMINT is a subset of OSINT and the most
PII-sensitive discipline.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Sherlock** (~400 sites, fast) | Passive | Username presence across platforms | `sherlock <username> --csv` |
| **Maigret** (~2,500 sites, richer) | Passive | Username presence **+ extracted profile data** | `maigret <username> --pdf --html` |
| **WhatsMyName** (web-app + JSON site list), Blackbird, Social-Analyzer | Passive | Consistency-check a handle across platforms | Cross-verify a candidate handle |
| **LinkedIn** (logged-out dorks) | Passive | Org chart, roles, tech stack (via listed skills), personnel moves | `"@target.com" site:linkedin.com`; `site:linkedin.com/in "Target" "role"` |
| **X/Twitter advanced operators** | Passive | Targeted public posts | `from:handle`, `to:handle`, `since:2024-01-01 until:2024-06-30`, `filter:media` |
| **Instagram/Facebook/Reddit/GitHub/Telegram** | Passive | Public posts, connections, geotags, projects | Picuki/Imginn for no-account viewing; GitHub repos reveal employer projects |
| **GHunt**, Epieos | Passive | Pivot a Google account / email → linked services & reviews | Given an email/account, surface public links |

**Per-profile manual verification** (`methodology.md` §4): read bio/contact/links; reverse-image the
profile photo (§8); analyze post history, posting-time patterns (infer time zone), connections,
language and device tells. Never treat a matched username as the same person until corroborated -
handle collisions and impersonation are common.

**How to pivot.** Username → all platforms carrying it (Sherlock/Maigret) → per-profile detail →
**Email** (§5), **Phone** (§6), **Image** (§8). Employer/role → **Company** (§7). Profile photo →
reverse image (§8) → other appearances. A confirmed email → breach exposure (§5).

**Cautions / legal note.** This is the heart of the PII/GDPR problem. A public profile is still
**personal data** - accessibility grants no right of use (Clearview AI, `legal-ethics-opsec.md` §4.3).
Apply **data minimization** (collect only what the authorized purpose needs) and **proportionality**
(mapping a person's entire digital life over a single posted photo is disproportionate). Investigating
a person requires a legitimate, documented basis (consent, legitimate interest with a balancing test,
or legal obligation) and staying in scope. **Never** cross into contact, harassment, stalking,
doxxing, or impersonation to reach private content - those are hard red lines
(`legal-ethics-opsec.md` §2.2, §6.3). Viewing public profiles logged-out is passive; a research
("sock-puppet") account is ethics- and ToS-gated (`legal-ethics-opsec.md` §6.3). Special-category data
(health, politics, religion, sexuality, art. 9 GDPR) is off-limits absent an explicit lawful basis.

---

## 5. Email & credentials / breach awareness

**What you are collecting.** For a domain: the corporate email-address *pattern* and known addresses.
For a specific (own or authorized) address: which services it is registered on, and whether it appears
in known breaches. The credential angle is strictly **exposure awareness** - confirming that accounts
are exposed so they can be reset - never obtaining or using others' credentials.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Hunter.io** (hunter.io) | Passive | Domain email pattern (`first.last@`, `f.last@`) + known addresses | Reveals exec/support/security contacts and the format |
| **theHarvester** | Passive | Emails, subdomains, hosts from public sources | `theharvester -d target.com -b google,bing,linkedin,crtsh` |
| **holehe** | Passive/semi-active | Which of 100+ services an email is registered on (no alert to owner) | `holehe target@example.com --only-used` |
| **Epieos** | Passive | Reverse-email → linked socials, Google account/Maps reviews | Pivot an address to a person |
| **Have I Been Pwned** (haveibeenpwned.com) | Passive | Whether an email/domain is in known breaches/paste dumps | Domain-wide monitoring (API/subscription) of employee exposure |
| **Email-header analysis** - MXToolbox, Gaijin.at, MailHeader.org | Passive | Parse `Received:` chain → originating IP, SPF/DKIM/DMARC alignment, spoofing indicators | Read bottom-most `Received:` for source; check Return-Path mismatch |

**Breach-awareness services** (DeHashed, IntelX, Leak-Lookup, LeakCheck, h8mail, commercial Flare.io /
SpyCloud) index leaked-credential and stealer-log corpora. **Awareness vs. use is a bright line**
(`legal-ethics-opsec.md` §5): checking whether *your own org* or an *authorized in-scope subject*
appears in a breach is legitimate and often a compliance duty; **downloading, aggregating, querying
for other people's credentials/PII, or using any leaked credential to access anything is not** - a
leaked dump is *not an open source*, and using it can be handling stolen goods (recel) and fraudulent
access. Stealer-log awareness (that malware exfiltrates and resells credentials) is context for
**defensive** monitoring and forced resets, not a route to obtain working credentials.

**How to pivot.** Email pattern (Hunter.io) + employee names (LinkedIn, §4) → generate likely
addresses for authorized phishing-simulation planning. Email → registered-services map (holehe) →
**Person / SOCMINT** (§4). Email → breach exposure (HIBP) → force a reset (defensive outcome). Header
originating IP → **IP / infrastructure** (§2). Verified address → domain → **Domains** (§1).

**Cautions / legal note.** Hunter.io/theHarvester/HIBP/holehe are passive. Email addresses are PII -
GDPR minimization and purpose-limitation apply (`legal-ethics-opsec.md` §4). Use breach lookups
**only** for own or authorized-in-scope subjects. Do not obtain, store, redistribute, or act on other
people's leaked credentials (`legal-ethics-opsec.md` §5, §2.3). Run these through managed APIs, not the
local host.

---

## 6. Phone

**What you are collecting.** For a phone number: validity and formatting (E.164), country/region,
carrier and line type (mobile/landline/**VoIP**), and any public footprint (search-engine mentions,
linked accounts).

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **PhoneInfoga** | Passive | Format validation, carrier, region, VoIP detection, search-engine footprint | `phoneinfoga scan -n "+33123456789"` |
| **`phonenumbers`** (Python lib) | Passive (offline) | Carrier name, geocoding, line type from the numbering plan | `carrier.name_for_number(...)`, `geocoder.description_for_number(...)` |
| **Epieos phone lookup**, Truecaller / reverse-lookup directories | Passive | Caller-ID name, linked accounts | Cross-reference cautiously (crowd-sourced, error-prone) |

**How to pivot.** Number → linked accounts/registrations → **Person / SOCMINT** (§4) and **Email**
(§5). VoIP flag (Google Voice, TextNow, Twilio, burner apps) → indicates disposable/anonymized use,
common in fraud. Carrier/region → geographic context. Reverse-lookup name → identity corroboration
(never treat a single crowd-sourced name as confirmed).

**Cautions / legal note.** A phone number is **personal data** - minimization, purpose-limitation, and
proportionality apply (`legal-ethics-opsec.md` §4). Reverse-lookup and caller-ID databases are
frequently inaccurate and can misattribute a number; **corroborate before asserting** (`methodology.md`
§3-4) and never use a number to contact, harass, or de-anonymize a subject outside an authorized,
lawful purpose (`legal-ethics-opsec.md` §2.2). Passive tooling only.

---

## 7. Company / corporate / financial (FININT)

**What you are collecting.** Legal entity data: registration, officers/directors, ownership and
beneficial ownership, filings and financials, group/subsidiary links, IP (patents/trademarks),
tenders/contracts, and adverse-media / sanctions / PEP flags. This is the backbone of KYC, due
diligence, and corporate investigation.

**Global sources & tools.**

| Source | What it gives |
|--------|---------------|
| **OpenCorporates** (API) | Largest cross-jurisdiction company DB - entities, officers, links |
| **SEC EDGAR** (US) | 10-K / 10-Q / DEF 14A proxy → exec names, comp, ownership, risk factors |
| **Crunchbase** | Funding rounds, M&A, investors, headcount signals |
| **OCCRP Aleph**, **Offshore Leaks Database** (ICIJ, 785k+ entities) | Cross-border/offshore structures, leaks + official docs |
| **Companies House** (UK), state Secretary-of-State registries (US), other national registers | Filings, officers, beneficial owners |
| **Patents/trademarks** - Espacenet (EPO), WIPO, national IP offices | R&D axes, sometimes pre-announcement |
| **Sanctions/PEP** - OFAC SDN, EU consolidated list, UN, UK OFSI; adverse-media search | Sanctions, watchlist, politically-exposed-person and reputational flags |

**France-specific sources.**

| Source | What it gives |
|--------|---------------|
| **Pappers** (pappers.fr) | Free company data: officers, financials, filings, beneficial owners, capital links, group structure |
| **Infogreffe** | Official registry filings - statutes, minutes, appointments, certified extracts (Kbis) |
| **BODACC** (Bulletin Officiel des Annonces Civiles et Commerciales) | Creations, modifications, transfers, insolvencies/procedures collectives |
| **INPI** (inpi.fr) | National patents & trademarks (R&D signals) + registre national des entreprises |
| **data.gouv.fr** | Public-procurement awards (chosen providers/tech/amounts), subsidies, inspections, cadastre |
| **Legifrance** | Case law + CNIL decisions (past litigation, GDPR sanctions) |
| **Societe.com / Societe.ninja** | Turnover, results, directors, secondary establishments, capital links |

**How to pivot.** Officers/directors → **Person / SOCMINT** (§4). Registered domains / website →
**Domains** (§1). Beneficial owners & capital links → parent/subsidiary graph → adverse-media &
sanctions screening. Filed addresses/phones/emails → §5-6. Group structure → each entity re-run
through this section. Build the corporate graph in Maltego (`methodology.md` §5.1).

**Cautions / legal note.** Corporate registry data is designed to be public and is the lowest-risk
discipline - but officers'/beneficial-owners' names are still personal data (GDPR minimization /
purpose-limitation, `legal-ethics-opsec.md` §4). Frame the work as **KYC / due diligence / compliance**
with a documented purpose. Sanctions/PEP screening must use authoritative, current lists and be
corroborated - a name match is a *lead*, not a determination (grade with the Admiralty code,
`methodology.md` §3). Keep adverse-media findings proportionate and sourced; do not conflate an
allegation with a fact. Leaks-database material (OCCRP Aleph, Offshore Leaks) is legitimate for
journalism/investigation but treat downstream use of any personal data lawfully.

---

## 8. Images / GEOINT

**What you are collecting.** From an image: where else it appears online (reuse, origin, mislabeling);
embedded metadata (camera, software, timestamps, and sometimes GPS); and - from visual content alone -
the *location* (geolocation) and *time* (chronolocation) it was captured.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Reverse image** - Google Images, **Yandex** (strongest, esp. faces), TinEye, Bing | Passive | Where/when an image appeared; origin; identify place/object | Detects recycled/older/mislabeled media |
| **ExifTool** (exiftool.org) | Passive (on a downloaded file) | EXIF/metadata: camera, software, author, timestamps, GPS | `exiftool image.jpg | grep -i gps` |
| **Jeffrey's Exif Viewer**, Metadata2Go, FotoForensics | Passive | Web-based EXIF read + error-level image forensics | Detect manipulation/splicing |
| **InVID / WeVerify** | Passive | Video keyframe extraction + verification | Break a video into frames for reverse search |
| **Geolocation** - Google Earth Pro (historical imagery), Street View, Yandex/Bing/Baidu Maps, Overpass Turbo (OSM), PeakFinder | Passive | Match landmarks, signage, terrain, mountain skylines | Confirm a candidate location in 3D + Street View |
| **Chronolocation** - SunCalc.org, shadow/sun-position analysis | Passive | Time-of-day / date from shadow direction & length | Combine sun azimuth + known coordinates |

**Geolocation methodology** (`methodology.md` §5): read **global indicators** (architecture, road
markings, vegetation, driving side, sun position) then **local indicators** (language on signage,
license-plate format, storefront names, phone-number prefixes); form a location hypothesis; confirm on
satellite/Street View by matching fixed landmarks. Chronolocation cross-checks shadow geometry against
the sun's position for the candidate coordinates and date.

**How to pivot.** Reverse image → other pages/profiles carrying it → **Person / SOCMINT** (§4) or a
source website → **Domains** (§1). EXIF GPS → coordinates → map/Street View context. EXIF author/
software/username → §4. A confirmed location + timestamp → build/verify a **timeline**
(`methodology.md` §5.2).

**Cautions / legal note.** Note that major platforms (X, Facebook, Instagram) **strip EXIF on upload**,
so GPS usually survives only on files shared by email/forum/direct download - do not assume its
absence means anything. **Facial-recognition** reverse search (e.g. PimEyes) is heavily constrained:
untargeted facial-image scraping to build recognition databases is unlawful (Clearview AI; EU AI Act
bans; `legal-ethics-opsec.md` §4.3) - use face search only where a lawful basis exists and never to
de-anonymize or track a private individual. Images of identifiable people are personal data and engage
the right to one's image; geolocating a private person's home/movements can cross into stalking - a
hard red line (`legal-ethics-opsec.md` §2.2). Run reverse-image and map lookups through managed tools.

---

## 9. Documents & metadata

**What you are collecting.** Public documents belonging to a target (PDF, DOC/DOCX, XLS, PPT) and the
**metadata** embedded in them: authors and usernames, software and versions, creation/modification
dates, and internal file-system or network paths that leak org structure.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Document discovery** - Google dorks, **Metagoofil**, **FOCA** (Windows GUI) | Passive/Mixed | Harvests public docs from a domain, then extracts their metadata | `metagoofil -d target.com -t pdf,docx,xlsx`; dork `site:target.com filetype:xlsx` |
| **ExifTool**, Apache **Tika**, MediaInfo, Metashield | Passive | Metadata from documents/media (authors, software, paths) | `exiftool report.pdf` → author, producer, creation date |
| **Octopii** (redhuntlabs) | Passive | OCR+regex+NLP PII scanner over public-facing docs/images | Scan *your own* exposed storage for leaked PII |

**How to pivot.** Author/username in metadata → **Person / SOCMINT** (§4), often the corporate
username format. Software versions → tech-stack + vulnerability context (§3). Internal paths / server
names → network/infra hints (§2) and org structure. Email addresses in documents → §5.

**Cautions / legal note.** Harvesting and reading metadata from *published* documents is passive and
lawful, and Octopii-style scanning of **your own** exposed storage is a defensive best practice
(strip metadata before publishing). Authors' names and internal identifiers are personal data (GDPR,
`legal-ethics-opsec.md` §4). **Leaked / stolen documents are not an open source** - awareness that a
leak exists is fine, but obtaining, storing, or acting on stolen document contents to harm is
handling stolen goods (`legal-ethics-opsec.md` §5, §2.3). Fetch documents through managed tools, not
the local host.

---

## 10. Archives & historical

**What you are collecting.** Prior states of a web resource: removed or edited pages, old endpoints,
past ownership and design, deleted posts, and content that has since been de-referenced. The archival
layer both *recovers* vanished information and *preserves* your own findings as third-party-witnessed
evidence.

**Key sources & tools.**

| Tool | Passive/Active | What it gives | Example |
|------|----------------|---------------|---------|
| **Wayback Machine** (web.archive.org), Memento | Passive | Historical snapshots of a URL over time | Compare snapshots to spot removed pages / silent edits |
| **Archive.today** (archive.ph), Perma.cc | Passive | On-demand snapshot + permanent citation | Freeze a page that may change/vanish |
| **`waybackurls`, `gau`** | Passive | All historical URLs known for a host | Feed into §3 endpoint discovery (old `.env`, admin paths) |
| **Cached pages** (search-engine cache) | Passive | Recently-removed content still in cache | Recover a just-deleted page |
| **WaybackPy** (`--save`) | Passive | Programmatically submit a page to Wayback (third-party witness) | Preserve evidence with an independent timestamp |

**How to pivot.** Old page content → emails, names, endpoints, past domains → §1, §3, §4, §5.
Historical WHOIS/DNS (§1) + Wayback history together reconstruct past ownership and infra. Removed
content → the reason it was removed (a lead in itself).

**Cautions / legal note.** Archives are passive and are a core verification and evidence-preservation
tool (`methodology.md` §4, §7.2). **But the right to erasure (GDPR art. 17) matters**: storing
personal data that has been de-referenced or erased at the source, purely because a cache/archive
still holds it, can itself violate art. 17 (`legal-ethics-opsec.md` §4). Archive for legitimate,
proportionate evidentiary purposes; do not build a private store of erased personal data. Timestamp
and hash preserved evidence (`methodology.md` §7.2). Fetch archives via managed tools.

---

## 11. Dark web & paste sites (awareness level only)

**What you are collecting - and the hard boundary.** This discipline is included at **awareness /
defensive-monitoring level only**: detecting whether *your own organization's* data, credentials,
domains, or brand are mentioned on paste sites, breach-leak sites, or dark-web forums, so you can
respond. It is **not** a route to marketplaces, transactions, or acquiring others' stolen data - this
skill provides **no operational guidance for accessing, transacting on, or navigating criminal
marketplaces**.

**Key sources (monitoring your own exposure).**

| Source | What it gives |
|--------|---------------|
| **Paste-site monitoring** - Pastebin/Gist and successors, Telegram-channel monitoring | Early sight of leaked snippets / credentials naming your org |
| **Intelligence X** (intelx.io) | Searchable index spanning leaks, pastes, historical data, some dark-web references |
| **Managed dark-web monitoring** - Flare.io, Recorded Future, Flashpoint, Intel 471, SpyCloud, SpiderFoot HX | Continuous, *managed* monitoring of your own org's exposure (credentials, mentions, leaked data) |
| **CT / typosquat monitoring** - dnstwist (`--registered --mxcheck`), CertStream | Look-alike domains and phishing infra targeting your brand |

**How to pivot (defensive).** A leaked credential mention → force resets (§5). A mention of your org
or a partner → incident response / NIS2 notification consideration. A typosquat domain → abuse
reporting to the registrar (WHOIS abuse contact, §1). A leak-site listing → threat-intel enrichment
(actor, TTPs) *without* acquiring the underlying stolen data.

**Cautions / legal note.** Strongest caution in this file. **Prefer managed monitoring services** -
they surface exposure without you touching illicit infrastructure, and they keep the local IP clear
(no-local rule, `legal-ethics-opsec.md` §6.1). A leaked dump / stolen dataset is **not an open source**
(`legal-ethics-opsec.md` §0, §5): monitoring for mentions of your own exposure is legitimate;
**purchasing, downloading, aggregating, or using stolen data - or transacting on any marketplace - is
out of scope and unlawful** (handling stolen goods; fraudulent access; possibly financing crime,
`legal-ethics-opsec.md` §2.3, §5). If an investigation surfaces active criminal activity, the correct
move is **abuse reporting / law-enforcement referral**, not engagement (`methodology.md` reporting +
`legal-ethics-opsec.md`). Sock-puppet use in these spaces is doubly gated by ToS and law
(`legal-ethics-opsec.md` §6.3).

---

## 12. Cross-discipline pivot map (quick reference)

The disciplines form one graph. Common productive pivots:

- **Domain** → resolved **IP** → **ASN** → org's full range → more hosts
- **Domain** → **CT/subdomains** → **Web** live-host probing → tech stack → **CVE**
- **Domain WHOIS** → registrant **Company** → **officers/people** → **email** → **breach exposure**
- **Person/username** → all-platform presence → profile detail → **email** / **phone** / **image**
- **Email** → registered services (holehe) + pattern (Hunter.io) → **people**; header → originating **IP**
- **Image** → reverse search → source page/profile → **person/domain**; **EXIF GPS** → location + timeline
- **Company** → subsidiaries/beneficial owners → sanctions/PEP + adverse media; filings → domains/contacts
- **Any finding** → **archive it** (Wayback/Archive.today) for verification + evidence

At every hop: grade the finding (Admiralty code), corroborate with independent sources, record
provenance (URL + UTC + tool + hash), stay on the lowest intrusiveness rung that answers the
requirement, confirm the pivot is still in authorized scope, and route the retrieval through a managed
tool - never the local IP. See `methodology.md` §2-5, §7 and `legal-ethics-opsec.md` throughout.
