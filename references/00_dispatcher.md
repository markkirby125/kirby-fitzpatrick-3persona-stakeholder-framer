# 3Persona Stakeholder Framer — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [I’m an Editor, these are the 7 First-Draft Mistakes I Fix All the Time](https://www.youtube.com/watch?v=bBg7GyIfn0A)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: Audience-State Fan-Out and the Three-Reader Matrix

### 1.1 The first-draft mistake

Fitzpatrick's editorial thesis is that flat writing is not an expertise failure but a **direction failure**:

> *"You're writing the way you think, not the way readers read. When you ignore reader expectations, you give readers work instead of value."*

Every first draft is shaped by a single imagined reader: the one sitting inside the author's head, already holding the context, already convinced the work mattered, already aware of the failure mode that motivated it. That reader does not exist. What exists is a heterogeneous audience — and, more importantly, a **heterogeneous sequence of states inside each individual reader**.

Two of the lecture's seven first-draft mistakes are direct symptoms of this single-reader hallucination:

- **Burying the main point** — the draft narrates the author's discovery sequence, because the imagined reader is the author, who already knows the point and therefore does not need it early.
- **Not framing ideas as solutions to problems readers actually have** — the draft presents the change in the author's frame (what was built) instead of in each reader's frame (what it changes *for them*).

In prose, this costs attention. In engineering artifacts it costs **decisions**: a PR that is context-starved gets deferred, an RFC that is ROI-silent gets deprioritized, and a migration plan that is risk-opaque gets blocked in a review meeting three weeks later by someone who was never going to say yes on the evidence provided.

The 3-Persona Stakeholder Matrix is the counter-measure. It replaces the single imagined reader with the three default states an engineering audience occupies, and it forces the artifact to answer each state's governing question in that state's own unit of proof.

### 1.2 The three default states

The personas are **not job titles and not people**. They are the three default states any competent reader enters, defined by *what they lack* and *what they therefore require*:

| Persona | Default state | Governing question | Required deliverable | Evidence unit | Failure when omitted |
|---|---|---|---|---|---|
| **Uninformed** | Has the skill to evaluate, but not the local context (no prior thread, no topology, no vocabulary) | *"What is this, and can I even evaluate it?"* | **Context** — minimum viable shared ground | Diagram, path + line, data flow, glossary | Rewrites it wrong, defers, or asks a human (the doc did not transmit) |
| **Indifferent** | Can evaluate immediately, but has no stake and no reason to spend attention | *"Why should I care, and why now?"* | **ROI** — value in the reader's own unit | Minutes saved, incidents/quarter, engineer-days, p99 ms, dollars, a dated trigger | Silent non-review; the artifact dies in an unread queue |
| **Skeptical** | Has stake, has a competing interest or a scar from a prior incident, and is actively hunting for the disconfirming case | *"Where does this break, and what happens when it does?"* | **Risk mitigation** — pre-empted falsifiers, bounded scope, undo path | Blast radius, failure modes + detectors, rollback with measured time, named unknowns + owner + date | Blocks late, re-litigates settled ground, demands re-work, or approves socially and disengages |

The matrix is a **coverage device**: three questions, three deliverables, three units. An artifact is "framed" when all three questions are answerable *from the artifact itself*, in the correct order, without a human in the loop.

### 1.3 The states are sequential passes, not audience segments

The most common misuse of a stakeholder matrix is to read it as segmentation — "the PM is the Indifferent one, the security lead is the Skeptical one, so I'll write three sections for three people." That is wrong, and it produces the failure mode this skill exists to prevent.

The three states are the **stages of one reader's attention funnel**, and they are strictly ordered. A reader cannot judge whether a change is worth their attention (Indifferent) before they can evaluate it at all (Uninformed), and cannot judge whether it is safe (Skeptical) before they have decided it is worth judging. Anyone — designer, SRE, staff engineer, your own self in six months — walks the same funnel, at speed, in seconds:

