# Legal, Ethics, Authorization & OPSEC - the Gate

> **Read this file before conducting any OSINT work with this skill.** It is loaded on
> demand and it is not optional. It defines the boundary between legitimate open-source
> intelligence and conduct that is unlawful, unethical, or dangerous. If a request cannot
> be satisfied without crossing a line described here, this skill **refuses** and says so.

---

## 0. THE CORE RULE - read this first

> ## **"Publicly accessible" ≠ "permitted to use."**
>
> Accessibility grants no right of use. Data being reachable - via a search engine, an
> open directory, a cached page, a misconfigured server, or a leaked dump - does **not**
> make it lawful, ethical, or in-scope to collect, retain, cross-reference, or act upon.

OSINT, as practiced by this skill, is exactly one thing:

**The passive collection of lawfully-accessible open sources, for an authorized purpose,
followed by analysis, corroboration, and contextualization into an intelligence product.**

It is **NOT**:

- hacking or any unauthorized access to a system;
- bypassing authentication, paywalls, rate limits, CAPTCHAs, or any access control;
- using found, guessed, purchased, or leaked credentials;
- impersonating a person or organization *in order to gain access* to non-public content;
- direct interaction with a target's systems in a way the target did not invite;
- surveillance, tracking, or profiling of private individuals without a lawful basis.

Two doctrinal distinctions carry the whole discipline. Keep them in front of you at all times.

**OSINF is not OSINT.** Open-source *information* is raw, un-analysed data. OSINT is that
information *after* it has passed the intelligence cycle - evaluated for source reliability,
corroborated across independent sources, analysed, and contextualized. The value, and the
professional responsibility, lives in the analysis, not in the collection. Never ship raw
findings as if they were intelligence.

**"Open source" means the *source* is open, not the *content*.** A leaked mailbox, a
hacked database, or a credential dump is **not** an open source, even after its contents
begin circulating publicly. The origin taints it. Treat such material under the breach-data
rules in §5, not as ordinary open-source material.

---

## 1. Authorization & scope - establish these before you collect anything

No collection begins until authorization and scope are confirmed. This is the first stage
of the intelligence cycle (planning & direction) for a reason: an investigation without a
defined question and a defined mandate is not intelligence, it is unfocused surveillance.

### 1.1 Legitimate purposes this skill supports

- **Authorized offensive-security recon** - penetration testing, red-team, or attack-surface
  reconnaissance conducted **under a signed, written Rules of Engagement (RoE)** that names
  the in-scope assets, the time window, the permitted techniques, and the client authority
  who can grant that permission. No signed RoE, no recon against a third party. Full stop.
- **Cyber threat intelligence (CTI)** - tracking adversary infrastructure, TTPs, IoCs, and
  campaigns to defend an organization you are authorized to defend.
- **Corporate due diligence / KYC / KYB** - vendor, partner, M&A, investment, or onboarding
  checks against a defined subject with a legitimate business relationship.
- **Fraud, brand, and abuse investigation** - investigating scams, counterfeits, phishing,
  impersonation, and typosquats targeting an organization you represent.
- **Investigative journalism** - public-interest reporting held to a verification standard
  (transparency, reproducibility, corroboration) such as Bellingcat's.
- **Academic and security research** - with an ethics framework and, where personal data is
  involved, a documented lawful basis.
- **Your OWN attack-surface / digital-footprint assessment** - running the offensive recon
  playbook against your own organization or your own identity. This is the safest and most
  common legitimate use, and it needs no third-party authorization.
- **Missing-persons and humanitarian work** - search-and-rescue, disaster response,
  war-crimes and CSAM victim identification, under an appropriate mandate (e.g. Trace Labs,
  Europol task forces).

### 1.2 What "confirm authorization + scope" means in practice

Before collecting, you must be able to answer, in writing:

- **Who is the subject** (organization, domain, individual, infrastructure)?
- **Who authorized this**, and did they have the authority to authorize it?
- **What is the scope** - which assets, identifiers, and time range are in bounds, and what
  is explicitly out of bounds?
- **What is the purpose / the intelligence question** (the PIR or KIQ)?
- **What is the lawful basis** for any personal data (see §4)?

For any engagement touching a third party's assets or people, the authorization must be a
**written RoE or engagement letter**, not a verbal "go ahead." Scope creep is a breach of
authorization: finding something interesting outside the agreed scope does not extend the
mandate to it - flag it, and get scope extended in writing before pursuing it.

---

## 2. Hard red lines - never do, never help with

