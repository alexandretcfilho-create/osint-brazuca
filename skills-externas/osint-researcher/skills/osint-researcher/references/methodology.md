# OSINT Methodology - Reference

> Loaded on demand. This file is the analytic backbone of the `osint-researcher` skill:
> the intelligence cycle, the intrusiveness ladder, source evaluation (Admiralty code),
> verification and anti-disinformation, analysis techniques, confidence/estimative language,
> and reporting. Scope is **legitimate, authorized, ethical OSINT** only - self-assessment,
> consented due diligence, sanctioned threat intelligence, investigative journalism, academic
> research, authorized missing-persons work. For legal boundaries, PII/GDPR, ToS, sock-puppet
> policy, evidence handling and investigator OPSEC, see `legal-ethics-opsec.md` (cross-referenced
> throughout). For the tool/source catalogue by discipline, see `tools-catalog.md`.
>
> One rule sits above all methodology: **"public" ≠ "permitted" and "public" ≠ "true."**
> Accessibility never confers authorization, and availability never confers accuracy.

---

## 1. The intelligence / OSINT cycle

Intelligence is produced, not found. The cycle is the discipline that turns scattered open-source
*data* into a decision-ready *product*. It is **iterative** - feedback re-enters planning - and
every legitimate engagement runs through it end to end. Different sources number the stages
differently (5-stage joint-doctrine, 6-stage CTI, 7-step OSINF-centric); they are the same skeleton.

| # | Stage | Also called | Core question | Key outputs |
|---|-------|-------------|---------------|-------------|
| 1 | **Planning & Direction** | Requirements definition, tasking, scoping | *What must we answer, for whom, by when, and are we authorized?* | PIRs/KIQs, EEIs, scope, authorization, source plan, OPSEC level |
| 2 | **Collection** | Gathering, footprinting | *Where does the answer live and how do we obtain it (least-intrusive first)?* | Raw findings, captures, hashes, source metadata |
| 3 | **Processing / Collation** | Normalization | *How do we make raw data usable and de-duplicated?* | Parsed, deduped, normalized, enriched, translated data |
| 4 | **Analysis & Production** | Assessment | *What does it mean, how confident are we, and so what?* | Graded findings, hypotheses, assessment, the intelligence product |
| 5 | **Dissemination & Feedback** | Sharing, reporting | *Who needs this, in what form, and did it answer the question?* | Report delivered to the right audience + feedback loop |

### 1.1 Planning & Direction (the stage that is most often skipped)

Everything downstream is only as good as the requirements set here.

- **PIR - Priority Intelligence Requirement**: the top-level question the requester needs answered
  (e.g. "What is our organization's externally exposed attack surface?"). Also framed as **KIQs**
  (Key Intelligence Questions).
- **EEI - Essential Elements of Information**: the specific sub-facts that, together, answer a PIR
  (e.g. active subdomains, exposed services + versions, leaked credentials, employee email format).
  EEIs are what you actually go collect.
- **Scope**: the explicit boundaries - which organizations, individuals, infrastructure, identifiers,
  and **time range** are in bounds; what is out of bounds.
- **Authorization / legal standing**: confirmed *before* collection. Written authorization for
  engagements against third parties (pentest/Red Team → signed Rules of Engagement + Scoping &
  Specification Document). See `legal-ethics-opsec.md`.
- **Source plan & prioritization**: which sources map to which EEI, ranked by expected value and
  intrusiveness.
- **OPSEC sensitivity level**: how covert must collection be; is attribution to the investigator a
  risk. Drives the intrusiveness ceiling (see §2).

Deliverable of this stage: a short **collection plan** - a table of `EEI → candidate sources →
intrusiveness → priority`.

### 1.2 Collection

Gather data against the EEIs, **least-intrusive first** (§2). Ensure findings are relevant, accurate,
and timely. Capture provenance *at the moment of collection* - URL, UTC timestamp, tool used, and a
hash/screenshot/archive (chain-of-evidence, §7.2). Pivot as new identifiers surface (§5.3).

### 1.3 Processing / Collation

Convert raw collection into an exploitable, comparable form *before* interpreting it: parse, **dedupe**,
normalize (consistent formats for dates/names/addresses), enrich (resolve IPs, expand identifiers),
translate, and triage by importance/urgency. This is mechanical hygiene - it is *not* analysis, and
conflating the two is a common error that lets raw noise masquerade as findings.

