# Design Spec — MVP_SYNTHESIZER (Project Initiator V1.1)
# Created: 2026-06-04 | Author: Shubham Upadhyay (fiftyfive-tech)
# Status: Approved for implementation

---

## Context

V1.0 delivered DISCOVERY, which reads rough client docs and produces `discovery.md`. V1.1 delivers the next skill in the chain: **MVP_SYNTHESIZER**, which reads `discovery.md` and produces `mvp-scope.md`.

Two driving problems:
1. After DISCOVERY, there is no structured step to convert extracted requirements into a scoped MVP — consultants do this informally and inconsistently.
2. `mvp-scope.md` needs to be thorough enough to source both client-facing documents (proposals, SOWs) and internal documents (sprint plans, developer handoff) without additional consultant effort.

---

## V1.1 Scope

**Ships:** MVP_SYNTHESIZER skill + Readiness Gate pattern + 2 test fixtures (PASS-path + BLOCK-path)

**Does not ship:** ARCH_PROPOSER (V1.3), orchestrator (V1.2), SESSION_STATE.md, per-engagement PARKING_LOT.md

**Chain position:** DISCOVERY → **MVP_SYNTHESIZER** → (ARCH_PROPOSER) → (BACKLOG_GENERATOR)

---

## Design Decisions

### 1. Readiness Gate — Option A (gate embedded in each downstream skill)

Each downstream skill opens with a dedicated Readiness Gate section that validates the upstream artifact. No separate skill. No orchestrator dependency.

Rationale: gate logic is small and skill-specific. Duplication is acceptable. Reuse only justified when 3+ downstream skills exist (V1.3+).

### 2. Gate asks inline for BLOCKs, does not redirect

When discovery.md has gaps, MVP_SYNTHESIZER asks inline to fill them. Consultant is never told to "go back to DISCOVERY." Reduces friction and keeps the session flowing.

### 3. mvp-scope.md designed for dual audience

Output format designed so:
- **Client-facing use:** Scope: In table (Feature + Rationale) → SOW deliverables; Scope: Out → phase plan; Success Metrics → acceptance criteria
- **Internal use:** Scope: In → sprint backlog seed; User Journeys → QA test scenarios; Open Questions → ARCH_PROPOSER briefing

### 4. Effort signals deferred to ARCH_PROPOSER

S/M/L/XL sizing without architecture context produces noise. Deferred to ARCH_PROPOSER which has tech stack and build order context.

### 5. Framing selection is mandatory

Consultant must pick one of 3 framings (Time-boxed / Risk-first / Value-first) before prioritization begins. The skill cannot produce a sensible prioritization without knowing what lens to apply.

---

## Readiness Gate Specification

**Required fields from discovery.md:**