```text
THE READER'S FUNNEL (any role, any artifact, ~20–40 s to reach a verdict)
─────────────────────────────────────────────────────────────────────────────
PASS 1  UNINFORMED     "Can I evaluate this?"
        ├─ yes ─────────────► PASS 2
        └─ no  ─────────────► defers · misreads · pings a human      [DEAD END]

PASS 2  INDIFFERENT    "Is this worth my attention, in my unit?"
        ├─ yes ─────────────► PASS 3
        └─ no  ─────────────► silent non-review                       [DEAD END]

PASS 3  SKEPTICAL      "Where does it break? What if I'm the one paged?"
        ├─ answer present ──► approve · dissent on evidence · merge
        └─ answer absent ───► block late · request re-work            [REWORK]
```

Two consequences follow, and both are load-bearing:

1. **Section order is forced.** Context → ROI → Risk is not a style preference; it is the funnel's own order. ROI placed before context forces the reader to evaluate value on a subject they cannot yet evaluate, which reads as marketing. Risk placed first reads as defensive apology and *manufactures* skepticism in a reader who arrived indifferent — the opposite of the intended effect.
2. **The personas are the same reader over time.** Serving them is not broadcasting to three people; it is making one reader's three consecutive passes each terminate in a decision rather than an exit.

### 1.4 Before vs. after

```text
❌ BEFORE — SINGLE-READER MONOLOGUE (author-shaped, one persona served: the author)
┌──────────────────────────────────────────────────────────────────────────────────┐
│ PR #1284 — "session-store: swap to Redis cluster, add token bucket"              │
│                                                                                  │
│ While debugging the checkout timeout I traced it to the session store and        │
│ after ruling out the LB I landed on a Redis-cluster-backed session store with    │
│ a token bucket. There is an env var SESSION_BACKEND. I also removed the         │
│ LegacySessionAdapter because nothing used it. Config lives in the usual place.   │
│ Should be fine — locally it's much snappier.                                     │
└──────────────────────────────────────────────────────────────────────────────────┘
UNINFORMED  ✗  "LegacySessionAdapter" — used by which callers? "the usual place"?
INDIFFERENT ✗  "snappier" — in which unit? costs what to run? why this sprint?
SKEPTICAL   ✗  "should be fine" — blast radius? drain? rollback? dual-write window?
VERDICT: 3 dead ends. Reviewer pings in DM → answer never lands in the artifact →
         the next reader pays it again. The PR becomes a meeting.
```

```text
✅ AFTER — THREE-STATE FRAME (three passes, one shared technical core)
┌──────────────────────────────────────────────────────────────────────────────────┐
│ CHANGE  Session state moves from per-pod in-memory maps to a shared Redis        │
│         cluster; 214 lines of session-adapter code deleted. Revert = 1 flag.     │
├──────────────────────────────────────────────────────────────────────────────────┤
│ [1] CONTEXT — for readers who have never seen this path                       │
│     Holds ⌈cart⌉ · existing path: BFF → LegacySessionAdapter → in-mem map      │
│     (session.ts:41) · 3 pods, sticky routing removed last quarter              │
│     Term: "token bucket" = per-user request limiter, 20 rps default            │
├──────────────────────────────────────────────────────────────────────────────────┤
│ [2] ROI — for readers deciding whether this deserves 20 minutes                 │
│     Prevents: 4 checkout timeouts this month (INC-4501, INC-4522, +2)           │
│     Saves: ~6 engineer-hrs/mo of session-related on-call investigation          │
│     Cost: 1 cluster (reuses existing `cache-prod`) · 2 engineer-days            │
│     Why now: sticky routing is already removed; in-mem maps are unsafe today    │
├──────────────────────────────────────────────────────────────────────────────────┤
│ [3] RISK — for readers asking where this breaks                                  │
│     Blast radius: checkout session path only; no schema, no wire format         │
│     Failure mode: Redis latency > 40 ms → p95 checkout +12 ms (measured,        │
│       bench/session_bench.py RUNS=5, baseline main@4f9c1ab)                     │
│     Detector: alert `session_store_error_rate > 0.5%` (added, fires to #sre)    │
│     Rollback: SESSION_BACKEND=memory, ≤ 60 s, sessions re-login once            │
│     Unknown: cost at 10× traffic — unmeasured. Owner: @author, before GA.       │
└──────────────────────────────────────────────────────────────────────────────────┘
VERDICT: pass 1 terminates in comprehension, pass 2 in attention, pass 3 in a
         decision on evidence. No DM required.
```