### 1.4 Analysis & Production

Interpret the processed data: connect entities, build timelines, test hypotheses, mitigate bias
(§5), grade each fact (§3), assign confidence (§6), and answer the **"so what."** Discard material of
insufficient quality. Produce a report tailored to the audience that is **accurate and actionable**.
The analysis *is* the value - never ship raw findings as if they were intelligence.

### 1.5 Dissemination & Feedback

Deliver the product to the right stakeholders, **in the right form, securely, with enough context to
act.** Apply the correct handling/sharing marking (TLP - see `legal-ethics-opsec.md`). Then close the
loop: collect feedback ("did this answer the PIR? what is still missing?"), which re-tasks the cycle.

### 1.6 OSINF vs OSINT - the load-bearing distinction

| | OSINF (open-source **information**) | OSINT (open-source **intelligence**) |
|---|---|---|
| What | Collected data shared **as-is**, raw, no context | Information run through the full cycle |
| Processing | None or minimal | Evaluated, corroborated, analysed, contextualised |
| Reliability | Unassessed | Source graded + credibility graded (§3) |
| Value | A pile of findings | A decision-ready assessment |

> **The gate:** never present OSINF as OSINT. If it has not been evaluated for source reliability,
> corroborated, and analysed, it is not intelligence yet - it is a lead. The added value of OSINT is
> the *analysis*, not the *collection*.

---

## 2. The intrusiveness ladder (passive → semi-passive → active)

Collection intrusiveness is a spectrum, not a binary. It is the cleanest legal/OPSEC compass in the
discipline: **the more active you get, the closer you move to activity that anti-intrusion law may
capture, and the more detectable you become to the target.** Always start passive and only climb the
ladder when a lower rung cannot answer the EEI *and* authorization plus scope permit it.

| Rung | Definition | What it entails (examples) | Detectable by target? | Legal/OPSEC risk |
|------|-----------|----------------------------|-----------------------|------------------|
| **Passive** | **No interaction with the target's systems.** Only third-party, cached, or already-collected data. | Search engines, Google dorks, cached pages, Wayback Machine, Certificate Transparency logs (`crt.sh`), passive DNS, WHOIS history, breach-data lookups, `shodan`/`censys` queries (their scan, not yours), public social profiles viewed logged-out, satellite/Street View imagery, public records/registries | No - you never touch the target | Lowest. Baseline default. |
| **Semi-passive** | **Light, normal-looking interaction** indistinguishable from an ordinary visitor; nothing intrusive, no probing beyond public-facing surfaces. | Loading the target's own website/DNS at normal volume, resolving its public DNS records, reading its published documents, viewing a public profile while logged in | Possibly, but blends into legitimate traffic | Low-moderate. Watch ToS and rate. |
| **Active** | **Direct, deliberate interaction with target systems** - sending packets/queries designed to elicit information. | DNS brute-force / subdomain forcing, port/service/OS scanning (Nmap `-sS -sV -O`), directory/endpoint enumeration, crawling, banner grabbing, credential-stuffing tests | Yes - appears in the target's logs/IDS | Highest. **Requires explicit authorization and scope.** Can constitute an offense without it. |

### 2.1 Why passive-first

- **Legality**: passive collection of openly available information is broadly lawful; active
  enumeration against systems you do not own or are not authorized to test can breach anti-intrusion
  statutes (CFAA, Loi Godfrain, Budapest Convention). See `legal-ethics-opsec.md`.
- **Detectability / OPSEC**: passive work leaves no trace on the target; active work lands in their
  logs and can tip off the subject or a Blue Team (in Red Team/TLPT engagements this matters
  operationally).
- **Sequencing**: passive findings *shape* any later active step - you scan far less, far more
  precisely, once passive recon has mapped the surface.

### 2.2 Mapping to collection disciplines

OSINT is one discipline among several; know where a finding actually comes from, because it sets
authorization and reliability expectations.

