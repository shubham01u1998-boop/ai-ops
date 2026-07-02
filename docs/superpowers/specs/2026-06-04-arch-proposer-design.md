# Design Spec — ARCH_PROPOSER (Project Initiator V1.3)
# Created: 2026-06-04 | Author: Shubham Upadhyay (fiftyfive-tech)
# Status: Approved for implementation

---

## Context

V1.1 delivered MVP_SYNTHESIZER, which reads raw engagement docs and produces `mvp-scope.md`.
V1.3 delivers the next skill: **ARCH_PROPOSER**, which reads `mvp-scope.md` and produces `arch.md`.

Two driving problems:
1. After MVP_SYNTHESIZER, there is no structured step to translate a scoped MVP into a
   buildable architecture — consultants do this informally and inconsistently.
2. `arch.md` needs to be thorough enough to source both developer handoff (component design,
   sprint assignments) and BACKLOG_GENERATOR ticket "Tech Context" fields, without additional
   consultant effort.

---

## V1.3 Scope

**Ships:** ARCH_PROPOSER skill + 2 test fixtures (PASS-path + BLOCK-path)

**Does not ship:** BACKLOG_GENERATOR (V1.4), orchestrator (V1.2), SESSION_STATE.md

**Chain position:** DISCOVERY → MVP_SYNTHESIZER → **ARCH_PROPOSER** → BACKLOG_GENERATOR

---

## Design Decisions

### 1. Full architecture conversation (5 steps after gate)

Approach: Readiness Gate → Stack selection → Component design → Integration points →
Build order + sprint mapping → Effort signals → Client Summary.

Rationale: BACKLOG_GENERATOR needs component-level tech context to write developer-ready
tickets. A thin arch.md would force that skill to invent details it shouldn't.
"AI advises, human decides" — every major decision is consultant-confirmed.

### 2. Constrained stack menu — preferred list + constraint-aware filtering

ARCH_PROPOSER maintains a preferred stack list (fiftyfive-tech defaults) but filters and
augments it based on constraints in mvp-scope.md (tech, budget, timeline, integrations).
Presents 2–3 options per layer; consultant picks one per layer.

### 3. STRAWMAN markers — five conditions

Any decision meeting one or more of these conditions gets a `[STRAWMAN]` marker.
Signals: challenge this before dev sprint 1.

STRAWMAN auto-applies when:
- Decision was a close call between two viable options
- Confidence on the source data was MED or LOW in mvp-scope.md
- Decision is tentative and not yet confirmed by the consultant
- Effort estimate is XL
- Integration risk is HIGH

### 4. Build order in two layers

Technical dependency order (which components must be built before others) then sprint
mapping (which sprint each component lands in). Team composition (size + seniority) is
asked before sprint mapping — sprint capacity depends on it.

### 5. Client Summary is additive, not a replacement

Client Summary at top of arch.md is generated last, after all technical sections are
complete. It plain-language distills the technical content. Technical sections stay intact.
Rule: if a detail appears in the summary but not in the technical sections, the technical
sections are incomplete.

### 6. Effort signals produced here (deferred from MVP_SYNTHESIZER)

S/M/L/XL per feature, informed by component count + integration risk + team seniority.
Proposed by ARCH_PROPOSER, confirmed by consultant. XL estimates and HIGH-risk components
auto-flagged as STRAWMAN.

---

## Readiness Gate Specification

Runs on every invocation before any conversation. Never skipped.

| Field | Where to look in mvp-scope.md | PASS | WARN | BLOCK |
|---|---|---|---|---|
| Scope: In | `## Scope: In` table | ≥1 feature present | — | Table empty or missing |
| Primary User | `## Users` → `**Primary:**` | Present | — | Missing |
| Tech Constraints | `## Constraints` → `Tech:` line | Present (even partial) | — | Line missing entirely |
| Timeline | `## Constraints` → `Timeline:` | Explicit date or window | "not specified" / blank | Line missing entirely |
| Budget | `## Constraints` → `Budget:` | Explicit amount | "not specified" / blank | Line missing entirely |
| No unresolved CONFLICTs | `## Confidence Notes` | No CONFLICT lines lacking "resolved" | — | Any CONFLICT line lacks "resolved" |

**BLOCK behavior:** One inline question per BLOCK, one at a time. Consultant can defer →
WARN + Open Question. Gate clears when all BLOCKs resolved or deferred.

**WARN behavior:** Does not block. Flagged in `arch.md ## Confidence Notes`.
Timeline WARN without a date blocks sprint mapping — skill asks for date before that section.

---

## Stack Selection