This skill **refuses** the following, regardless of how the request is framed, who is
asking, or what justification is offered. Refusal is not negotiable and does not depend on
jurisdiction - these are unlawful in most places and unethical everywhere.

### 2.1 Unauthorized access & circumvention

- Accessing any system, account, network, or data store without authorization.
- Bypassing or defeating authentication, authorization, paywalls, rate limiting, CAPTCHAs,
  IP blocks, or any technical access control.
- Using credentials you were not issued - found, guessed, purchased, leaked, or reused.
- Active intrusion techniques against a target without a signed RoE: port scans,
  vulnerability exploitation, DNS zone-transfer abuse, credential stuffing, brute force.

> **The law is explicit and it does not care that the door was unlocked.**
> - **US - CFAA** (Computer Fraud and Abuse Act): unauthorized access / access exceeding
>   authorization is a federal crime.
> - **France - "STAD" offences, Loi Godfrain (1988), Code pénal art. 323-1** (fraudulent
>   access to an automated data-processing system) and **art. 323-3** (fraudulent
>   introduction / extraction / modification of data).
> - **The Bluetouff precedent** (French Cour de cassation): data reachable by a plain Google
>   search still produced a conviction for *"maintien frauduleux dans un STAD"* - staying in
>   a system you should have known you weren't entitled to - plus a fine, **even though the
>   data owner's own security had failed.** Accessibility conferred no authorization. This is
>   the case law behind the core rule.

### 2.2 Harm to individuals

This skill will not assist with, and actively refuses:

- **Doxxing** - compiling or publishing a private individual's personal details to expose,
  intimidate, or endanger them.
- **Stalking, harassment, or intimidation** of any individual.
- **Surveillance of private individuals without a lawful basis** - building a profile of a
  private person's movements, relationships, or life from their scattered public traces
  beyond what a legitimate, proportionate, authorized purpose justifies.
- **Deanonymizing** a person where doing so would endanger them (activists, journalists,
  survivors, sources, dissidents).
- **Targeting special-category / protected data** to profile or harm - health, sex life,
  sexual orientation, religion, political opinion, trade-union membership, ethnicity,
  biometrics, genetics.
- **Intimate-image, minor-related, or grooming-adjacent collection** of any kind.

### 2.3 Trafficking in and weaponizing illicit data

- Buying, selling, soliciting, or using **stolen breach data** to harm individuals.
- Knowingly handling data obtained through a crime and passing it on - in France this can
  be **recel** (handling stolen goods, Code pénal art. 321-1) layered on top of the
  underlying data offence.

### 2.4 Enabling a downstream crime

- Any collection whose evident purpose is to enable fraud, extortion, blackmail, social
  engineering against a victim, physical harm, retaliatory action, or unauthorized access.
- Impersonation-for-access (see §6.3) and pretexting to extract information from people.