| Discipline | Source | Typical rung | Note |
|-----------|--------|-------------|------|
| **OSINT** | Public web, social, records, imagery | Passive → semi-passive | The skill's core; active only when authorized |
| **SOCMINT** | Social-media platforms | Passive → semi-passive | Subset of OSINT; ToS-sensitive |
| **GEOINT** | Satellite/aerial/maps imagery | Passive | Geolocation, chronolocation |
| **FININT** | Corporate filings, registries, transactions, crypto ledgers | Passive | SEC EDGAR, OpenCorporates, blockchain explorers |
| **SIGINT** | RF, packet captures | Active | Out of OSINT scope unless on owned infra |
| **HUMINT** | Interviews, elicitation, pretexting | Active + interactive | **Out of OSINT scope**; pretexting/impersonation is a red line (see legal-ethics-opsec.md) |

> Active enumeration (Nmap, DNS brute force, crawling) and any HUMINT/social-engineering step is
> **beyond passive OSINT**. It belongs to an authorized pentest/Red Team engagement, gated by a
> signed ROE - never to speculative or unauthorized research.

---

## 3. Source evaluation & reliability

Two things must be judged for every finding, and they are **independent axes**: (1) how much do we
trust the *source*, and (2) how credible is the *specific piece of information*. A completely reliable
source can still relay something doubtful; an unreliable source can occasionally carry something that
turns out confirmed. Decoupling the two is the whole point of the Admiralty code.

### 3.1 The Admiralty / NATO code (the default instrument)

A **two-character rating**: a letter for source reliability + a number for information credibility
(e.g. **A1**, **C4**, **F6**). Origin: NATO / UK Joint Doctrine Publication 2-00. Use it on every
material fact.

**Source reliability (letters A-F)** - from technical review of the source's capabilities or, for a
human/reporting source, its track record:

| Code | Reliability | Meaning |
|------|-------------|---------|
| **A** | Completely reliable | No doubt on authenticity/trustworthiness/competence; consistently reliable |
| **B** | Usually reliable | Minor doubt; valid information most of the time |
| **C** | Fairly reliable | Doubt exists, but has provided valid information in the past |
| **D** | Not usually reliable | Significant doubt, though has provided valid info before |
| **E** | Unreliable | Lacks authenticity/trustworthiness/competence; history of invalid info |
| **F** | Reliability cannot be judged | No basis to assess (e.g. brand-new/unknown source) |

**Information credibility (numbers 1-6)** - from the probability of the claim and its degree of
corroboration by other sources:

| Code | Credibility | Meaning |
|------|-------------|---------|
| **1** | Confirmed | Confirmed by **independent** sources; logically self-consistent; consistent with other info |
| **2** | Probably true | Not confirmed; logical; consistent with other info |
| **3** | Possibly true | Not confirmed; reasonably logical; agrees with some other info |
| **4** | Doubtful | Not confirmed; possible but not logical; no other info |
| **5** | Improbable | Not confirmed; illogical; contradicted by other info |
| **6** | Truth cannot be judged | No basis to assess |

**Reading a rating:** `A1` = totally reliable source + confirmed information (act on it). `B2` = solid.
`C3` = usable lead, corroborate further. `F6` = source and claim both unknown - a raw lead, not a
finding. `A5` (reliable source, improbable claim) or `E1` (bad source, confirmed elsewhere) are the
instructive edge cases that prove the axes are independent.

The grade is inherently subjective; its value is that it forces an explicit, auditable judgment and
signals to the reader how much weight to place on each fact. **Re-score as new information arrives** -
credibility especially moves as corroboration accumulates.

### 3.2 Weighted rubrics (when a defensible, auditable score is needed)

For high-stakes findings, back the letter/number with a weighted 1-5 scoring model (assign weights by
risk profile, score each criterion 1-5, sum weighted scores).

*Source-reliability criteria:* reputation & track record · credibility & expertise · data sources &
collection methods · collaboration with trusted peers · transparency (does it disclose its sources
/methods/update cadence) · timeliness & accuracy · depth of analysis & context · conformity with
standards · legal & ethical compliance.

*Content-credibility / corroboration criteria:* multi-source **independent** corroboration ·
consistency with known facts/tactics · technical detail & evidence · plausible attribution/motivation ·
timing & context fit · habitual reliability of the reporting · peer/trusted-group validation · red
flags & anomalies noted and explained.

### 3.3 Corroboration, provenance, primary vs secondary, recency