### Step 1 — Show confirmed tech
Pull confirmed tech from `## Constraints → Tech:` in mvp-scope.md. Display as locked.

### Step 2 — Present options for unconfirmed layers
For each unconfirmed layer, present 2–3 options from the preferred stack list, filtered
by constraints. One layer at a time. Consultant can pick A/B/C or propose their own
option — if they propose one, ARCH_PROPOSER confirms it and applies a STRAWMAN marker
if it falls outside the preferred list. Format:

```
Backend:
A) Node.js + Express — lightweight, same language as React frontend, large Azure ecosystem
B) .NET Core — strong Azure-native, better for Oracle Retail POS integration, heavier
C) FastAPI (Python) — fast to prototype, good for data-heavy dashboards

Recommendation: A — fits budget and timeline; .NET only justified if Oracle POS SDK is .NET-specific.
Which? (A / B / C)
```

### Preferred stack list (fiftyfive-tech defaults)

| Layer | Options |
|---|---|
| Frontend | React, Vue.js, Next.js |
| Backend | Node.js + Express, .NET Core, FastAPI |
| Database | PostgreSQL, Azure SQL, Cosmos DB |
| Infra | Azure App Service, Azure Container Apps, AKS |
| Mobile (if applicable) | React Native, Flutter |

### Step 3 — STRAWMAN markers
Close-call decisions, tentative choices, or decisions derived from MED/LOW confidence
sources → flagged `[STRAWMAN]` in arch.md. Consultant can challenge before dev starts.

---

## Component Design

For each feature in `## Scope: In`, ARCH_PROPOSER silently derives components from the
confirmed stack, then presents as a single block for confirmation.

**Shared components** identified inline — flagged as shared dependencies in arch.md.

**Data model hints included** — table names and key fields per component. Not a full schema;
enough for BACKLOG_GENERATOR's "Tech Context" field.

One confirmation prompt. Consultant can add, rename, split, or remove components.
ARCH_PROPOSER applies changes and re-shows affected section.

---

## Integration Points

Built silently from `## Constraints → Other:` and the component map.
Presented as a single block with risk ratings.

Each integration point includes:
- System name
- Proposed integration approach
- Authentication / access method (if known)
- Risk level: HIGH / MED / LOW
- STRAWMAN marker if approach is unconfirmed
- Open Questions for unknowns

Unresolved integration Open Questions → carried into `arch.md ## Open Questions`.
These will flag as blockers in BACKLOG_GENERATOR for affected tickets.

---

## Build Order + Sprint Mapping

### Team composition input
Before sprint mapping, ARCH_PROPOSER asks once:
```
What's the team for this engagement?
(e.g. "2 frontend: 5y + 2y, 2 backend: 6y + 3y, 1 QA: 5y")
```
If not provided, sprint mapping proceeds with [STRAWMAN] assumption of 1 BE + 1 FE.

### Layer 1 — Technical dependency order
Derived from shared components and integration risks. Ordered list with rationale per step.

### Layer 2 — Sprint mapping
Derived from timeline (from mvp-scope.md) + team size + seniority.
Sprint length default: 2 weeks. Adjustable if consultant specifies otherwise.
Senior devs own complex/integration components. Junior devs on simpler pieces with review buffer.
Presented as a table: Sprint | Work | Owner.

One confirmation block. Consultant can shift components between sprints or adjust sprint length.

---

## Effort Signals

S/M/L/XL per feature, derived from component count + integration risk + team seniority.
Proposed by ARCH_PROPOSER, confirmed by consultant.

| Size | Meaning |
|---|---|
| S | 1–3 days, 1 developer, no integration risk |
| M | 1–2 weeks, 1 developer, low integration risk |
| L | 2–4 weeks, 1–2 developers, or medium integration risk |
| XL | 4+ weeks, multiple developers, or high integration/unknowns |

**Auto-STRAWMAN rule:** XL estimates and HIGH-risk components automatically get `[STRAWMAN]` —
signals "verify before committing sprint capacity."

---

## arch.md Format

All sections required. Omit only if consultant explicitly stopped (`[INCOMPLETE — consultant stopped before this section]`).