**If a request requires any of the above, refuse, explain briefly why, and - where there is
a legitimate underlying goal - offer the lawful, in-scope alternative (e.g. "I can't profile
that private individual, but I can assess *your own* exposed footprint").**

---

## 3. Passive-first - the intrusiveness ladder

Collection intrusiveness is a spectrum, and it maps directly onto legal risk. Always operate
at the lowest rung that answers the question.

| Rung | What it is | Legal / OPSEC posture |
|------|-----------|------------------------|
| **Passive** | No interaction with the target's systems. Third-party sources, search-engine caches, CT logs, passive DNS, WHOIS history, already-scanned data (Shodan/Censys), archived pages. | Generally lawful without authorization; undetectable; **the default**. |
| **Semi-passive** | Light, normal-looking interaction indistinguishable from an ordinary visitor. | Low risk, but you are now touching the target. |
| **Active** | Direct interaction: port scans, DNS brute force / AXFR, connection attempts, intensive crawling, live probing. | Generates logs, is detectable, and **against a third party can constitute intrusion** under CFAA / art. 323 CP / Loi Godfrain. **Requires written authorization.** |

Two operational consequences:

- **Do not tip off the target.** Passive-first also protects the investigation: active
  contact can alert a target, who may then mount a counter-investigation, destroy evidence,
  or spear-phish the investigator.
- **The line is often in the *use*, not the *look*.** Google dorking is lawful (it uses a
  public engine's own features). *Using* a credential or config file the dork surfaced is
  fraudulent access. "The frontier is in the use, not the search."

---

## 4. Data protection - GDPR and the personal-data regime

Public personal data is still personal data. A visible LinkedIn profile, a directory phone
number, a photo of an identifiable person - all are personal data the moment they concern an
identified or identifiable individual, and the full data-protection regime applies to how you
collect, store, and process them.

### 4.1 GDPR principles that bind an OSINT investigator

- **Lawful basis (art. 6).** You need one. For defensive enterprise OSINT it is usually
  **legitimate interest (art. 6.1.f)** - asset protection, threat detection, compliance -
  but legitimate interest requires a **documented balancing test** against the data subject's
  rights and freedoms. Other bases: **consent** (someone commissioning OSINT on themselves)
  or **legal obligation**.
- **Purpose limitation (art. 5.1.b).** Data collected for a security audit cannot be reused
  for HR, marketing, or surveillance.
- **Data minimization (art. 5.1.c).** Collect only what is strictly necessary. This forbids
  mass, untargeted aspiration of profiles. Mapping a person's entire digital life off one
  posted badge photo is not minimized - it is disproportionate.
- **Storage limitation (art. 5.1.e).** Retain only as long as the purpose requires; then
  securely delete (see §7).
- **Right to erasure / right to be forgotten (art. 17).** Retaining de-referenced or erased
  data (pulled from Wayback or caches) in an internal OSINT base can itself violate art. 17,
  even though the copy is technically still reachable.
- **Records of processing (art. 30)** and the **information obligation (art. 14)** may apply.

### 4.2 Special-category data (art. 9)

Health, sex life, sexual orientation, religion, political opinion, trade-union membership,
racial/ethnic origin, biometric and genetic data get **heightened protection**. Do not make
these the target of collection or profiling absent a specific, narrow, documented lawful
basis. When in doubt, exclude them (an internal OSINT charter typically hard-excludes health,
political opinion, and sex life).

### 4.3 The Clearview AI precedent - scraping PII at scale is unlawful

The French **CNIL fined Clearview AI €20 million** (deliberation SAN-2021 series) for
scraping billions of publicly accessible photos and building a facial-recognition database
without a lawful basis. **The public character of the data was neither justification nor
excuse.** This is the reference case for the whole domain: automated, mass collection and
biometric processing of public PII is not made lawful by the data being public. The **EU AI
Act** independently bans untargeted facial-image scraping to build recognition databases.

### 4.4 French Code pénal and Code civil provisions to respect

- **Art. 226-1 CP** - invasion of privacy (capturing/recording/transmitting a person's words
  or image in a private place without consent): up to 1 year / €45,000. **Art. 226-2** -
  retaining/disclosing/using such recordings.
- **Art. 226-18 CP** - collecting personal data by **fraudulent, unfair, or unlawful means**
  (*"frauduleux, déloyal ou illicite"*): up to 5 years / €300,000. "Unfair" can capture
  collection unbeknownst to the subject without a valid legal basis, **even if the data is
  technically accessible**.
- **Art. 226-4-1 CP** - identity theft / usurpation d'identité: creating a fake account with
  a false identity to reach private-account content can qualify (see §6.3).
- **Art. 9 Code civil** - right to privacy and right to one's image (*droit à l'image*).
  Cour de cassation (25 Nov 2020, n° 17-19.523): online collection of even public information
  **can be a privacy breach once it exceeds what the legitimate purpose justifies** - a direct
  restatement of the proportionality principle.

### 4.5 Terms of Service - a real, if civil, boundary

ToS violations are generally a **civil / contractual** matter, not a criminal one, but they
are still a boundary: they can void the admissibility of evidence, expose the investigator to
account bans and civil liability, and - where the ToS violation is the vehicle for a false
identity used to reach private content - feed a criminal impersonation claim. Respect ToS;
many platforms prohibit scraping and automated collection outright. Where a ToS forbids what
the task needs, that is a signal to reconsider, not a hurdle to route around.

### 4.6 International frames (awareness)

- **Council of Europe Convention on Cybercrime (Budapest, 2001)** - the treaty backbone for
  cross-border computer-crime law; harmonizes the unauthorized-access offences.
- **Tallinn Manual** - scholarship on how international law applies to cyber operations;
  relevant context for state-adjacent work.
- **NIS2 (EU Dir. 2022/2555)** - mandates threat-intelligence and attack-surface work for
  essential/important entities, which *strengthens* the legitimate-interest basis for
  defensive OSINT (an uncatalogued attack surface is itself a compliance gap).

Keep this practitioner-usable: the point is not to recite statutes but to recognize the line
before you step over it, and to be able to document why what you did was lawful and in scope.

---

## 5. Breach & leak data - awareness vs. use

There is a bright line between *checking exposure* and *trafficking / weaponizing*.

- **Legitimate:** checking whether **your own organization** or an **authorized in-scope
  subject** appears in known breaches - via services like Have I Been Pwned, or by monitoring
  paste sites and dark-web leak sites for your own leaked credentials and data. This is
  defensive, it is proportionate, and it is often a compliance obligation.
- **Not legitimate:** downloading, aggregating, redistributing, or querying breach corpora to
  obtain other people's credentials or PII; using leaked credentials to access anything;
  buying stolen data; or handing breach-sourced personal data to a client to *harm* the data
  subjects.

Remember §0: a leaked dump is **not an open source**. Awareness of an exposure is fine;
possession and use of the underlying stolen data to harm is where it becomes handling stolen
goods (recel) and, if used to access systems, fraudulent access.

---

## 6. OPSEC & attribution management

Protecting the investigation and the investigator is part of doing this lawfully and safely.
A target may have equal or greater recon skills and may counter-investigate.

### 6.1 Never expose the investigator's real IP or identity to a target

This is both an OPSEC rule and a firm collection policy:

> **Route all collection through the connected scraping / search MCP tools and managed APIs
> (Firecrawl, SerpApi, Tavily, Exa, Apify Actors, and the other configured providers) - NEVER
> from the local machine.** No direct HTTP to a third-party target from this host. No local
> `requests` / `httpx` / `curl` / browser automation against targets. The local IP is never
> exposed to a target.

This dovetails with the no-local-scraping posture above. For any live
retrieval, the request must egress through a managed tool, not the workstation.

### 6.2 Environment isolation & compartmentation

- Use a **dedicated research environment / browser** (hardened profile, anti-fingerprinting)
  separated from your personal identity and accounts. A snapshottable VM is ideal.
- **Compartmentalize** personal, professional, and operational identities. Never log into a
  personal account on investigation infrastructure; never cross-contaminate personas.
- Test any new tool in an isolated environment first - "today's wonder tool may be tomorrow's
  security liability." Prefer local, open-source tools verified at their **code repository**
  (GitHub/GitLab), not lookalike "service" sites that clone a real tool's branding to harvest
  your query data or serve malware. Web OSINT services receive *your* query data and engage
  *your own* data-protection compliance, not just theirs.

### 6.3 Sock-puppet / research-account ethics - and where they become unlawful

Research personas ("sock puppets") have legitimate uses (protecting the investigator's
identity, journalism, licensed investigation). They also carry hard limits:

- **They violate most platforms' Terms of Service** (ban risk, evidence-admissibility risk).
- **Using a false identity to access otherwise-private content can be criminal
  impersonation** - France: usurpation d'identité, **art. 226-4-1 CP**; platform ToS (e.g.
  Instagram) explicitly forbid impersonation. Creating a fake account to view a *public*
  timeline is a different matter from using one to breach a *private* one - the latter is the
  line, and this skill stays on the lawful side of it.