- **Corroboration / triangulation**: a claim rises to "confirmed" only when **multiple *independent*
  sources** support it. Independence is the trap - three outlets all citing the same original tweet
  are **one** source, not three. Trace every claim back to its origin before counting it.
- **Primary vs secondary**: prefer **primary** sources (the original document, filing, image, dataset,
  the person who witnessed it) over **secondary** (reporting *about* the primary). Every secondary
  source is a place where distortion can enter.
- **Provenance**: record where each item came from, when, and how - this is both an analytic input
  (a source's chain affects its grade) and an evidentiary requirement (§7.2).
- **Recency**: fresher generally scores higher, but *dated* is not *false* - check that a fact is still
  current for the question's time window, and that an "old" fact has not been silently superseded.

---

## 4. Verification & anti-disinformation

**"Public ≠ true."** Open sources are a vector *for* disinformation, not just a window onto truth -
fake fact-checks, staged imagery, recycled old media, and AI-generated fabrication all circulate
freely. Verification is the analyst's antidote and the core of the Bellingcat-style method.

### 4.1 The Bellingcat verification doctrine

1. **Transparency** in the research process - show your work.
2. **Cross-checking / corroboration** - triangulate multiple *independent* public sources before
   asserting anything.
3. **Link the evidence** directly in the output - reproducible, auditable sourcing so any reader can
   re-verify.

### 4.2 Verification techniques

| Technique | What it does | Notes / tools |
|-----------|-------------|---------------|
| **Cross-referencing** | Confirm a claim against independent sources | The foundation; count independent origins, not restatements |
| **Reverse image search** | Detect reused/older/mislabeled images | Google Images, **Yandex** (strong on faces), TinEye, Bing |
| **Geolocation** | Establish *where* a photo/video was taken | Landmarks, signage, road markings, foliage, architecture; confirm on Google Earth / Street View / Yandex-Baidu-Bing Maps |
| **Chronolocation** | Establish *when* | Shadow direction & length (SunCalc), weather records, foliage/snow, timestamps, visible events |
| **Metadata / EXIF checks** | Extract embedded GPS/time/device/author | ExifTool, Jeffrey's Exif Viewer - but note metadata is easily stripped or forged; treat presence as a lead, absence as neutral |
| **Video verification** | Detect edits/recycled footage; extract keyframes | InVID / WeVerify |
| **Image forensics** | Detect manipulation/compositing | FotoForensics (error-level analysis) - indicative, not conclusive |

### 4.3 Detecting fabrication & manipulation

- **Recycled/old media**: reverse-image search flags an image being reused out of its original context
  - one of the most common disinformation tactics.
- **Staged / composited images**: forensic tools plus internal-consistency checks (shadows,
  reflections, resolution seams).
- **Deepfakes & synthetic media**: treat unverifiable audio/video as **F6** until corroborated by
  independent evidence. Look for temporal/lip-sync artifacts, lighting/blink anomalies, and
  audio-visual mismatch - but do **not** rely on "it looks real"; require external corroboration
  (a second angle, a primary witness, a document). Voice can be synthesized: on synchronous channels,
  **verify identity out-of-band**, not by the sound of a voice.
- **Fabricated documents/accounts**: check creation date, activity history, network of connections,
  and whether the artifact is internally consistent and consistent with known facts.

> Verification failure modes to name explicitly: false corroboration (one source echoed many times),
> anchoring on a striking image, and accepting metadata at face value. When something cannot be
> verified, **say so** and grade it low - do not launder a rumor into a finding.

---

## 5. Analysis techniques

Analysis converts verified findings into meaning. Use structured techniques (SATs) rather than
intuition, and actively fight your own biases.

### 5.1 Link / entity analysis

Map entities (people, orgs, domains, IPs, accounts, wallets, phones) and the relationships between
them to expose structure, intermediaries, and hidden connections. Build the graph as you collect;
follow relationships outward. Tools: Maltego (seed entity → run transforms → expand graph → follow
relationships → export), plus general graph tooling. Watch for the key nodes - the hub that connects
otherwise-separate clusters is often the answer.

### 5.2 Timeline building

Order events chronologically (UTC) to reveal sequence, causation, gaps, and contradictions. A timeline
frequently breaks a fabricated narrative - an event that cannot fit the established sequence is a red
flag. Anchor each entry to a graded, sourced fact.

### 5.3 Pivoting logic

A **pivot** is using one confirmed identifier to discover the next. This is how OSINT investigations
actually progress. Investigations are organized by identifier type - **Username · Email · Domain · IP ·
Phone · Person · Company · Cryptocurrency** - and each is a launch point into the others.

Example pivot chain: `LinkedIn profile → username → same username on X → post reveals a location →
geolocation confirms a site → an associated account surfaces a second person`. Record every pivot in a
link chart so the reasoning is reproducible. Each new node re-enters collection and must itself be
graded and verified - pivots multiply both leads *and* error if you skip verification.

### 5.4 Analysis of Competing Hypotheses (ACH)

The core SAT for contested questions. It defends against the natural urge to seek confirming evidence.

1. **Enumerate all plausible hypotheses** up front (including the uncomfortable ones).
2. **List the evidence** and each item's diagnostic value.
3. For each piece of evidence, ask whether it is **consistent / inconsistent** with each hypothesis.
4. **Seek disconfirming evidence** - evidence that *rules a hypothesis out* is far more valuable than
   evidence that merely fits.
5. **Rank by fewest inconsistencies** - the surviving hypothesis is the *least disconfirmed*, not the
   most confirmed.
6. Note what evidence *would* change the conclusion (sensitivity), and re-run as new data arrives.

### 5.5 Cognitive-bias mitigation

Different analytic problems call for different approaches, and every analyst carries biases that must
be consciously overcome for objective analysis.

| Bias | What it does | Countermeasure |
|------|-------------|----------------|
| **Confirmation bias** | Over-weights evidence that fits the favored hypothesis | ACH; actively hunt disconfirming evidence; assign a devil's advocate |
| **Mirror-imaging** | Assumes the subject thinks/acts as you would | Model the subject's actual context, incentives, culture, constraints |
| **Anchoring** | First impression dominates later judgment | Generate hypotheses *before* deep-diving one; revisit initial framing |
| **Availability** | Recent/vivid items feel more probable | Weight by evidence and base rates, not salience |
| **Base-rate neglect** | Ignores prior probabilities | State the prior explicitly before updating on new evidence |

### 5.6 The "so what" / assessment step

Every product must answer **"so what?"** - the implication for the requester's decision, not just the
raw findings. State the assessment, the confidence in it (§6), and what should be done or watched next.
An OSINT product that stops at "here is what we found" is OSINF, not OSINT (§1.6).

---

## 6. Confidence & estimative language

Keep **two things separate** and never let one stand in for the other:

- **Source reliability / information credibility** (§3) = how good is the *input* (Admiralty grade).
- **Analytic confidence** = how much trust we place in *our own judgment*, given evidence quantity,
  quality, source diversity, corroboration, and the strength of the reasoning.

A high-confidence assessment can rest on partly uncertain sources if they converge; a single A1 fact
does not by itself make an *assessment* high-confidence. Say which you mean.

### 6.1 Probability yardstick (estimative language)

Use a consistent likelihood vocabulary and, where useful, attach a rough percentage band. Do not mix
scales, and avoid vague words like "could" that carry no probability.

| Term | Approx. probability band |
|------|--------------------------|
| Almost certain / highly likely | ~90-99% |
| Likely / probable | ~65-85% |
| Roughly even chance | ~45-55% |
| Unlikely / improbable | ~15-35% |
| Very unlikely / remote | ~1-10% |

(Bands are illustrative; adopt one scale for the whole product and label it.)

### 6.2 Analytic confidence levels

| Level | When to use |
|-------|-------------|
| **High** | Multiple independent, reliable sources corroborate; strong reasoning; little contrary evidence |
| **Moderate** | Credible but incomplete/uncorroborated sourcing, or some contrary evidence |
| **Low** | Fragmentary, single-source, or heavily caveated; plausible but unverified |

Write assessments in the form: *"[Estimative term] that X (analytic confidence: [level]),"* e.g.
*"It is **likely** the account is operated from Region Y (**moderate** confidence)."*

---

## 7. Reporting

### 7.1 OSINT report template

A finished product tailored to its audience. Every material fact carries a **grade + source +
date**; every conclusion carries **confidence**.

```
OSINT REPORT - <title>
Reference: <id>            Date (UTC): <YYYY-MM-DD>
Classification/Handling: <TLP marking - see legal-ethics-opsec.md>
Prepared for: <requester>  Prepared by: <analyst/persona>

1. REQUIREMENT
   - PIR / KIQ answered; scope; authorization reference; time window covered.

2. BLUF / EXECUTIVE SUMMARY
   - Bottom Line Up Front: the answer in 2-4 sentences, with the overall
     assessment and its analytic confidence. Written for the decision-maker.

3. FINDINGS
   - Each finding as a discrete, verifiable statement, with:
       • the fact
       • Admiralty grade (e.g. B2)
       • source (primary where possible) + URL/reference
       • collection date (UTC)
       • verification method(s) applied
   - Group by EEI or by entity. Distinguish fact from inference.

4. ANALYSIS & ASSESSMENT
   - Link/timeline/ACH results; the "so what."
   - Assessment(s) in estimative language + analytic confidence (§6).
   - Alternative hypotheses considered and why rejected.

5. INTELLIGENCE GAPS
   - What could NOT be established, and why (unverifiable, out of scope,
     no authorization). Explicit gaps prevent false certainty.

6. RECOMMENDATIONS
   - Actionable next steps / mitigations, prioritized. Tie each to a finding.

7. APPENDICES
   - Chain-of-evidence manifest (§7.2), link charts, timelines,
     full source list, methodology notes.
```

**Report discipline:** BLUF first (busy readers stop after the summary). Never present an ungraded fact
or an unsourced conclusion. State gaps as prominently as findings. Match depth and language to the
audience - strategic (decision-makers), operational (campaign/response teams), tactical (front-line
analysts).

### 7.2 Chain-of-evidence note

Findings are only defensible if they can be re-verified and have not been tampered with. Preserve
evidence at collection time (see `legal-ethics-opsec.md` for full handling, retention, and TLP rules):

- **Capture**: full-page screenshot + a self-contained archive (single-file HTML) + submit to a
  third-party archive (Wayback / Archive.today) as an independent witness.
- **Hash**: `sha256sum` every captured artifact into a manifest at the moment of collection.
- **Timestamp**: everything in **UTC**.
- **Log**: an evidence log with columns `UTC timestamp | type | description | SHA256 | source/tool`.
- **Preserve**: keep the original untouched + a working copy in separate locations; re-check integrity
  periodically; reports reference the **hash manifest**, not raw attachments.

---

## 8. End-to-end workflow checklist

A compact run-sheet for a single engagement. Do not skip the gate steps.

- [ ] **1. Define the question** - write the PIR/KIQ and its EEIs; set scope + time window.
- [ ] **2. Authorization check** - confirm legal standing / written authorization (ROE) **before any
      collection**; set the OPSEC level and intrusiveness ceiling. → `legal-ethics-opsec.md`.
- [ ] **3. Plan collection** - map each EEI to candidate sources, ranked by value and intrusiveness.
- [ ] **4. Passive collection** - start at the lowest rung; capture provenance (URL, UTC, hash,
      screenshot/archive) as you go. → `tools-catalog.md`.
- [ ] **5. Pivot** - use confirmed identifiers to reach the next (username → email → domain → …);
      log every pivot in a link chart; climb the intrusiveness ladder only if authorized and needed.
- [ ] **6. Verify** - cross-reference *independent* sources; reverse-image / geolocate / chronolocate /
      check metadata; flag fabrication; remember public ≠ true.
- [ ] **7. Grade** - Admiralty A-F / 1-6 on every material fact; re-score as corroboration changes.
- [ ] **8. Process & collate** - dedupe, normalize, enrich.
- [ ] **9. Analyze** - link/entity, timeline, ACH; mitigate bias; answer "so what"; assign estimative
      likelihood + analytic confidence.
- [ ] **10. Report** - requirement → BLUF → graded/sourced findings → assessment + confidence → gaps →
      recommendations; attach the chain-of-evidence manifest; apply the correct TLP marking.
- [ ] **11. Disseminate & feedback** - deliver securely to the right audience; capture feedback;
      re-task the cycle for remaining gaps.

> Golden threads across the whole workflow: **authorization before collection · passive before active ·
> corroborate before asserting · grade every fact · separate source-reliability from analytic
> confidence · state your gaps · public ≠ permitted and public ≠ true.**