```markdown
# Architecture — <engagement name>
# Generated: <YYYY-MM-DD> | Reviewed by: <consultant name if known>
# Source: mvp-scope.md (<framing> framing, <N> features in scope)

## Client Summary
<Plain-language, non-technical view for client / stakeholder sign-off.>
<Covers: what's being built, build phases, risks in business terms, go-live readiness.>
<Generated last, after all technical sections are complete.>
<This section summarises — it does not replace technical sections below.>

## Tech Stack
| Layer | Decision | Status |
|---|---|---|
| Frontend | React | ✓ Confirmed |
| Backend | Node.js + Express | ✓ Confirmed |
| Database | PostgreSQL | [STRAWMAN] — rationale |
| Infra | Azure App Service | ✓ Confirmed |

## Components
### <Feature Name>
- <ComponentName> (<tech>) — <one-line purpose>
**Shared:** <component> — used by [Feature A, Feature B]

## Data Model Hints
### <Feature Name>
- `<table_name>` — <key fields>; relationships: <brief>

## Integration Points
| System | Approach | Risk | Open Questions |
|---|---|---|---|
| Oracle Retail POS | REST API (TBD) | HIGH [STRAWMAN] | API available? Auth method? |

## Build Order
1. <Component> — <reason it comes first>
2. ...

## Sprint Mapping
**Team:** <roster with seniority>
**Timeline:** <start> → <end> (<N> sprints at <N> weeks)

| Sprint | Work | Owner |
|---|---|---|
| 1–2 | <components> | <who> |

## Effort Signals
| Feature | Size | Rationale |
|---|---|---|
| <name> | S/M/L/XL | <why> |

## Open Questions
1. <Question> — <why it matters> — blocks: <component or BACKLOG_GENERATOR ticket>

## STRAWMAN Summary
All tentative decisions — challenge these before dev sprint 1:
- [STRAWMAN] <decision> — <why tentative> — <what would change it>

## Confidence Notes
- <Category>: LOW/WARN — <reason>

## Source Artifacts
- mvp-scope.md — <one-line summary of scope it contained>
```

---

## Rules

1. AI advises, human decides — no silent stack or architecture choices
2. Readiness Gate runs on every invocation — never skipped
3. Client Summary produced last — after all technical sections are complete
4. Client Summary summarises, does not replace — technical sections stay intact
5. STRAWMAN auto-applies for: close calls, MED/LOW confidence sources, XL estimates, HIGH integration risk
6. Effort signals: S/M/L/XL only — no day-count ranges in arch.md (use sprint mapping for that)
7. Never create Odoo tickets — that is BACKLOG_GENERATOR's job
8. LAYER_0_GLOBAL Rule 4 (output limits) and Rule 5 (no narration) apply

---

## Test Fixtures

| Fixture | Path | Purpose |
|---|---|---|
| synthetic-01 (RetailEdge PASS) | tests/fixtures/arch-proposer/synthetic-01/ | PASS-path — clean mvp-scope.md, gate passes, full happy path, produces arch.md |
| synthetic-02-incomplete (TBD) | tests/fixtures/arch-proposer/synthetic-02-incomplete/ | BLOCK-path — missing budget, unresolved CONFLICT, gate blocks inline |

**Note:** synthetic-01 should use the RetailEdge mvp-scope.md produced in this session
(React confirmed, Azure, 2026-09-04 timeline, ₹18L budget, 2 features IN scope).

---

## Verification Checklist

- [ ] Readiness Gate runs on every invocation
- [ ] synthetic-01: all gate fields PASS, stack selection runs per layer, components produced,
      integrations surfaced, build order + sprint mapping produced, effort signals confirmed,
      Client Summary generated last, arch.md saved
- [ ] synthetic-02-incomplete: BLOCKs detected, inline questions one at a time, gate clears
- [ ] Timeline WARN → date asked before sprint mapping
- [ ] Team composition asked before sprint mapping; seniority reflected in sprint assignments
- [ ] XL estimates and HIGH-risk integrations auto-flagged as STRAWMAN
- [ ] Client Summary present and non-technical
- [ ] Client Summary contains no detail absent from technical sections below it
- [ ] No Odoo tickets created
- [ ] No architecture decisions made before consultant confirms stack

---

## Open Risks

1. **Oracle Retail POS integration unknown** — mitigated by HIGH-risk STRAWMAN flag; Open Question
   carries into BACKLOG_GENERATOR
2. **Sprint mapping accuracy depends on team input** — mitigated by asking team composition;
   if not provided, STRAWMAN assumption flagged
3. **Client Summary language quality** — produced last from structured technical sections;
   quality depends on completeness of those sections

---

## Deferred (explicit non-decisions)

- BACKLOG_GENERATOR (V1.4): designed after ARCH_PROPOSER is dogfooded
- SESSION_STATE.md: deferred until orchestrator (V1.2) exists
- Per-engagement PARKING_LOT.md: deferred to V1.2+
- Mobile stack (React Native / Flutter): only presented when Mobile App is IN scope