### 1.5 The pattern in one line, and its boundary

```text
ONE SHARED CORE  +  CONTEXT (can I evaluate?)  +  ROI (why now, in my unit?)
                 +  RISK (what breaks, what undoes it?)
```

The boundary matters as much as the pattern: **framing is not duplication**. The frame supplies three *entry points* and three *deliverables* around one canonical technical body. If context, ROI, and risk each restate the mechanism, the numbers drift, the doc doubles, and the artifact develops divergence debt — the same defect class this skill exists to remove. One number, one home; the personas get pointers and units, never copies.

Composition with the suite: the three-state frame is the **macro routing layer**. Within each block, [Cathedral Taxonomy](../../kirby-fitzpatrick-cathedral-taxonomy/SKILL.md) announces the structure, [Topic–Comment Elaboration](../../kirby-fitzpatrick-topic-comment-elaboration/SKILL.md) keeps each paragraph focused on the reader's concern, [Linear Relay Linking](../../kirby-fitzpatrick-linear-relay-linking/SKILL.md) and the [Gricean Bridge Connector](../../kirby-fitzpatrick-gricean-bridge-connector/SKILL.md) make the traversals explicit, and [Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md) keeps the headings self-supporting so the funnel survives a 15-second skim.

---

## 2. Core Transformation Protocols

1. **Enumerate the readership before drafting a single sentence.** Write down the actual humans who will touch this artifact (e.g. *mobile client dev, SRE on-call, analytics owner, platform reviewer, PM, security*). Then map each to the state they will *enter*, not to the persona you find most flattering to write for. Do not begin framing until the list exists — the frame is derived from the list, never from the author's imagination.

2. **Classify by default state, never by title.** A PM reading a schema migration that breaks their dashboard is **Skeptical**, not Indifferent. An SRE reading an unfamiliar service is **Uninformed** first. Titles predict nothing; the state is determined by *what the change does to the reader's world*. Re-derive the state per artifact, not per person.

3. **Lead with a persona-invariant Change Statement.** One or two sentences, present tense, identical for all three states: *what changes, on what surface, and what undoes it.* This is the lecture's "lead with your main point" fix applied to engineering artifacts. Test: the statement must be true and useful to a reader in any of the three states.

4. **Supply minimum viable shared ground — not history.** The context block is the smallest set of facts that lets an uninformed-but-competent reader evaluate the change: one diagram of the current path, the file + line anchors, the two or three domain terms the artifact will reuse, and the prior-state baseline. It is **not** a narrative of how the codebase got here. Test: a reader who has never opened this service can restate the change model after reading only block 1.

5. **Fix the vocabulary once, then never drift.** A single canonical term per concept, declared in block 1, used verbatim in blocks 2 and 3 and in the diff. Divergent terminology across a multidisciplinary audience is what makes a reader believe two systems are being described — the lecture's "inconsistent key terms" mistake, and the most common cause of a phantom objection in review.

6. **Price the change in the reader's unit, never the author's.** "Cleaner", "more idiomatic", "reduces tech debt" are author units. Convert to a reader unit before writing: minutes per review, on-call hours per month, incidents per quarter, engineer-days, CI minutes, p99 ms, dollars, or a dated trigger (`compliance date 2026-11-01`). If a change genuinely has no reader-unit value, say so explicitly rather than inflating it — `ROI: none beyond prerequisite for X` is honest framing, and it protects the artifact's credibility.

7. **Name the trigger date or the compounding rate in the ROI block.** "Why now" is a distinct question from "is it worth it". Indifference is usually not disagreement — it is *deferral*, and deferral is defeated only by a trajectory (`+7 incidents/quarter`), a deadline, or a cost that grows while nothing is done.

8. **Pre-empt the three most probable falsifiers in the risk block.** Write the skeptic's three most likely objections as headers, then answer each with an artifact: blast radius, failure mode + detector, measured rollback time. Answering an objection the reader was about to raise is the single highest-leverage move in the entire frame — it converts the skeptic from an adversary into a co-signer.

