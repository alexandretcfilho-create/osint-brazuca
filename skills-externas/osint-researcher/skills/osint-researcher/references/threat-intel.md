# Cyber Threat Intelligence (CTI) - Reference

> Loaded on demand. This file covers the **Cyber Threat Intelligence application of OSINT**:
> the CTI types and their audiences, the CTI cycle and how OSINT feeds it, indicators of
> compromise and the Pyramid of Pain, the adversary frameworks (MITRE ATT&CK, the Diamond
> Model, the Cyber Kill Chain), threat-actor profiling and infrastructure pivoting, the OSINT
> CTI source ecosystem, how to produce and disseminate a CTI product, defensive attack-surface /
> brand / exec monitoring, and the regulatory-driven testing context (TIBER-EU / DORA TLPT).
>
> **Scope and framing.** Everything here is **defensive and authorized**: understanding
> adversaries in order to detect, prevent, and respond; monitoring *your own* (or a consenting
> client's) exposure; sharing intelligence responsibly. CTI extracts *adversary tradecraft and
> defensive awareness* - never offensive attack operations. Where a technique could be dual-use
> (infrastructure pivoting, credential-leak monitoring), it is described for **monitoring,
> attribution, detection, and takedown**, not for conducting attacks.
>
> This file assumes the analytic backbone in `methodology.md` (the intelligence cycle, the
> intrusiveness ladder, the Admiralty grading code, verification, confidence/estimative language,
> reporting) and the boundaries in `legal-ethics-opsec.md` (authorization, red lines, PII/GDPR,
> TLP handling, evidence). It does not repeat them; it specializes them for CTI. For the
> discipline-organized tool catalogue, see `tools-catalog.md`; for routing collection through the
> connected search/scraping tools, see `mcp-tooling.md`.
>
> One rule carried from the methodology sits above everything here: **do not over-attribute.**
> Infrastructure overlaps and TTP similarities are *leads*, not proof; naming a threat actor is
> a high-consequence analytic judgment that demands corroboration and estimative caveats.

---

## 1. What CTI is, and its four types

**Cyber Threat Intelligence** is the product of running threat data through the intelligence
cycle so that a defender can make a decision: what to detect, what to patch, what to block, where
to invest. It is **not** a feed of raw indicators, and it is not a vendor dashboard. As with all
OSINT, the OSINF→OSINT gate applies (see `methodology.md` §1.6): a list of IPs is data; an
*assessed, contextualized, actionable* judgment about a threat is intelligence.

CTI is conventionally split into four types by **altitude and audience**. The same underlying
threat produces all four; they differ in abstraction, shelf-life, and consumer.

| Type | Question it answers | Time horizon | Typical content | Primary audience |
|------|---------------------|--------------|-----------------|------------------|
| **Strategic** | *Who might target us and why? Where should we invest?* | Long (months-years) | Threat landscape, adversary intent & capability, sector/geopolitical trends, risk to the business, motivations | Executives, board, CISO, risk & policy owners |
| **Operational** | *What campaigns/actors are active against organizations like us, and how do they operate?* | Medium (weeks-months) | Active groups & campaigns, their TTPs, targeting, timing, likely impact | Threat-intel team, IR leadership, security architects |
| **Tactical** | *What TTPs must we detect and defend against right now?* | Short-medium | Adversary tactics, techniques and procedures, tooling behavior, detection logic (Sigma/YARA) | SOC analysts, detection engineers, hunters |
| **Technical** | *What specific artifacts indicate this threat, so I can block/alert?* | Very short (indicators age fast) | Atomic IOCs - hashes, IPs, domains, URLs, C2 - malware detail | SOC tooling / SIEM / firewall / EDR (largely automatable) |

Notes on the split:
- **Strategic and operational** are analyst-written prose products; **tactical and technical**
  are increasingly machine-consumable (rules and feeds).
- **Tactical vs technical** are often conflated. The clean distinction: *technical* = the atomic
  artifacts (an MD5, an IP); *tactical* = the behavior and how the adversary uses them (spear-phish
  → macro → scheduled task). Tactical intelligence outlives technical intelligence because
  behavior changes far more slowly than infrastructure (see the Pyramid of Pain, §3).

### 1.1 Consumers and what each needs

- **SOC / detection engineers** - need tactical + technical: detection rules, hunt hypotheses,
  fresh IOCs with context and expiry, ATT&CK technique coverage gaps.
- **Incident responders** - need operational + technical during an incident: is this a known
  actor? what do they do next (persistence, lateral movement, exfil)? what related infrastructure
  should we sweep for?
- **Security architecture / vulnerability management** - need operational + strategic: which
  exposures are being exploited in the wild, prioritized by adversary interest, not just CVSS.
- **Leadership / board** - need strategic: are we a plausible target, by whom, at what business
  risk, and is spend proportionate. Estimative language and analytic confidence (see
  `methodology.md` §6) matter most here - never present a strategic assessment as certainty.

> Golden rule of dissemination: **match the product to the consumer's decision.** A board deck of
> MD5 hashes is useless; a SOC ticket that says "APT-ish actor, be careful" is useless. Altitude
> is not optional polish - it is the point.

---

## 2. The CTI cycle and how OSINT feeds it

CTI runs the same intelligence cycle described in `methodology.md` §1 - do not re-learn a separate
one. The CTI-specific framing of the stages:

| Stage (methodology.md) | CTI expression |
|------------------------|----------------|
| **1. Planning & Direction** | Set **Intelligence Requirements (IRs)** / PIRs driven by the org's risk: which threats, sectors, assets, and adversaries matter. This is where you decide *what to hunt*. |
| **2. Collection** | Gather from the OSINT CTI source ecosystem (§5) plus internal telemetry. Passive-first (see `methodology.md` §2). |
| **3. Processing / Collation** | Parse, deduplicate, normalize, and **enrich** IOCs; translate; de-duplicate against what's already known. Aim for a low false-positive rate - junk indicators poison detections. |
| **4. Analysis & Production** | Map observations to frameworks (§4), profile actors (§4b), grade and score indicators, assign confidence, answer "so what." |
| **5. Dissemination & Feedback** | Push the right product to the right consumer at the right TLP marking; feed detections; capture whether the intelligence was actionable and re-task. |

### 2.1 Requirements first (the CTI failure mode is skipping this)

The most common CTI failure is collecting indicators nobody asked for. Anchor collection to
**Intelligence Requirements** tied to the organization's actual risk profile: its sector, its
crown-jewel assets, its threat model, the adversaries that historically target its peers. A retail
firm and a defense contractor need different intelligence; a generic firehose serves neither. State
the IR before you collect ("Which ransomware crews are actively targeting our sector, and what are
their initial-access TTPs?"), then let it drive source selection.

### 2.2 Where OSINT sits

OSINT is the dominant **collection** source for CTI, and one of several:
- **OSINT** - vendor threat reports, CERT/CSIRT advisories, CVE/NVD, malware repositories,
  cyber-security blogs, professional forums, threat-sharing communities (§5).
- **SOCMINT** - social-media monitoring for actor claims, leak announcements, targeting chatter.
- **Deep/dark-web monitoring** - criminal forums, marketplaces, and ransomware leak sites, for
  compromised-credential sales, resold access (VPN/RDP), and mentions of your org or partners. This
  can detect a breach *before* internal telemetry does. (Monitoring only - see `legal-ethics-opsec.md`
  §5 on breach data and §2 on trafficking; the goal is awareness and takedown, never purchase or use.)
- **Internal telemetry** - SIEM/EDR/firewall/proxy logs. Not OSINT, but the essential *fusion*
  partner: cross-referencing an external signal against internal logs is what turns a generic
  indicator into an actionable alert ("this C2 domain from the feed was actually resolved by an
  internal host").

The value multiplier is **fusion**: OSINT tells you what adversaries do generally; internal
telemetry tells you whether it is happening to you. CTI is the join.

---

## 3. IOCs and the Pyramid of Pain

### 3.1 Indicator types

An **Indicator of Compromise (IOC)** is an observable artifact associated with malicious activity.
The common atomic types:

| IOC type | Examples | Notes |
|----------|----------|-------|
| **File hashes** | MD5, SHA-1, SHA-256; fuzzy hashes (ssdeep, imphash) | Trivially changed by the adversary (one byte flips the hash); fuzzy/import hashes are more durable |
| **IP addresses** | C2, scanning, phishing-host IPs | Easily rotated; often shared/CDN-fronted → false-positive risk |
| **Domain names** | C2, phishing, malware-distribution domains | Cheap to register; DGA-generated domains defeat static blocklists |
| **URLs** | Payload URLs, phishing kit paths, webshell paths | More specific than a domain; still cheap to move |
| **Email artifacts** | Sender addresses, subjects, attachment names, header patterns | Useful for phishing detection; spoofable |
| **Network/host artifacts** | User-agent strings, mutexes, registry keys, named pipes, JA3/JA4 TLS fingerprints, certificate hashes | More durable than IPs/domains; tied to tooling behavior |
| **TTPs** | Tactics, techniques, procedures (mapped to ATT&CK, §4) | The most durable and most valuable - behavior, not artifacts |

### 3.2 The Pyramid of Pain (David Bianco)

The Pyramid of Pain ranks indicator types by **how much pain it causes the adversary when you deny
them that indicator**. Detecting low-pyramid indicators is cheap for you and cheap for the adversary
to evade; detecting high-pyramid indicators is harder for you but *forces the adversary to change how
they operate.*

```
                    /\        TTPs                - "Tough!"  (change = re-tooling their whole operation)
                   /  \       Tools               - "Challenging"
                  /----\      Network/Host arts.   - "Annoying"
                 /      \     Domain names         - "Simple" (a bit of effort to rotate)
                /--------\    IP addresses         - "Easy"   (rotate at will)
               /          \   Hash values          - "Trivial" (one byte → new hash)
              /____________\
```

- **Bottom (hashes, IPs):** trivial/easy for the adversary to change. High false-positive and
  churn cost for you. Still worth blocking (cheap, catches the lazy and the automated), but do not
  build your program on them - they expire fast (technical intelligence, §1).
- **Middle (domains, network/host artifacts, tools):** progressively more painful to change.
- **Top (TTPs):** the adversary's *behavior* - how they gain access, escalate, persist, move,
  exfiltrate. Changing this means retraining, re-tooling, redesigning the operation. **This is why
  TTPs > atomic indicators:** a detection built on a technique (e.g. "scheduled task creating a
  process from a user-writable path") survives infrastructure rotation, catches unseen campaigns,
  and imposes real cost on the adversary. Atomic IOCs catch yesterday's campaign; TTP detections
  catch the actor.

> Practical stance: **collect and block the bottom for hygiene; invest analysis and detection
> engineering at the top.** Every atomic IOC you receive should, where possible, be lifted to the
> TTP it exemplifies and mapped to ATT&CK (§4).

### 3.3 IOC lifecycle, scoring, and formats

IOCs are perishable and noisy. Manage them:

`Collection → Validation → Enrichment → Scoring → Deployment → Expiration`

- **Validation** - is it real, is it malicious, is it still live. Filter false positives *before*
  deployment; a bad indicator (e.g. a shared CDN IP) generates alert fatigue and erodes trust.
- **Enrichment** - resolve context: passive DNS, WHOIS, geolocation, related samples, ATT&CK
  mapping, first/last-seen.
- **Scoring** - score by source reliability, recency, context (targeted vs generic noise),
  confidence (multi-source corroboration), and fidelity (false-positive potential). This is the
  Admiralty code (`methodology.md` §3) applied to indicators; aim for a low false-positive rate.
- **Expiration** - indicators must have a shelf-life and be retired; a permanent blocklist of stale
  IPs eventually blocks legitimate reassigned addresses.

**Structured formats & transport (awareness):**
- **STIX 2.x** - the standard structured representation of threat intelligence (indicator objects
  with `pattern`, `pattern_type`, `valid_from`, `labels: ["malicious-activity"]`; plus objects for
  threat actors, campaigns, malware, attack patterns, relationships).
- **TAXII** - the transport protocol for exchanging STIX feeds (client/server, collections, poll/push).
- **MISP** format - the de-facto community format for event/attribute sharing, natively TLP-aware.

---

## 4. Adversary frameworks

Three complementary models. They are not competitors; a mature analyst uses all three - ATT&CK to
describe *what* an adversary did technique-by-technique, the Kill Chain to describe *the sequence*,
and the Diamond Model to describe *the event's four facets* and pivot between them.

### 4.1 MITRE ATT&CK (the central TTP taxonomy)

ATT&CK is a curated, community knowledge base of adversary behavior, structured as:

- **Tactics** - the adversary's *goal* at a stage (the "why"), e.g. `TA0043 Reconnaissance`,
  `TA0001 Initial Access`, `TA0002 Execution`, `TA0003 Persistence`, `TA0005 Defense Evasion`,
  `TA0008 Lateral Movement`, `TA0011 Command and Control`, `TA0010 Exfiltration`, `TA0040 Impact`.
- **Techniques / sub-techniques** - *how* the goal is achieved (the "what"), e.g. `T1566.001`
  Spearphishing Attachment, `T1059.001` PowerShell, `T1053.005` Scheduled Task.
- **Procedures** - the specific in-the-wild implementation by a given actor/tool.

**How to use it in analysis:**
- **Map every observation** to its technique ID. This is the common language across analysts, SOCs,
  IR, and red teams - an ATT&CK ID is unambiguous where prose is not.
- **Build an actor's technique profile** (the set of techniques a group is known to use) and compare
  it to yours: an ATT&CK Navigator layer highlights *coverage gaps* (techniques you cannot detect).
- **Prioritize detection engineering** around the techniques your relevant adversaries actually use,
  not the whole matrix.
- The **Reconnaissance tactic (TA0043)** is where OSINT-relevant adversary behavior lives (active
  scanning, gathering victim identity/host/org info, searching open sources) - the mirror image of
  your own defensive attack-surface monitoring (§7).

### 4.2 The Diamond Model of Intrusion Analysis

*(From general CTI doctrine - not detailed in the ATT&CK-centric source material; stated here as
established general knowledge.)*

Every intrusion event has **four vertices**, each connected to the others:

```
                Adversary
                    /\
                   /  \
        Capability /    \ Infrastructure
                   \    /
                    \  /
                    \/
                  Victim
```

- **Adversary** - the actor/operator (who).
- **Capability** - the tools/techniques/malware used (how).
- **Infrastructure** - the C2, domains, IPs, email, delivery infra (what they used to reach the victim).
- **Victim** - the target org/person/asset (against whom).

Plus **meta-features**: timestamp, phase, result, direction, methodology, resources.

**How to use it in analysis:** the model's power is **pivoting** - knowing one vertex lets you
discover the others. From a **capability** (a malware sample) you pivot to the **infrastructure** it
beacons to; from that infrastructure you pivot (passive DNS, shared certs) to *other* infrastructure
and thence to a broader **adversary** cluster; from the **victim** set you infer adversary
**intent**. Each edge is a hypothesis to corroborate. This is the conceptual engine behind §4b's
infrastructure pivoting.

### 4.3 The Cyber Kill Chain (Lockheed Martin)

*(From general CTI doctrine - stated here as established general knowledge.)*

A **linear, sequential** model of an intrusion's phases:

`Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control (C2) → Actions on Objectives`

**How to use it in analysis:**
- **Locate where an observation falls** in the sequence, and reason about what came before and what
  comes next ("we caught C2 traffic - assume Delivery/Exploitation/Installation already succeeded;
  hunt backward").
- **Defensive framing:** the earlier in the chain you break it, the cheaper the defense - the
  original doctrine is that a defender need only disrupt *one* link to defeat the intrusion.
- The Kill Chain is coarse (7 linear phases) where ATT&CK is granular (hundreds of techniques) and
  non-linear. Use the Kill Chain for the **narrative/sequence**, ATT&CK for the **technique detail**;
  they nest cleanly (each Kill Chain phase maps to one or more ATT&CK tactics).

> Which to reach for: **Kill Chain** = the story arc of an intrusion. **ATT&CK** = the technique-level
> catalogue for detection and coverage. **Diamond** = the pivoting engine for attribution and
> infrastructure discovery. Report using ATT&CK IDs; investigate using the Diamond; narrate using the
> Kill Chain.

---

## 4b. Threat-actor profiling & infrastructure pivoting

### 4b.1 The threat-actor profile

A reusable template for documenting an adversary. Every field carries a grade and confidence
(`methodology.md` §3, §6); attribution fields especially must be caveated.

```
THREAT-ACTOR PROFILE - <name / designation>
- Aliases / cross-vendor names   (see naming caveat §4b.4)
- Suspected origin / sponsorship (grade + confidence; often the weakest, most-caveated field)
- Motivation      : financial | espionage | hacktivism | disruption/sabotage
- Target sectors & geographies
- TTPs            : mapped to MITRE ATT&CK technique IDs (the core of the profile)
- Known tooling   : malware families / commodity tools + their detection surface
- Infrastructure  : C2 patterns, registrar/hosting patterns, certificate/JA3-JA4 fingerprints,
                    naming conventions, reused templates/images
- Campaign history: timeline of observed campaigns (UTC)
- Detection       : Sigma / YARA / Snort rules that fire on this actor's behavior
- Attribution basis + confidence (what evidence, how corroborated, alternatives considered - ACH)
- References      : primary vendor reports, CERT advisories, sample hashes
```

### 4b.2 Infrastructure pivoting (defensive monitoring)

Adversaries reuse and cluster infrastructure; correlating those artifacts lets a defender **discover
related malicious infrastructure to block/monitor** and **cluster activity to an actor**. Techniques
(all passive - see `methodology.md` §2):

- **Passive DNS** - historical domain↔IP resolutions reveal co-hosted domains, hosting moves, and
  infrastructure reused across campaigns. (SecurityTrails, DNSDB-class services, VirusTotal relations.)
- **Certificate pivoting** - Certificate Transparency logs (`crt.sh`, censys) surface subdomains and
  *shared certificates*; a reused/self-signed cert or a distinctive Subject/serial ties otherwise
  unrelated hosts together. JARM/JA4S TLS-server fingerprints cluster servers running the same
  malware C2 or phishing stack.
- **WHOIS / registration patterns** - a reused registrant email, registrar, name-server set,
  registration cadence, or privacy-service choice links domains. Historical WHOIS (before redaction)
  is often decisive.
- **Shared-hosting / ASN analysis** - bulletproof-hosting ranges, ASN reuse, and co-located victims.
- **Content/behavioral fingerprints** - reused phishing-kit templates, favicon hashes, HTML/JS
  artifacts, C2 response signatures, mutex/named-pipe patterns.
- **Malware-relationship pivoting** - from a sample, pivot to the domains/IPs it contacts, to other
  samples signed with the same cert or built with the same toolchain (imphash), to the campaign.

Chain example (defensive): `phishing URL → domain → passive DNS → co-hosted domains → shared TLS cert
→ 12 more lookalike domains → registrant pattern → cluster → monitor/block all, report to registrar
abuse (see legal-ethics-opsec.md §7 and takedown below).`

### 4b.3 Tracking C2 / phishing infrastructure

For **defensive monitoring** - early warning and blocking, not interference: watch for newly
registered lookalike/typosquat domains (§7), monitor Certificate Transparency in near-real-time for
certs issued on your brand or on known-bad patterns, ingest community C2 trackers (abuse.ch, §5), and
pre-emptively block or alert on the clustered infrastructure. Feed confirmed hostile infrastructure to
DNS-level blocking and SIEM detections; pursue takedown through **registrar/hosting abuse contacts**
and CDN abuse forms (never through unauthorized access or interference - see `legal-ethics-opsec.md`).

### 4b.4 The over-attribution caveat (read before naming anyone)

Attribution is the highest-consequence, lowest-certainty judgment in CTI. Discipline:

- **Infrastructure/TTP overlap is a lead, not proof.** Shared hosting, a common phishing kit, a
  reused tool, or an ATT&CK-technique match can arise from commodity tooling, false flags, shared
  contractors, copycats, or coincidence. Adversaries **deliberately plant false flags.**
- **Distinguish clustering from attribution.** You can confidently say "these 40 domains belong to one
  activity cluster" (defensible, technical) long before you can say "this cluster is Group X"
  (interpretive, political, high-stakes). Report the cluster; caveat the name.
- **Vendor names ≠ one actor.** Different vendors track overlapping-but-not-identical activity sets
  under different names (e.g. one campaign may appear under several vendor designations); these are
  *not* interchangeable and mapping them is itself an analytic claim.
- **Use estimative language and ACH** (`methodology.md` §5.4, §6): enumerate alternative
  attributions, seek disconfirming evidence, and state confidence. "We assess *with moderate
  confidence* that…" - never "this was definitely Group X" off infrastructure overlap alone.
- **Attribution is rarely the defender's actual requirement.** Detection and mitigation usually do
  not need a name; do not chase attribution the mission does not require.

---

## 5. OSINT CTI source ecosystem

Prioritize free/open, authoritative sources; corroborate across them (`methodology.md` §3). All are
consumed passively.

### 5.1 Vulnerability & advisory intelligence
- **CVE / NVD** - the canonical vulnerability identifiers and scoring (CVSS); NVD adds CPE and
  metadata. Pair with **CISA KEV** (Known Exploited Vulnerabilities) to prioritize *what is actually
  exploited in the wild* over raw CVSS.
- **Vendor & CERT/CSIRT advisories** - first-party vendor security bulletins, national CERT
  advisories, sector ISAC reports. Primary sources - grade accordingly.
- **Exploit-DB / GitHub** - public PoC/exploit availability signals real-world risk and timing (for
  *awareness and prioritization*, not for offensive use).

### 5.2 Adversary knowledge bases
- **MITRE ATT&CK** - technique/tactic taxonomy, group and software pages, mitigations (§4.1).

### 5.3 Threat-sharing platforms & feeds
- **MISP** - open-source threat-intel sharing platform; community/trust-based IOC storage,
  correlation, and export to SIEM/firewall/IDS; natively TLP-aware; `to_ids` flagging separates
  detection-grade indicators from context. (PyMISP for automation.)
- **OpenCTI** - self-hosted CTI knowledge platform (STIX-native, connects feeds, ATT&CK, actors).
- **AlienVault OTX** - community IOC "pulses."
- **abuse.ch** - the essential free C2/malware trackers: **URLhaus** (malware-distribution URLs),
  **ThreatFox** (IOCs by malware family), **Feodo Tracker** (botnet C2), **SSL Blacklist**,
  **MalwareBazaar** (samples).

### 5.4 Enrichment & internet-observation
- **VirusTotal** - multi-engine file/URL/domain/IP reputation and, critically, **relationships** for
  pivoting (contacted domains/IPs, sibling samples, communicating files); VT Intelligence supports
  retrohunt-style searches (defensive/authorized use).
- **GreyNoise** - labels internet **background noise** (mass scanners) so you can suppress the
  ubiquitous and focus on the *targeted*; a strong false-positive reducer.
- **Shodan / Censys** - internet-wide host/service observation; identify exposed services, malware C2
  panels, and - turned inward - your own exposed assets (§7). Passive (their scan, not yours).
- **AbuseIPDB, Pulsedive, URLScan.io** - reputation and URL detonation/enrichment.

### 5.5 Human, leak, and forum signals
- **Paste / leak monitoring** - Pastebin-class sites, paste aggregators, and breach indices for
  **your own organization's** exposed data (credentials, keys, source). This is authorized
  self-exposure monitoring - see `legal-ethics-opsec.md` §5; monitor and remediate, never traffic in
  or use third-party breach data.
- **Social / forum monitoring (SOCMINT)** - actor claims, leak announcements, targeting chatter,
  brand/exec mentions. Deep/dark-web forums and ransomware leak sites for mentions of your org (many
  are frequently seized/relaunched). Monitoring only, within legal and ToS bounds.
- **Commercial (awareness)** - Recorded Future, Mandiant Advantage, CrowdStrike, Flashpoint, Intel
  471, Digital Shadows/ReliaQuest, SpyCloud (deeper access; corroborate, don't treat as gospel).

### 5.6 TLP for sharing (pointer)

Everything you *disseminate* from these sources carries a **TLP marking** (RED / AMBER+STRICT /
AMBER / GREEN / CLEAR). The full ruleset - need-to-know, "marking must not be changed on
re-transmission," RED to named individuals only - lives in `legal-ethics-opsec.md`. Respect the TLP
of any intelligence you *receive*, and mark everything you produce.

---

## 6. Producing a CTI product

### 6.1 The threat report / threat profile

Use the OSINT report template (`methodology.md` §7.1) - requirement → BLUF → graded/sourced findings
→ analysis + confidence → gaps → recommendations - specialized for CTI:

```
CTI REPORT - <title>
Reference: <id>   Date (UTC): <YYYY-MM-DD>   TLP: <marking>
Prepared for: <audience/altitude>   Prepared by: <analyst>

1. INTELLIGENCE REQUIREMENT  - the IR/PIR answered; scope; time window.
2. BLUF / KEY JUDGMENTS      - the assessment in estimative language + analytic
                               confidence; written for the target altitude (§1).
3. THREAT OVERVIEW           - actor(s)/campaign; motivation; targeting; why it
                               matters to THIS org.
4. TTPs                      - mapped to MITRE ATT&CK IDs; narrated against the
                               Kill Chain; per-vertex Diamond notes where useful.
5. INDICATORS (IOCs)         - table: indicator | type | context | first/last seen
                               | Admiralty grade | confidence | expiry | to_ids?
                               (Prefer TTPs over atomic IOCs; lift IOCs to techniques.)
6. INFRASTRUCTURE / ATTRIBUTION - clusters (technical, high-confidence) vs actor
                               naming (caveated, ACH-tested); alternatives considered.
7. IMPACT / RISK ASSESSMENT  - "so what" for the consumer's decision.
8. INTELLIGENCE GAPS         - what could NOT be established, and why.
9. RECOMMENDATIONS / DETECTIONS - prioritized actions; detection logic (§6.3).
10. APPENDICES               - evidence manifest, full IOC list, references.
```

### 6.2 Confidence & estimative language (pointer)

CTI lives and dies on this. Separate **source reliability/credibility** (Admiralty grade on each IOC
and report) from **analytic confidence** (trust in your own judgment), and express likelihood with a
consistent estimative vocabulary - full treatment in `methodology.md` §6. Attribution and strategic
assessments *must* be caveated; a false "high confidence" attribution can misdirect an entire response.

### 6.3 Feeding detections (Sigma / YARA / Snort - awareness)

The output of tactical/technical CTI is often a **detection rule**, so a CTI analyst should be
detection-literate even if a detection engineer writes the rule:

- **Sigma** - generic, SIEM-agnostic detection rules over **log events** (the "YARA for logs");
  expresses *behavioral* detections and therefore targets the top of the Pyramid of Pain. Convertible
  to Splunk/Elastic/Sentinel queries.
- **YARA** - pattern-matching rules over **files/memory** (strings, byte patterns, conditions) to
  identify malware families - capability-level detection.
- **Snort/Suricata** - **network** signatures over traffic/packets (C2 patterns, exploit signatures).

Doctrine: prefer **behavioral (Sigma/YARA-family) detections tied to ATT&CK techniques** over brittle
atomic-IOC blocklists - they survive infrastructure rotation and impose real adversary cost (§3.2).
Deliver detections with their ATT&CK mapping and a false-positive note.

### 6.4 Dissemination & sharing (pointer)

Push at the right altitude to the right consumer (§1), at the correct **TLP** marking, through an
appropriate channel (MISP/OpenCTI for machine-consumable feeds; briefings/reports for prose). STIX/TAXII
for structured exchange. Full sharing governance, channel OPSEC, and TLP rules: `legal-ethics-opsec.md`.
Then capture feedback - was it actionable? - and re-task.

---

## 7. Defensive OSINT: attack-surface, brand & exec monitoring

The dual-use core of CTI: **run the adversary's reconnaissance playbook against your own (or a
consenting client's) organization first, and monitor continuously.** This is authorized
self-assessment - see `legal-ethics-opsec.md` §1 (own assets, or written client authorization).

### 7.1 External attack-surface monitoring
- Enumerate your own exposed footprint the way an attacker would (passively): subdomains
  (Certificate Transparency, passive sources), exposed services and versions (Shodan/Censys `org:` /
  `net:` filters), forgotten dev/test/staging environments, expired-but-live assets, exposed panels,
  and leaked secrets in public repos.
- Prioritize by exploitability (cross-reference exposed versions against CVE/NVD + CISA KEV) and feed
  the result to vulnerability management. This is the mirror image of ATT&CK Reconnaissance (§4.1).

### 7.2 Brand / typosquat / lookalike-domain monitoring
- Watch for **lookalike domains** targeting your brand - homoglyph, bitsquatting, transposition,
  omission, addition, and TLD variants (tooling class: dnstwist and similar). Flag those that are
  *registered*, have **MX records** (mail-capable → phishing/BEC ready), or carry a TLS certificate
  (live → active).
- Monitor **Certificate Transparency in near-real-time** (CertStream-class) for certificates issued
  on your brand strings or lookalikes - often the earliest signal of a phishing site being stood up.

### 7.3 Credential / data-leak monitoring (own org)
- Monitor breach indices and paste sites for **your** domains/employees: exposed credentials,
  infostealer ("stealer") logs, leaked API keys and source. Even old, dead passwords yield valid
  **username/identifier** lists that feed credential-stuffing against you - so they matter defensively.
- Remediate (force resets, rotate keys, enforce MFA); monitor for reuse. **Awareness and remediation
  only** - never purchase, redistribute, or operationally use third-party breach data
  (`legal-ethics-opsec.md` §5, §2).

### 7.4 Executive / VIP exposure
- Assess the digital footprint of high-value individuals (execs, admins) as attackers do: exposed
  personal data enabling spear-phishing, whaling, or account takeover. Report defensively (reduce
  footprint, harden accounts) - this is protective, and must respect PII/GDPR and the individual's
  consent framing (`legal-ethics-opsec.md` §4).

### 7.5 Standing it up as a program
A defensive OSINT program is **continuous, not a one-off audit**: standing collection against the
above, tied to Intelligence Requirements (§2.1), feeding detections and vulnerability management,
with feedback. The TechSphere-style close-out - centralize asset inventory, isolate test
environments, enforce MFA + password manager, secure repos with a secrets manager, and *keep
monitoring* - is the recurring "so what" of an attack-surface assessment.

---

## 8. Regulatory-driven testing context: TIBER-EU / DORA TLPT

Regulation now makes **intelligence-led** offensive testing mandatory in some sectors, and OSINT-derived
CTI is the fuel. Brief awareness (the OSINT-relevant part):

- **TLPT - Threat-Led Penetration Testing** - an intelligence-driven simulation of realistic attacks
  against an organization's **Critical or Important Functions**, measuring operational resilience, not
  just vulnerability count. It is *led by* threat intelligence: real adversary TTPs shape the scenarios.
- **TIBER-EU** - the ECB's *Threat Intelligence-Based Ethical Red Teaming* framework that structures
  such tests across the EU.
- **DORA (Art. 26)** - makes TLPT a regulatory obligation for significant financial-sector entities
  (broadly, at least every three years, on **production** systems, supervised by national TIBER cyber
  teams).
- **The OSINT/CTI role.** TLPT is founded on a threat-intelligence phase: a **Generic Threat
  Landscape** feeds a **Targeted Threat Intelligence Report (TTIR)** that documents the org's digital
  footprint, external exposure, credential leaks, underground-forum mentions, and the likely
  adversaries plus their TTPs. That TTIR - largely OSINT-derived - is the direct bridge from
  intelligence to the red team's scenario design; threat modelling explicitly includes exposed assets,
  lookalike/typosquat domains, and exploitable info leaks as realistic initial-entry points.
- **Framing.** This is **authorized, scoped, non-destructive** testing under a signed Rules of
  Engagement and Scoping & Specification Document, with data securely destroyed afterward. The OSINT
  contribution is the *reconnaissance and threat-scenario* input - squarely defensive/authorized. The
  actual red-team execution is out of this skill's scope; see `legal-ethics-opsec.md` for the
  authorization and boundary doctrine that governs any such engagement.

---

## 9. CTI quick-reference

- **Four types → four audiences:** strategic (leadership), operational (intel/IR), tactical
  (SOC/detection), technical (tooling). Match the product to the decision.
- **Same cycle** as `methodology.md` §1; the CTI failure mode is skipping **requirements** -
  collect against IRs tied to real risk, not a firehose.
- **Pyramid of Pain:** block hashes/IPs for hygiene; **invest at the top - TTPs** - because behavior
  is expensive for the adversary to change. Lift every atomic IOC to its ATT&CK technique.
- **Three frameworks:** ATT&CK (technique detail, common language), Kill Chain (sequence/narrative),
  Diamond (four-vertex pivoting engine). Use all three.
- **Pivot infrastructure passively** (passive DNS, certs, WHOIS, ASN, fingerprints) to *cluster*
  malicious infra for blocking/monitoring - but **cluster ≠ attribution**; do not over-attribute; use
  ACH + estimative confidence; false flags are real.
- **Sources:** CVE/NVD + CISA KEV, ATT&CK, MISP/OpenCTI/OTX, abuse.ch (URLhaus/ThreatFox/Feodo),
  VirusTotal, GreyNoise, Shodan/Censys, own-exposure leak/paste monitoring. Corroborate; grade; mark TLP.
- **Produce** at the right altitude, with graded/sourced IOCs, ATT&CK-mapped TTPs, caveated
  attribution, explicit gaps, and detection logic (Sigma/YARA/Snort tied to ATT&CK). Disseminate at
  the right TLP; capture feedback.
- **Defensive OSINT program:** attack-surface + brand/typosquat + own-credential-leak + exec-exposure
  monitoring, continuously, against your own or a consenting client's org.
- **Regulatory:** TIBER-EU / DORA TLPT are intelligence-led; the OSINT-derived TTIR is the bridge from
  CTI to authorized scenario-based testing.

> Threads carried from `methodology.md`: authorization before collection · passive before active ·
> corroborate before asserting · grade every fact · separate source reliability from analytic
> confidence · state your gaps · **public ≠ permitted and public ≠ true** · and, for CTI specifically,
> **cluster ≠ attribution - do not over-attribute.**