- A persona used to **contact, deceive, or pretext** a target crosses out of passive OSINT
  entirely into social engineering - refused (§2.4).

Legitimate persona use is passive-observation-only, ToS-limited, and never a vehicle to reach
non-public content or to interact with targets.

### 6.4 Do not tip off the target

Prefer passive collection; avoid active probing that generates logs or alerts; do not contact
the subject; keep operational activity off channels the target can observe.

---

## 7. Evidence handling & chain of custody

If the work may ever be relied upon - in a report, a disclosure, litigation, or a
law-enforcement referral - it must be preserved so that it is traceable, reproducible, and
tamper-evident. Online content vanishes; capture it at the moment of collection.

**Golden rules (apply on every significant finding):**

1. **Timestamp everything in UTC** - collection date and time.
2. **Screenshot** the finding (full page where possible).
3. **Archive the page** to an independent third-party witness - **archive.today** and/or the
   **Wayback Machine** (WaybackPy `--save`) - so the record does not depend on you or the
   source staying online.
4. **Hash the artifacts** - `sha256sum` every captured file into a manifest.
5. **Log the provenance** - source URL, the tool used, who collected it, and when. Distinguish
   **verified facts** from **hypotheses** in the log.
6. **Keep an original + a working copy** in separate locations; re-check integrity periodically.

**Suggested case structure:** a per-case directory with `case_info.md`, `evidence_log.md`
(columns: UTC timestamp | type | description | SHA256 | source/tool), and subfolders for
screenshots, archives, files, and hashes. Archival toolchain: self-contained HTML archive
(Monolith) → full-page screenshot → Wayback save → hash manifest.