9. **Label unknowns instead of hiding them.** Every unverified claim gets `UNKNOWN` + owner + target date. A skeptic who finds an unlabeled unknown concludes the whole document is unlabeled; a declared unknown is a *closed* objection and is routinely approved.

10. **Bound the scope and the frame.** State what is excluded (contracts not touched, traffic not migrated, teams not affected), and cap each persona block. Unbounded context becomes a history lesson; unbounded ROI becomes a pitch; unbounded risk becomes a 40-minute threat model that no one reads. Framing discipline is editorial discipline.

11. **Order delivery Context → ROI → Risk; order assembly Core → Risk → ROI → Context.** The reader traverses the funnel forward; the author should build backward, because writing the risk block first is what reveals which context facts are actually load-bearing. Never ship a frame whose Risk block was written last — that is where the unexamined assumptions hide.

12. **Never write three documents.** If the frame has grown into three essays, the artifact has diverged into three sources of truth. Compress to one core plus three entry points; anything more belongs in a linked appendix, not in the frame.

13. **Keep the frame traversable.** Add one relay sentence at each block boundary stating what the reader now knows and what the next block answers, so a reader who enters at block 2 (a skim-referral, a Slack link with an anchor) can still reconstruct the path.

14. **Re-run the frame when the audience changes, not when the code changes.** A migration that adds a regulated data flow gains a new Skeptical state (compliance) that the original frame never served — the retired context fact must be pruned in the same edit that adds the new risk.

### 2.15 Persona triage table — state → unit → non-omittable

| State | Diagnostic signature in the reader's behaviour | Mandatory block content | The one thing you may never omit |
|---|---|---|---|
| **Uninformed** | Asks "which service?", "what's a token bucket?", re-reads the same sentence twice, opens the diff to answer a summary question | Current-state diagram · path/line anchors · glossary of reused terms · baseline ref | The **current state**, drawn on one screen |
| **Indifferent** | Skims headings, does not open the diff, defers with "let's look next sprint", never replies | Reader-unit value · magnitude · cost to run · trigger date or compounding rate | A **number in their unit** plus **why now** |
| **Skeptical** | Opens the diff first, greps for callers, asks about rollback and load, cites a prior incident | Blast radius · top-3 falsifiers answered with artifacts · detector/alert · rollback with measured time · unknowns with owners | The **undo path**, as one operation |

### 2.16 Transformation table: anti-patterns and clean replacements

| Anti-Pattern | State it loses | Reader's unanswerable question | Clean Replacement |
|---|---|---|---|
| "Migrates session handling to a shared store." | Uninformed | From what, to what, on which path? | "Session state moves from per-pod in-memory maps (`session.ts:41`) to the shared `cache-prod` Redis cluster; 3 pods, sticky routing already removed." |
| "Reduces tech debt and cleans up legacy code." | Indifferent | Worth my attention in which unit? | "Deletes `LegacySessionAdapter` (last caller removed in #1190); removes ~6 engineer-hrs/month of session-related on-call investigation." |
| "Should be fine — tested locally." | Skeptical | Fine per which check? What breaks at 10× load? | "14 unit cases + `bench/session_bench.py RUNS=5`: p95 checkout +12 ms. Unmeasured above 10× traffic — owner @author, before GA." |
| "See the design doc for background." | Uninformed | The design doc is gated, 40 pages, and predates this change | Two-sentence change statement + one current-state diagram inline; the doc is a citation, not a prerequisite. |
| "As discussed in the platform sync…" | All three | The reader was not in the room; the decision has no home | Restate the decision and its rationale inside the artifact. |
| "CC'ing the stakeholders for visibility." | Indifferent | Visibility is not a reason to spend attention | Address the state explicitly: the ROI block names the value and the decision you are asking for. |
| "This is a big change, so here is the full threat model." | Skeptical (by drowning them) | Which risk actually applies to me? | Three named falsifiers, each answered; link the full threat model as an appendix. |
| "Not a breaking change." | Uninformed + Skeptical | Breaking against which contract, verified how? | "No API signature, schema, config, or CLI change; `orders.v1` payload byte-identical on the 100k sample." |
| "The skeptic will object anyway, so I'll pre-apologize up front." | Indifferent (turns them Skeptical) | Why is this defensive before I even asked? | Risk content moves *after* ROI; blockers are stated as bounded facts, not concessions. |
| "Everyone knows the checkout path." | Uninformed (which is everyone new) | I do not, and now I cannot evaluate this | One diagram. The block costs 8 lines and removes 3 review round-trips. |