| Field | PASS | WARN | BLOCK |
|---|---|---|---|
| Core Problem (## Core Problem) | Present, HIGH or MED | Present, LOW | Missing |
| Primary Users (## Users) | Present | — | Missing |
| Features (≥2 in ## Features Mentioned) | ≥2 items | 1 item | 0 items |
| Timeline (## Constraints → Timeline:) | Explicit date/window | "not specified"/blank | Line missing |
| No unresolved CONFLICTs (## Confidence Notes) | All CONFLICT lines also contain "resolved" | — | Any CONFLICT line lacks "resolved" |

**CONFLICT detection:** Case-insensitive scan of `## Confidence Notes`. Any line with "CONFLICT" that lacks "resolved" → BLOCK.

**WARN behavior:** WARNs do not block. Flagged in `mvp-scope.md ## Confidence Notes`. Timeline WARN + time-boxed framing = skill asks for date before prioritization.

**BLOCK behavior:** Each BLOCK → one inline question. One at a time. Consultant may defer → becomes WARN + Open Question. Gate proceeds only after all BLOCKs resolved or deferred.

---

## Framing Options

| Framing | When to use | Scope driver |
|---|---|---|
| Time-boxed | Fixed, non-negotiable deadline | What fits before the deadline? |
| Risk-first | PMF uncertain, tech unproven | What validates the riskiest assumption? |
| Value-first | Deadline flexible, stakeholder buy-in critical | What delivers most value to primary user? |

---

## Feature Prioritization Logic

**Silent draft first.** Apply framing lens + confidence modifiers + dependency rules. Then present:
- HIGH-confidence obviously-in features → batch confirm
- Non-obvious features (MED/LOW, ambiguous fit) → one at a time
- Dependency rule: Feature B cannot be IN if Feature A (which B requires) is OUT
- "Defer all remaining" accepted at any point

---

## mvp-scope.md Format

All sections required. Omitting a section is only acceptable if consultant explicitly stopped mid-session (mark as `[INCOMPLETE — consultant stopped before this section]`).

| Section | Primary client use | Primary internal use |
|---|---|---|
| Problem Restatement | Proposal intro | Context briefing |
| MVP Framing | Scope rationale for client | Explains prioritization to dev team |
| Scope: In | SOW deliverables list | Sprint backlog seed |
| Scope: Out | Phase plan / deferred commitments | Phase 2 planning |
| Key User Journeys | Acceptance scenarios | QA test scenarios |
| Success Metrics | Acceptance criteria | Definition of done |
| Constraints | Client agreement anchors | Non-negotiables for ARCH_PROPOSER |
| Assumptions | Risk register seed | Dev team awareness |
| Open Questions | Client follow-up items | ARCH_PROPOSER briefing |
| Effort Signals | (deferred) | (deferred to ARCH_PROPOSER) |
| Confidence Notes | Transparency on uncertainty | Quality signal for reviewers |
| Source Artifacts | Traceability | Document chain |

---

## "Stop" Handling

If consultant stops mid-skill: proceed to draft immediately. Fill skipped sections with `[INCOMPLETE — consultant stopped before this section]`. Save gate still applies.

---

## Rules (inherited and new)

1. AI advises, human decides — no automatic scope choices
2. Never choose architecture, tech stack, or build order
3. Readiness Gate runs on every invocation — never skipped
4. Effort signals deferred to ARCH_PROPOSER — not produced here
5. Confidence levels: HIGH, MED, LOW only
6. LAYER_0_GLOBAL Rule 4 (output limits) and Rule 5 (no narration) apply

---

## Test Fixtures

| Fixture | Path | Purpose |
|---|---|---|
| synthetic-01 (SupplySync) | tests/fixtures/mvp-synthesizer/synthetic-01/ | PASS-path — clean discovery.md, gate passes, full happy path |
| synthetic-02-incomplete (RetailEdge) | tests/fixtures/mvp-synthesizer/synthetic-02-incomplete/ | BLOCK-path — 3 BLOCKs: missing Core Problem, missing Timeline, unresolved CONFLICT |

---

## Verification Checklist

- [ ] Readiness Gate runs on every invocation
- [ ] synthetic-01: all gate fields PASS, framing presented, prioritization runs, mvp-scope.md saved
- [ ] synthetic-02-incomplete: 3 BLOCKs detected, inline questions asked one at a time, gate clears after resolution
- [ ] Time-boxed + Timeline WARN → date asked before prioritization
- [ ] "Stop" mid-skill → partial draft with [INCOMPLETE] markers, save gate applies
- [ ] No effort numbers in mvp-scope.md (deferred notice only)
- [ ] Scope: In table sufficient to draft SOW deliverables without additional input
- [ ] Open Questions sufficient to brief ARCH_PROPOSER without re-running DISCOVERY
- [ ] No architecture decisions made anywhere in skill

---

## Open Risks

1. **Framing unfamiliar to consultants** — mitigated by "Best when:" hints per option
2. **Long prioritization loops** — mitigated by batching HIGH-confidence obvious-in features
3. **mvp-scope.md drift from discovery.md** — Problem Restatement must be rewritten, not copy-pasted; skill instruction enforces this

---

## Deferred (explicit non-decisions)

- ARCH_PROPOSER (V1.3): designed after MVP_SYNTHESIZER is dogfooded
- Effort signals: deferred to ARCH_PROPOSER — requires tech stack context
- SESSION_STATE.md: deferred until orchestrator (V1.2) exists
- parse_docx MCP tool: still deferred