**Retention & secure deletion.** Retain only as long as the purpose and any legal obligation
require (GDPR storage limitation, §4.1). Define a retention period up front and securely
delete when it lapses. Storing erased/de-referenced personal data indefinitely is itself a
violation (art. 17).

**Sharing - the Traffic Light Protocol (TLP v2.0, FIRST).** Mark every product you share and
respect the marking on anything you receive:

| Marking | Who may see it |
|---------|----------------|
| **TLP:RED** | Named individual recipients only; no onward sharing; individual addresses, never generic mailboxes. |
| **TLP:AMBER+STRICT** | Recipient's organization only, need-to-know. |
| **TLP:AMBER** | Recipient's organization and its clients, on a need-to-know basis. |
| **TLP:GREEN** | The wider community. |
| **TLP:CLEAR** | Unrestricted. |

Rules: a marking may be *relaxed* over time as risk drops, but the **TLP marking must NOT be
changed on re-transmission**. Share on a **need-to-know** basis. De-personalize / anonymize
where the recipient does not need the identities.

---

## 8. Pre-engagement checklist & STOP conditions

### 8.1 Pre-engagement checklist - all boxes before collection

- [ ] **Authorization** - is there a signed RoE / engagement letter / documented mandate, from
      someone with authority to grant it? (Self-footprint assessment needs no third-party
      authorization.)
- [ ] **Scope** - are the in-scope subjects, assets, identifiers, and time window written down,
      with the out-of-scope items named?
- [ ] **Purpose** - is the intelligence question (PIR/KIQ) defined?
- [ ] **Lawful basis** - for any personal data, is the GDPR basis identified and, for
      legitimate interest, is the balancing test documented? Are special categories excluded?
- [ ] **Passive-first** - is the plan the least-intrusive path that answers the question? Is
      any active step covered by explicit written authorization?
- [ ] **OPSEC set** - collection routed through managed MCP tools / APIs (never the local IP);
      dedicated environment; personas (if any) ToS-limited and observation-only; compartmented.
- [ ] **Evidence handling** - timestamp + screenshot + archive + hash + provenance-log workflow
      ready before the first capture.
- [ ] **Retention plan** - retention period and secure-deletion trigger defined; TLP marking
      chosen for the eventual product.

### 8.2 STOP - do not proceed if…

- there is **no authorization** and the work touches a third party's assets or people;
- the request needs **bypassing any access control**, or **using any credential** you were not
  issued;
- the target is a **private individual** and the purpose is exposure, tracking, or profiling
  without a lawful, proportionate basis (doxxing / stalking / surveillance);
- the request is to **buy, use, or redistribute stolen / breach data** to reach or harm people;
- the plan relies on a **false identity to access non-public content**, or on **contacting /
  deceiving / pretexting** a target;
- the goal is to **deanonymize** someone in a way that could endanger them;
- the target is **special-category data** (health, sex life, religion, politics, ethnicity,
  biometrics) used to profile or harm;
- the evident purpose is to **enable a crime** (fraud, extortion, blackmail, unauthorized
  access, retaliation);
- proceeding would require **local scraping / direct HTTP to a target from this machine**;
- you find yourself **outside the agreed scope** - stop, flag it, get scope extended in
  writing before continuing.

When a STOP condition fires, the skill halts, states plainly which line was reached, and - if
a legitimate goal exists behind the request - offers the lawful, in-scope alternative.

---

## 9. Ethics beyond legality

Lawful is the floor, not the ceiling. *"Ce qui est légal n'est pas toujours éthique."*

- **Proportionality.** The depth and breadth of an investigation must match the threat or the
  question. Checking company credentials in breach DBs is proportionate; reconstructing an
  employee's whole private life from one badge photo is not.
- **Do no harm.** Weigh the impact of anything you collect and, especially, anything you
  disclose. Sensitive or misinterpreted findings can cause real harm to real people.
- **Transparency & verifiability** (the Bellingcat standard): be transparent about method,
  make findings reproducible, corroborate across independent sources before asserting, and
  link auditable evidence. This is also the antidote to disinformation and to careless
  doxxing - OSINT can be a *vector for* disinformation, and rigor is the defence.
- **Objectivity & bias.** Be conscious of your own biases; use structured techniques such as
  Analysis of Competing Hypotheses; seek disconfirming evidence, not just confirmation.
- **Responsible disclosure.** When you find a vulnerability or exposure, report it to the owner
  / CERT through a responsible-disclosure channel; do not publicize or exploit it.

The legitimacy of OSINT depends on **purpose and restraint**, never on the data being public.
That is the thread running through this entire file, and it is the standard this skill holds
itself and its user to.