### 2.17 Failure diagnostics

| Symptom in review | Frame diagnosis | Fix |
|---|---|---|
| "What exactly is this changing?" | Change Statement missing; block 1 is a history | Write the persona-invariant change statement first |
| "Why are we doing this at all?" | ROI absent or expressed in author units | Add reader-unit magnitude + trigger date |
| "Can you walk me through the architecture?" | Uninformed block absent or gated behind a link | Inline current-state diagram + path anchors |
| "What happens if this falls over at peak?" | Skeptic not pre-empted; no load evidence | Add failure mode + measured boundary + detector |
| "How do we back this out?" | Undo path not stated | One-operation rollback with measured time |
| Two reviewers argue about terminology | Vocabulary drift across blocks and diff | Canonical term table in block 1, used verbatim everywhere |
| Doc is 6 pages for a 40-line diff | Persona bloat — three essays instead of one frame | Compress to one core + three entry points; move depth to appendices |
| Approved, then re-opened a month later | Approval was social; risk block asserted rather than evidenced | Bind each risk claim to an artifact and a falsifiable check |
| A new team joins and immediately blocks the migration | Audience changed; the frame was never re-run for their state | Re-derive states from the new reader list (protocol 14) |

**Related dispatchers.** Sequence the underlying argument moves (Common Ground → Destabilizing Problem → Terms of Settlement) with the [3-Part Proposal Engine](../../kirby-fitzpatrick-3part-proposal-engine/SKILL.md); audit the frame for zero-context reconstruction load with the [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md); make each traversal explicit with the [Gricean Bridge Connector](../../kirby-fitzpatrick-gricean-bridge-connector/SKILL.md); strip pitch-language from the ROI block with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md); convert vague verbs of state into measurable ones with the [Empty Verb Extractor](../../kirby-fitzpatrick-empty-verb-extractor/SKILL.md); make headings carry the frame under a 15-second skim with the [Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — Reformatting a Comment So It Survives All Three States

A review comment is an artifact with the same three readers: the author (often **Uninformed** about your reasoning — they cannot see your terminal), the watching team (**Indifferent** — they will either absorb your point or never learn it), and the defensive author (**Skeptical** — invested in the change, hunting for why you are wrong).

The rule: a comment must carry **context** (where this sits on the current path), **ROI** (why the change is worth the author's next 20 minutes), and **risk mitigation** (the falsifiable check that settles it). A verdict with no artifact is a state-3 failure dressed as authority.

**Before — collapses state 1 and 3:**

> This is wrong — we keep tenant scoping in the repo layer. Also this will be slow.

*Diagnosis:* no context (which repo layer, which call sites), no ROI (why does the author care now?), no falsifiable check — and two unsourced verdicts, which converts a fixable note into a status contest.

**After — three-state comment:**

```markdown
**Tenant scoping — blocking**

- CONTEXT  `src/orders/handler.ts:88` queries `orders` directly. Every other read in this
  service goes through `TenantRepo` (`src/repo/tenant.ts:12`), which applies the tenant filter.
- ROI      Scoping in two places means the next divergence drops it silently — this is the
  shape of INC-4412 (2026-02-11), which cost ~9 engineer-hrs to trace across three services.
  Fixing it now is a 4-line change; fixing it post-merge is a data-repair task.
- RISK     Check: `grep -rn "from orders" src/orders/` returns 1 hit outside the repo layer
  today, 0 after. Behavior change: none on the current fixtures (14 cases pass unchanged).
  If a caller depends on the unfiltered read, `orders_repo_test.ts:60` fails loudly — it does
  not currently.
- Minimal fix: route through `TenantRepo.findOrders()`. One commit, no behavior change.
```

**Second pattern — the ROI-calibrated non-blocking note.** Curiosity must be framed as a question with a stated cost, or the author cannot tell a blocker from a preference:

```markdown
**Possible extra round trip — non-blocking, unverified**
No measurement on my side, so I'm asking for the shape rather than asserting an outcome:
`make bench-orders` before/after, p95, median of 5 runs, same runner. My prior is < 5 ms
(`handler.ts:112`). If it is under that, ignore me — I would not spend a CI run on this.
```

### 3.2 PR Descriptions — A Multidisciplinary PR Across Three Teams

Take a realistic migration PR: `payments-service` moves its session and idempotency state from per-pod memory to a shared Redis cluster. It touches backend, mobile (login flow), analytics (session events), and SRE (new dependency, new alert). One artifact, six readers, three default states present in the room.

The frame is one shared core plus three entry blocks, in funnel order:

```markdown
## Change (persona-invariant)
Session and idempotency state move from per-pod in-memory maps to the shared `cache-prod`
Redis cluster. No API, schema, or wire-format change. Revert = one env var.

## [1] Context — read this if you have not worked in session handling
    before:  client → BFF → LegacySessionAdapter → in-mem map (session.ts:41)  [3 pods]
    after:   client → BFF → SessionStore          → cache-prod Redis cluster
    Terms:   "idempotency key" = per-request dedupe id, TTL 24 h (idem.ts:12)
             "token bucket"    = per-user rate limiter, 20 rps default
    Baseline: main @ 4f9c1ab

## [2] ROI — why this deserves your 20 minutes
    Mobile      login survives a pod restart; today it silently drops and users re-auth
                (reported 11× in the last 30 days, Play Console ANR-adjacent tickets)
    SRE         removes ~6 engineer-hrs/month of session-related on-call investigation;
                4 checkout timeouts in the last 30 days trace here (INC-4501, INC-4522)
    Analytics   session events stop double-counting on pod churn; ~0.4% event inflation today
    Cost        2 engineer-days · 1 namespace on the existing cluster (no new infra)
    Why now     sticky routing was removed last quarter — in-mem maps are already unsafe

## [3] Risk — where this breaks, and what undoes it
    Blast radius   session + idempotency paths only; payments ledger untouched
    Failure mode   Redis p99 > 40 ms → checkout p95 +12 ms (bench/session_bench.py RUNS=5,
                   median of 5, same runner, vs. main @ 4f9c1ab, spread ±3 ms)
    Detector       alert `session_store_error_rate > 0.5%` → #sre (added in this PR)
    Rollback       SESSION_BACKEND=memory, ≤ 60 s; users re-login once; no data loss
                   (idempotency keys re-derived from request ids)
    Unknown        behavior above 10× current traffic — unmeasured.
                   Owner @author, gate: before GA, not before merge.
```

**Why the frame holds across disciplines.** The mobile dev enters at pass 1 (they have not read this service). The analytics owner enters at pass 2 (they can evaluate instantly, but their dashboard is not on fire — the 0.4% inflation number is the only thing that earns their attention). The platform reviewer enters at pass 3 (the Redis dependency and the alert are the only things that can stop the merge). All three terminate; none ping the author.

### 3.3 Architecture RFCs / ADRs — Cross-Team Platform Migration

The highest-stakes instance: an ADR for migrating four teams' workloads from a self-managed queue to a managed platform. Here the personas map to *institutional* states, and the frame must survive a reader who has veto power and no context.

**Step 1 — derive the states from the actual reader list (protocol 1–2).**

| Reader | Enters as | Their unit | The one thing their state must receive |
|---|---|---|---|
| Team A owner (migrating first) | Uninformed → Skeptical | migration engineer-days | current topology + per-team rollback |
| Team B owner (migrating last, sees no benefit) | Indifferent | on-call pages/quarter | the date their own pain starts |
| Platform team (runs the new system) | Skeptical | pages + support load | blast radius at fleet scale, detector, runbook |
| Security / compliance | Skeptical (competing interest) | control coverage | which controls survive the move |
| Director / PM (funds it) | Indifferent | engineer-weeks, dollars, risk of stop-work | cost of the migration *and* cost of inaction, both dated |
| New hire in month 3 | Uninformed | comprehension | the diagram and the glossary |

**Step 2 — assemble the frame (build Risk first, ship Context first).**

```markdown
## Change
Load-generating workloads move from self-managed queue `q-legacy` to managed platform
`streams.io` over 9 weeks, team by team, oldest-pain-first. No message format change.
Revert per team = one routing flag (`ROUTER=legacy|streams`), ≤ 5 min, no message loss.

## [1] Context
    current:  4 teams → q-legacy (3 brokers, 2 AZs, self-run) → consumers
    target:   4 teams → streams.io (managed) → same consumers, same wire format
    Terms:    "cutover" = a team's producer routes to the new topic; consumer unchanged
    Diagram:  docs/adr/0042-topology.md §1  (inlined below, 1 screen)

## [2] ROI
    Capacity    q-legacy saturates at 9k msg/s; peak was 7.8k in March (+41% YoY).
                Saturation = dropped messages, no backpressure (INC-4301)
    On-call     ~14 engineer-hrs/quarter across 4 teams on broker ops (rotation data)
    Cost        migration 38 engineer-days total; run cost +$1.2k/mo vs. -$0 self-hosted
                (offset: ~9 engineer-days/quarter of ops → ~36/yr)
    Why now     capacity headroom runs out in Q3 at current growth; after that the
                migration becomes an incident-driven, unscheduled cutover

## [3] Risk
    Falsifier 1 — "cutover drops messages"
      Dual-write producers for 7 days per team; replay verified by row-count diff
      (scripts/topic_diff.py → 0 losses on the 40M-message soak)
    Falsifier 2 — "we lose control X"
      Control map: TLS in transit ✓, at-rest encryption ✓ (platform-managed keys),
      audit log ✓ (export to our SIEM, 90-day retention — verified in POC),
      egress controls ✗ → compensating control: per-topic allowlist (security sign-off 04-11)
    Falsifier 3 — "a bad cutover takes everyone down"
      Blast radius: one team's topics at a time; router flag per team;
      measured rollback 4m12s (staging drill, runbook step 7)
    Unknown    streams.io behavior at 3× current peak — load test scheduled 05-02,
               owner @platform, result lands in §3 before the second cutover
```

**Step 3 — verify traversal.** A Team B owner reading only §1, the §2 "why now" line, and §3's three falsifiers can decide whether to argue — without reading the diff, the POC notes, or the runbook. That is the acceptance condition for the artifact, and it is exactly what a heterogeneous audience demands: not three documents, one frame with three working entrances.

---

## 4. Verification Checklist

- [ ] **Three-state coverage test passes.** Context, ROI, and risk are all answerable *from the artifact alone*; each persona's block names its own deliverable, and each block's content is in that state's unit (diagram/anchors; reader-unit number; artifact-bound risk). No state is served only by a link to another document.
- [ ] **Funnel order is preserved.** The Blocks appear Context → ROI → Risk, the first block is the persona-invariant Change Statement, and no risk or defensive content precedes the ROI block (pre-apologia is a defect, not caution).
- [ ] **Reader-unit ROI test passes.** Every value claim is expressed in a unit the reader owns (minutes, on-call hrs/quarter, engineer-days, p99 ms, dollars, incidents), includes magnitude and cost, and carries either a trigger date or a compounding rate; zero author-unit justifications (`cleaner`, `more idiomatic`, `reduces tech debt`) survive in the final text.
- [ ] **Skeptic pre-emption test passes.** The three most probable falsifiers are named as headers and each is answered with an artifact (blast radius, failure mode + detector, one-operation rollback with measured time); every unverified claim is labeled `UNKNOWN` with an owner and a date.
- [ ] **Single-source integrity test passes.** One canonical term per concept, declared once and used verbatim across all three blocks and the diff; no number or claim appears in two blocks; the frame is one core plus three entry points, not three duplicated documents.
- [ ] **Traversal test passes.** A reader entering at any block (anchor link, skim referral, forwarded highlight) can reconstruct the funnel, and a 15-second heading-only skim still yields the same conclusion as a full read.