# ARCH_PROPOSER Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the ARCH_PROPOSER skill (V1.3) that reads mvp-scope.md, validates it via Readiness Gate, guides a structured architecture conversation, and produces arch.md for both developer handoff and BACKLOG_GENERATOR.

**Architecture:** Single skill file following the same pattern as MVP_SYNTHESIZER.md — Pre-flight → Readiness Gate → 5 conversation sections → Draft Artifact → Save. Test fixtures follow the same RED-before-GREEN pattern: expected_behaviors.md written before the skill file, then manual test run to verify.

**Tech Stack:** Markdown skill file; Bash tool for pre-flight; Read/Write tools for artifact I/O.

**Design spec:** `docs/superpowers/specs/2026-06-04-arch-proposer-design.md`
**Reference skill:** `skills/project-initiator/MVP_SYNTHESIZER.md`

---

### Task 1: Create synthetic-01 fixture — RetailEdge PASS-path input

**Files:**
- Create: `tests/fixtures/arch-proposer/synthetic-01/mvp-scope.md`

This is the input for the PASS-path test. Adapted from the RetailEdge mvp-scope.md produced in
the MVP_SYNTHESIZER session — all 6 gate fields present and clean.

- [ ] **Step 1: Write mvp-scope.md for synthetic-01**

```markdown
# MVP Scope — RetailEdge
# Generated: 2026-06-04 | Reviewed by: Shubham Upadhyay
# Framing: Value-first — scope anchored to highest business value for store managers

## Problem Restatement
FreshMart store managers lack real-time operational visibility across inventory and sales.
The dashboards and inventory alerts are symptoms of a single gap: operational blindness at
the store level. RetailEdge addresses this as one integrated platform decision.

## Users
**Primary:** Store managers (12 stores) — daily use for inventory and sales visibility
**Secondary:** FreshMart HQ operations team — weekly cross-store performance review

## MVP Framing
**Approach:** Value-first
**Rationale:** No confirmed delivery deadline existed at scoping; stakeholder buy-in is
the critical constraint. Scoping to highest business value for store managers anchors
the MVP to demonstrable impact.
**Constraint driving scope:** Stakeholder buy-in before expanding to scheduling and mobile.

## Scope: In
| Feature | Description | Confidence | Rationale |
|---|---|---|---|
| Sales Dashboard | Daily/weekly/monthly sales by store and category | HIGH | Core operational visibility — directly solves operational blindness |
| Inventory Alerts | Real-time low-stock and expiry alerts by SKU | HIGH | Named in brief, high confidence, central to store-level visibility |

## Scope: Out
| Feature | Why Out | Deferred to |
|---|---|---|
| Staff Scheduling | MED confidence; lower immediate impact | Phase 2 |
| Mobile App | No platform decision confirmed; web-responsive for MVP | Phase 2 |
| Supplier Reorder | LOW confidence; single mention; depends on Inventory Alerts | Phase 2 |

## Key User Journeys
1. Store manager → views daily/weekly sales dashboard → identifies underperforming categories and acts — [Sales Dashboard]
2. Store manager → receives low-stock alert → acts on reorder before stockout occurs — [Inventory Alerts]
3. HQ ops team → reviews weekly cross-store performance → identifies and escalates underperforming locations — [Sales Dashboard]

## Success Metrics
- 75% reduction in manual effort — baseline: not yet established → target: confirmed with client before go-live

## Constraints
- Timeline: 2026-09-04 (3-month window confirmed 2026-06-04)
- Budget: ₹18 lakhs total approved
- Tech: Azure (preferred — existing Microsoft EA); React (frontend — confirmed 2026-06-04)
- Other: Must integrate with existing POS system (Oracle Retail); offline mode required for stores with poor connectivity

## Assumptions
- Web-responsive delivery sufficient for MVP; native mobile app deferred to Phase 2
- Oracle Retail POS integration is technically feasible within budget — source: inferred from brief
- Offline mode can be scoped within ₹18 lakhs — source: inferred; not validated

## Open Questions
1. ~~Target go-live date~~ **2026-09-04 confirmed** (3-month window from 2026-06-04)
2. Core Problem baseline — 75% reduction in manual effort requires a confirmed current-state baseline before launch
3. Success metric definition — "manual effort" needs scoping: which workflows, which roles, measured how?

## Effort Signals
⚠ Deferred to ARCH_PROPOSER — sizing without architecture context produces noise, not signal.
ARCH_PROPOSER will add sizing once tech stack and build order are determined.

## Confidence Notes
- Timeline: RESOLVED — 2026-09-04 confirmed 2026-06-04 (was WARN/Open Question at gate).
- Tech frontend: RESOLVED — React confirmed 2026-06-04. ARCH_PROPOSER unblocked.
- Mobile App: MED — native app mentioned but no platform decision (iOS/Android/both)
- Supplier Reorder: LOW — single mention, no stakeholder confirmation

## Source Artifacts
- discovery.md — RetailEdge: 5 features extracted, 2 users identified, tech CONFLICT resolved,
  timeline confirmed; core problem confirmed in MVP_SYNTHESIZER session
```

- [ ] **Step 2: Verify file written correctly**

Run: `cat tests/fixtures/arch-proposer/synthetic-01/mvp-scope.md | head -20`
Expected: Header and Problem Restatement visible.

- [ ] **Step 3: Commit**

```bash
git add tests/fixtures/arch-proposer/synthetic-01/mvp-scope.md
git commit -m "test(arch-proposer): add synthetic-01 RetailEdge PASS-path input"
```

---

### Task 2: Create synthetic-01 expected_behaviors.md — PASS-path test spec

**Files:**
- Create: `tests/fixtures/arch-proposer/synthetic-01/expected_behaviors.md`

This is the test spec. Write it BEFORE writing the skill file (RED phase).

- [ ] **Step 1: Write expected_behaviors.md**

```markdown
# Expected Behaviors — synthetic-01 (RetailEdge PASS-path)
# Tests ARCH_PROPOSER against a clean mvp-scope.md with all gate fields present.

## Readiness Gate
- [ ] Gate runs without being asked
- [ ] All 6 fields PASS: Scope:In ✓, Users ✓, Tech ✓, Timeline ✓, Budget ✓, No CONFLICTs ✓
- [ ] Gate output shown as a block before any conversation
- [ ] No BLOCKs triggered

## Stack Selection
- [ ] Confirmed tech shown as locked (React ✓, Azure ✓)
- [ ] Only unconfirmed layers presented (backend, database, infra)
- [ ] Each layer shows 2–3 options with trade-offs
- [ ] Recommendation stated with rationale
- [ ] One layer at a time — backend before database before infra
- [ ] Consultant can pick A/B/C or propose own option

## Component Design
- [ ] Components derived for Sales Dashboard and Inventory Alerts
- [ ] Shared components (e.g. OracleConnector) identified and flagged as shared
- [ ] Data model hints included (table names + key fields)
- [ ] Presented as single block, one confirmation prompt
- [ ] Consultant can add/rename/split/remove components

## Integration Points
- [ ] Oracle Retail POS surfaced as integration point
- [ ] Offline mode surfaced as integration point
- [ ] Risk ratings present (HIGH/MED/LOW)
- [ ] STRAWMAN marker on Oracle POS (integration method unconfirmed)
- [ ] Open Questions captured for unknowns
- [ ] One confirmation block

## Build Order + Sprint Mapping
- [ ] Team composition asked before sprint mapping
- [ ] Technical dependency order shown (e.g. OracleConnector first)
- [ ] Sprint mapping shown in table: Sprint | Work | Owner
- [ ] Seniority reflected in assignments (senior = complex/integration components)
- [ ] Timeline (2026-06-04 → 2026-09-04) drives sprint count
- [ ] [STRAWMAN] on sprint split if assumptions made
- [ ] One confirmation block

## Effort Signals
- [ ] S/M/L/XL per feature shown in table
- [ ] Rationale column present
- [ ] OracleConnector (HIGH risk) auto-flagged as STRAWMAN
- [ ] Consultant confirms or adjusts sizes

## Draft arch.md
- [ ] Client Summary section present and non-technical
- [ ] Client Summary generated after all technical sections
- [ ] Tech Stack table with confirmed/STRAWMAN status
- [ ] Components section per feature with shared components flagged
- [ ] Data Model Hints section
- [ ] Integration Points table
- [ ] Build Order numbered list
- [ ] Sprint Mapping table with team assignments
- [ ] Effort Signals table
- [ ] Open Questions section
- [ ] STRAWMAN Summary section listing all tentative decisions
- [ ] Confidence Notes section
- [ ] Source Artifacts section
- [ ] Save gate prompt shown before writing file

## Save
- [ ] arch.md written only after explicit consultant approval
- [ ] arch.md written to current working directory
- [ ] Completion message shown with "Next: run BACKLOG_GENERATOR"
```

- [ ] **Step 2: Commit**

```bash
git add tests/fixtures/arch-proposer/synthetic-01/expected_behaviors.md
git commit -m "test(arch-proposer): add synthetic-01 expected behaviors (RED phase)"
```

---

### Task 3: Create synthetic-02-incomplete fixture — BLOCK-path

**Files:**
- Create: `tests/fixtures/arch-proposer/synthetic-02-incomplete/mvp-scope.md`
- Create: `tests/fixtures/arch-proposer/synthetic-02-incomplete/expected_behaviors.md`

Deliberately incomplete mvp-scope.md to trigger gate BLOCKs.
Missing: Budget line, Timeline line, has unresolved CONFLICT.

- [ ] **Step 1: Write incomplete mvp-scope.md**

```markdown
# MVP Scope — StyleMart
# Generated: 2026-06-04 | Reviewed by: (consultant name not provided)
# NOTE: Deliberately incomplete fixture for testing ARCH_PROPOSER Readiness Gate.
# Expected gate result: 3 BLOCKs — missing Budget, missing Timeline, unresolved CONFLICT

## Problem Restatement
StyleMart's fashion retail platform needs a recommendation engine and checkout redesign
to reduce cart abandonment from 68% to under 30%.

## Users
**Primary:** Online shoppers — browse and purchase fashion items
**Secondary:** StyleMart merchandising team — manage product catalog

## MVP Framing
**Approach:** Value-first
**Rationale:** Conversion rate is the primary metric; highest-value features first.
**Constraint driving scope:** Conversion improvement before loyalty/social features.

## Scope: In
| Feature | Description | Confidence | Rationale |
|---|---|---|---|
| Recommendation Engine | ML-based product recommendations on PDP and cart | HIGH | Directly addresses abandonment |
| Checkout Redesign | Simplified 2-step checkout with guest option | HIGH | Removes friction at highest drop-off point |

## Scope: Out
| Feature | Why Out | Deferred to |
|---|---|---|
| Loyalty Program | Lower immediate conversion impact | Phase 2 |
| Social Sharing | No abandonment impact | Phase 2 |

## Key User Journeys
1. Shopper → views product → sees personalised recommendations → adds to cart — [Recommendation Engine]
2. Shopper → reaches checkout → completes in 2 steps as guest → order confirmed — [Checkout Redesign]

## Success Metrics
- Cart abandonment rate: 68% → <30% within 90 days of go-live

## Constraints
- Tech: CONFLICT — backend team prefers Python/FastAPI; CTO wants Node.js for stack consistency. Not resolved.
- Other: Must integrate with existing Shopify storefront via API

## Assumptions
- Shopify API supports the required recommendation injection points
- ML model can be trained on existing 6-month purchase history

## Open Questions
1. Backend framework — CONFLICT: Python/FastAPI vs Node.js (see Confidence Notes)
2. Recommendation model: collaborative filtering vs content-based — not decided

## Effort Signals
⚠ Deferred to ARCH_PROPOSER

## Confidence Notes
- Backend: CONFLICT — Python/FastAPI (backend team) vs Node.js (CTO). Not resolved.
- Recommendation model: MED — model type not decided; affects build complexity significantly

## Source Artifacts
- discovery.md — StyleMart: 2 features in scope, cart abandonment core problem, backend CONFLICT unresolved
```

- [ ] **Step 2: Write expected_behaviors.md for BLOCK-path**

```markdown
# Expected Behaviors — synthetic-02-incomplete (StyleMart BLOCK-path)
# Tests ARCH_PROPOSER Readiness Gate against a deliberately incomplete mvp-scope.md.
# Expected: 3 BLOCKs — missing Budget, missing Timeline, unresolved CONFLICT

## Readiness Gate
- [ ] Gate runs without being asked
- [ ] Scope:In — PASS (2 features present)
- [ ] Users — PASS (Primary present)
- [ ] Tech Constraints — PASS (present, even though CONFLICT)
- [ ] Timeline — BLOCK (Timeline: line missing entirely)
- [ ] Budget — BLOCK (Budget: line missing entirely)
- [ ] Unresolved CONFLICT — BLOCK ("Backend: CONFLICT — ... Not resolved.")
- [ ] Gate output block shown with all 3 BLOCKs marked ✗

## BLOCK Resolution (one at a time)
- [ ] First BLOCK asked: Timeline (or Budget — whichever comes first in gate order)
- [ ] Second BLOCK asked after first is resolved or deferred
- [ ] Third BLOCK asked after second is resolved or deferred
- [ ] Consultant can defer any BLOCK → becomes WARN + Open Question
- [ ] Gate clears after all BLOCKs resolved or deferred
- [ ] Skill proceeds to Stack Selection after gate clears

## Post-Gate
- [ ] Skill continues normally after gate clears
- [ ] Deferred BLOCKs appear as WARNs in arch.md Confidence Notes
- [ ] Deferred CONFLICT appears in STRAWMAN Summary
```

- [ ] **Step 3: Commit**

```bash
git add tests/fixtures/arch-proposer/synthetic-02-incomplete/
git commit -m "test(arch-proposer): add synthetic-02-incomplete StyleMart BLOCK-path fixtures (RED phase)"
```

---

### Task 4: Write ARCH_PROPOSER.md — Pre-flight + Readiness Gate

**Files:**
- Create: `skills/project-initiator/ARCH_PROPOSER.md` (initial, to be extended in Tasks 5–7)

- [ ] **Step 1: Write Pre-flight + Readiness Gate sections**

```markdown
# VERSION: 1.0 | Last updated: 2026-06-04 | Reviewed: pending
# ARCH_PROPOSER — Project Initiator V1.3
# Part of the fiftyfive-tech Project Initiator toolchain.

---

## Purpose

ARCH_PROPOSER reads a completed `mvp-scope.md`, validates it via Readiness Gate, guides
the consultant through a structured architecture conversation (stack selection → component
design → integration points → build order + sprint mapping → effort signals), and produces
`arch.md` for both developer handoff and BACKLOG_GENERATOR.

ARCH_PROPOSER does NOT create Odoo tickets (BACKLOG_GENERATOR) or define MVP scope
(MVP_SYNTHESIZER). Output: clean, consultant-reviewed architecture document ready to
anchor sprint planning and ticket creation.

V1.3 scope: this skill only. No orchestrator. No SESSION_STATE.md.

---

## Pre-flight

Before asking anything, silently:

1. Run `basename "$PWD"` via Bash tool. Use as engagement name throughout session and in
   `arch.md` header.
   - If name looks generic (e.g. `test`, `temp`, `folder`): ask once — "What's the
     engagement or client name?" Store the answer. Do not ask again.

2. Use Read tool to load `mvp-scope.md` from current working directory.
   - If not found: surface this message and stop.

```
No mvp-scope.md found in this folder.

ARCH_PROPOSER requires a completed mvp-scope.md as input.
Run MVP_SYNTHESIZER first from this engagement folder, then run ARCH_PROPOSER.

Expected folder structure:
  ~/fiftyfive-engagements/<client-name>/
    mvp-scope.md    ← produced by MVP_SYNTHESIZER
    discovery.md    ← produced by DISCOVERY
    input/          ← original raw docs
```

3. After loading successfully, output one line:
```
Engagement: <name> | mvp-scope.md loaded. Running readiness check.
```

---

## Readiness Gate

Validate that `mvp-scope.md` has minimum required content. Never skip.

| Field | Where to look | PASS | WARN | BLOCK |
|---|---|---|---|---|
| Scope: In | `## Scope: In` table | ≥1 feature present | — | Table empty or missing |
| Primary User | `## Users` → `**Primary:**` | Present | — | Missing |
| Tech Constraints | `## Constraints` → `Tech:` line | Present (even partial) | — | Line missing entirely |
| Timeline | `## Constraints` → `Timeline:` | Explicit date or window | "not specified" / blank | Line missing entirely |
| Budget | `## Constraints` → `Budget:` | Explicit amount | "not specified" / blank | Line missing entirely |
| No unresolved CONFLICTs | `## Confidence Notes` | No CONFLICT lines lacking "resolved" | — | Any CONFLICT line lacks "resolved" |

**CONFLICT detection:** Scan `## Confidence Notes` for any line containing "CONFLICT".
If that line contains "resolved" as affirmation (not in "Not resolved"), it PASSes.
If negated or absent, it BLOCKs.

**Gate output (show after checking all fields):**
```
Readiness: mvp-scope.md
✓ Scope: In — 2 features
✓ Users — present
✓ Tech — React, Azure
⚠ Timeline — not specified (will be flagged in output)
✗ Budget — line missing
✗ Unresolved CONFLICT: Backend framework
```

**For each BLOCK:** ask one question inline to resolve it. One at a time. Consultant can
say "defer" → becomes WARN + Open Question in arch.md. Do not proceed past the gate until
all BLOCKs resolved or deferred.

**Timeline WARN + sprint mapping:** If Timeline is WARN and consultant doesn't resolve it
at gate, ask for a date before the Build Order section — sprint mapping without a deadline
produces noise, not a plan.
```

- [ ] **Step 2: Verify section structure matches MVP_SYNTHESIZER.md pattern**

Read `skills/project-initiator/MVP_SYNTHESIZER.md` lines 1–50 for comparison.
Confirm: same header format, same Pre-flight structure, same gate output format.

- [ ] **Step 3: Commit initial file**

```bash
git add skills/project-initiator/ARCH_PROPOSER.md
git commit -m "feat(arch-proposer): add Pre-flight + Readiness Gate sections"
```

---

### Task 5: Write ARCH_PROPOSER.md — Stack Selection + Component Design

**Files:**
- Modify: `skills/project-initiator/ARCH_PROPOSER.md`

- [ ] **Step 1: Append Stack Selection section**

```markdown
---

## Stack Selection

### Step 1 — Show confirmed tech
Pull confirmed tech from `## Constraints → Tech:` in mvp-scope.md. Display as locked:
```
Already confirmed from mvp-scope.md:
- Frontend: React ✓
- Cloud: Azure ✓
```

### Step 2 — Present options for unconfirmed layers
For each unconfirmed layer, present 2–3 options from the preferred list, filtered by
constraints (budget, existing infra, integrations). One layer at a time.

Consultant can pick A/B/C or propose their own option. If they propose one outside the
preferred list, confirm it and apply a STRAWMAN marker.

Format:
```
Backend:
A) Node.js + Express — lightweight, same language as React frontend, large Azure ecosystem
B) .NET Core — strong Azure-native, better for legacy POS integration, heavier initial setup
C) FastAPI (Python) — fast to prototype, good for data-heavy dashboards, separate language

Recommendation: A — fits ₹18L budget and 3-month timeline; B only if Oracle POS SDK is .NET-specific.
Which? (A / B / C)
```

Present layers in this order: backend → database → infra → mobile (only if Mobile App is IN scope).

### Preferred stack list (fiftyfive-tech defaults)

| Layer | Options |
|---|---|
| Frontend | React, Vue.js, Next.js |
| Backend | Node.js + Express, .NET Core, FastAPI |
| Database | PostgreSQL, Azure SQL, Cosmos DB |
| Infra | Azure App Service, Azure Container Apps, AKS |
| Mobile | React Native, Flutter (only when Mobile App is IN scope) |

### Step 3 — STRAWMAN markers
Any stack decision that is a close call, tentative, or derived from MED/LOW confidence
source in mvp-scope.md → flag `[STRAWMAN]` in arch.md Tech Stack table.

---

## Component Design

For each feature in `## Scope: In`, silently derive components from the confirmed stack.
Present as a single block:

```
Proposed components for Sales Dashboard:
- DashboardUI (React) — store/date/category filters + chart views
- SalesAPI (Node.js) — /sales/summary, /sales/by-store, /sales/by-category
- SalesAggregator — scheduled job: aggregates raw POS data → summary tables
- OracleConnector — reads from Oracle Retail POS (shared)

Proposed components for Inventory Alerts:
- AlertsUI (React) — low-stock/expiry alert feed per store
- AlertsAPI (Node.js) — /alerts/active, /alerts/dismiss
- AlertEngine — scheduled job: evaluates SKU thresholds → writes alert records
- (reuses OracleConnector — shared)

**Shared component:** OracleConnector — used by [Sales Dashboard, Inventory Alerts].
Built once in Sprint 1 before either feature's API layer.

Data model hints:
- `sales_summary` — store_id, date, category_id, total_sales, units_sold
- `inventory_alerts` — sku_id, store_id, alert_type (low_stock|expiry), triggered_at, dismissed_at
- `oracle_sync_log` — sync_id, synced_at, status, records_processed

Any components to add, rename, split, or remove?
```

One confirmation prompt. Consultant can edit freely. ARCH_PROPOSER re-shows affected
section after changes. Shared components flagged inline and in `arch.md ## Components`.
```

- [ ] **Step 2: Commit**

```bash
git add skills/project-initiator/ARCH_PROPOSER.md
git commit -m "feat(arch-proposer): add Stack Selection + Component Design sections"
```

---

### Task 6: Write ARCH_PROPOSER.md — Integration Points + Build Order + Effort Signals

**Files:**
- Modify: `skills/project-initiator/ARCH_PROPOSER.md`

- [ ] **Step 1: Append Integration Points section**

```markdown
---

## Integration Points

Built silently from `## Constraints → Other:` and the component map. Presented as a
single block with risk ratings:

```
Integration points identified:

1. Oracle Retail POS — reads sales + inventory data
   Proposed approach: REST API (if available) or scheduled DB export
   Auth: TBD — needs Oracle Retail access credentials
   Risk: HIGH [STRAWMAN] — integration method unconfirmed; affects build order
   Open Question: Does Oracle Retail expose a REST API, or is this a DB/file export?

2. Offline mode — stores with poor connectivity
   Proposed approach: Service Worker + IndexedDB cache; sync on reconnect
   Risk: MED [STRAWMAN] — adds frontend complexity; staleness window not confirmed
   Open Question: What's the acceptable data staleness window when offline?

Any integrations to add, change, or remove? Any Open Questions you can resolve now?
```

Unresolved integration Open Questions → carried into `arch.md ## Open Questions`.
BACKLOG_GENERATOR will flag these as blockers on affected tickets.

---

## Build Order + Sprint Mapping

### Team composition input
Before sprint mapping, ask once:
```
What's the team for this engagement?
(e.g. "2 frontend: 5y + 2y, 2 backend: 6y + 3y, 1 QA: 5y")
```
If not provided: proceed with [STRAWMAN] assumption of 1 BE senior + 1 FE senior.

### Layer 1 — Technical dependency order
Derive from shared components and integration risks. Show as ordered list with rationale:
```
Build order (dependency-driven):
1. OracleConnector — shared; both features depend on it
2. SalesAggregator + data model — backend before API
3. SalesAPI + AlertEngine — logic before UI
4. DashboardUI + AlertsUI — frontend last
5. Offline mode layer — additive on top of DashboardUI
```

### Layer 2 — Sprint mapping
Derive from timeline (from mvp-scope.md) + team size + seniority.
Sprint length default: 2 weeks. Ask if consultant wants a different cadence.
Senior devs own complex/integration components. Junior devs on simpler pieces with
review buffer. Present as table:

```
Team: 2 FE (5y+2y), 2 BE (6y+3y), 1 QA (5y)
Timeline: 2026-06-04 → 2026-09-04 (13 weeks / 6 × 2-week sprints)

| Sprint | Work | Owner |
|---|---|---|
| 1–2 | OracleConnector + data model + SalesAggregator | BE-senior (6y) + BE-junior (3y) |
| 3–4 | SalesAPI + AlertEngine + AlertsUI | BE-senior + FE-senior (5y) |
| 5 | DashboardUI + QA integration testing | FE-senior + FE-junior (2y) + QA |
| 6 | Offline mode + UAT + go-live prep | FE-senior + QA |

[STRAWMAN] — assumes parallel BE+FE tracks from Sprint 3. Adjust if dependencies shift.
```

One confirmation block. Consultant can shift components or adjust sprint length.

---

## Effort Signals

S/M/L/XL per feature, derived from component count + integration risk + team seniority.
Proposed by ARCH_PROPOSER, confirmed by consultant.

| Size | Meaning |
|---|---|
| S | 1–3 days, 1 developer, no integration risk |
| M | 1–2 weeks, 1 developer, low integration risk |
| L | 2–4 weeks, 1–2 developers, medium integration risk |
| XL | 4+ weeks, multiple developers, high integration/unknowns |

Present as a table:
```
| Feature | Size | Rationale |
|---|---|---|
| Sales Dashboard | L | 4 components; OracleConnector dependency; data aggregation complexity |
| Inventory Alerts | M | 3 components; reuses OracleConnector; alert logic simpler than aggregation |
| OracleConnector (shared) | L | HIGH-risk [STRAWMAN] — integration method unconfirmed; could become XL if REST API unavailable |
| Offline mode | M | Service Worker + IndexedDB; additive; requires scale testing |

Agree, or adjust any sizes?
```

**Auto-STRAWMAN:** XL estimates and HIGH-risk components automatically get `[STRAWMAN]`.
```

- [ ] **Step 2: Commit**

```bash
git add skills/project-initiator/ARCH_PROPOSER.md
git commit -m "feat(arch-proposer): add Integration Points, Build Order, Effort Signals sections"
```

---

### Task 7: Write ARCH_PROPOSER.md — Draft Artifact + Save + Rules

**Files:**
- Modify: `skills/project-initiator/ARCH_PROPOSER.md`

- [ ] **Step 1: Append Draft Artifact section**

```markdown
---

## Draft Artifact

Produce the Client Summary last, after all technical sections are complete. Show the
full draft to the consultant. Use this exact format:

```markdown
# Architecture — <engagement name>
# Generated: <YYYY-MM-DD> | Reviewed by: <consultant name if known>
# Source: mvp-scope.md (<framing> framing, <N> features in scope)

## Client Summary
<Plain-language, non-technical view — 3–5 sentences.>
<Covers: what's being built, build phases, key risks in business terms, go-live readiness.>
<No component names, no framework names, no technical jargon.>
<This section summarises — it does not replace technical sections below.>

## Tech Stack
| Layer | Decision | Status |
|---|---|---|
| Frontend | React | ✓ Confirmed |
| Backend | Node.js + Express | ✓ Confirmed |
| Database | PostgreSQL | [STRAWMAN] — Azure SQL viable if Oracle POS requires it |
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
| Offline mode | Service Worker + IndexedDB | MED [STRAWMAN] | Staleness window? |

## Build Order
1. <Component> — <reason>

## Sprint Mapping
**Team:** <roster with seniority>
**Timeline:** <start> → <end> (<N> sprints × <N> weeks)

| Sprint | Work | Owner |
|---|---|---|
| 1–2 | ... | ... |

## Effort Signals
| Feature | Size | Rationale |
|---|---|---|
| <name> | S/M/L/XL | <why> |

## Open Questions
1. <Question> — <why it matters> — blocks: <component or ticket>

## STRAWMAN Summary
All tentative decisions — challenge these before dev sprint 1:
- [STRAWMAN] <decision> — <why tentative> — <what would change it>

## Confidence Notes
- <Category>: LOW/WARN — <reason>

## Source Artifacts
- mvp-scope.md — <one-line summary>
```

After showing the draft:
```
Looks complete — save arch.md? [A] Yes / [B] Edit a section / [C] Add more context
```

Natural-language approval ("yes", "save it", "looks good") → interpret as [A].
Edit offered → interpret as [B]. New info offered → interpret as [C].
Ambiguous → surface intent: "Reading this as [A] Save — correct?"

**Client Summary rule:** If any detail appears in the Client Summary that is absent from
the technical sections below it, the technical sections are incomplete. Fix them first.

---

## Save

Per LAYER_0_GLOBAL Rule 1: the Draft Artifact prompt is the permission gate. Do not write
`arch.md` until the consultant explicitly approves.

Write `arch.md` to the current working directory.

Output:
```
arch.md saved. ARCH_PROPOSER complete.
Next: run BACKLOG_GENERATOR to generate Odoo tickets from arch.md (not yet built — V1.4).
```

---

## Rules

1. Never create Odoo tickets — that is BACKLOG_GENERATOR's job (V1.4).
2. Never auto-select stack or architecture — surface proposed call with rationale, consultant confirms.
3. Readiness Gate: never skip it. Every invocation.
4. Client Summary produced last — after all technical sections are complete.
5. Client Summary summarises, does not replace — technical sections stay intact and complete.
6. STRAWMAN auto-applies for: close calls, MED/LOW confidence sources, tentative decisions, XL estimates, HIGH integration risk.
7. Effort signals: S/M/L/XL only. No day-count ranges in arch.md (use sprint mapping for that).
8. LAYER_0_GLOBAL Rule 4 output limits and Rule 5 (no narration) apply.
```

- [ ] **Step 2: Verify complete skill file structure**

Read `skills/project-initiator/ARCH_PROPOSER.md` in full.
Confirm all sections present: Purpose, Pre-flight, Readiness Gate, Stack Selection,
Component Design, Integration Points, Build Order + Sprint Mapping, Effort Signals,
Draft Artifact, Save, Rules.

- [ ] **Step 3: Commit**

```bash
git add skills/project-initiator/ARCH_PROPOSER.md
git commit -m "feat(arch-proposer): add Draft Artifact, Save, Rules — skill file complete"
```

---

### Task 8: Manual test — synthetic-01 PASS-path

**Files:**
- Create: `tests/fixtures/arch-proposer/synthetic-01/arch.md` (produced during test)

Run ARCH_PROPOSER against synthetic-01 and verify against expected_behaviors.md.

- [ ] **Step 1: Run ARCH_PROPOSER on synthetic-01**

Navigate to `tests/fixtures/arch-proposer/synthetic-01/` (or simulate from that context).
Say: `run ARCH_PROPOSER`
Watch the skill execute. Do not prompt it — let it run its pre-flight and gate automatically.

- [ ] **Step 2: Walk through the conversation**

Respond as a consultant:
- Stack selection: pick A for backend, PostgreSQL for database, Azure App Service for infra
- Component design: accept proposed components
- Integration points: accept; defer Oracle POS approach to Open Questions
- Team: "2 FE: 5y + 2y, 2 BE: 6y + 3y, 1 QA: 5y"
- Build order + sprints: accept proposed mapping
- Effort signals: accept proposed sizes
- Draft: approve → save

- [ ] **Step 3: Verify against expected_behaviors.md**

Open `tests/fixtures/arch-proposer/synthetic-01/expected_behaviors.md`.
Check each item. Mark any failures.

- [ ] **Step 4: Document gaps**

For each failed expected behavior, note the exact skill text that caused the failure.
These become REFACTOR tasks (Task 10).

- [ ] **Step 5: Commit arch.md output**

```bash
git add tests/fixtures/arch-proposer/synthetic-01/arch.md
git commit -m "test(arch-proposer): synthetic-01 PASS-path test run complete"
```

---

### Task 9: Manual test — synthetic-02-incomplete BLOCK-path

Run ARCH_PROPOSER against synthetic-02-incomplete and verify gate behavior.

- [ ] **Step 1: Run ARCH_PROPOSER on synthetic-02-incomplete**

Navigate to `tests/fixtures/arch-proposer/synthetic-02-incomplete/` context.
Say: `run ARCH_PROPOSER`

- [ ] **Step 2: Walk through the gate**

Respond as a consultant:
- Timeline BLOCK: defer — "No deadline confirmed yet"
- Budget BLOCK: provide — "₹20 lakhs total"
- CONFLICT BLOCK: defer — "Backend framework to be resolved with CTO before arch phase"

- [ ] **Step 3: Verify gate behavior against expected_behaviors.md**

Check each item in `tests/fixtures/arch-proposer/synthetic-02-incomplete/expected_behaviors.md`.
Confirm: 3 BLOCKs shown, one at a time, deferred items become WARNs in output.

- [ ] **Step 4: Document gaps**

Same as Task 8 Step 4.

- [ ] **Step 5: Commit**

```bash
git add tests/fixtures/arch-proposer/synthetic-02-incomplete/
git commit -m "test(arch-proposer): synthetic-02-incomplete BLOCK-path test run complete"
```

---

### Task 10: Iterate — close gaps found in testing

**Files:**
- Modify: `skills/project-initiator/ARCH_PROPOSER.md`

- [ ] **Step 1: Review all failures from Tasks 8 and 9**

List each gap. For each: identify which skill section is missing, ambiguous, or wrong.

- [ ] **Step 2: Fix each gap in ARCH_PROPOSER.md**

For each failure: edit the relevant section. Apply the same fix pattern as MVP_SYNTHESIZER —
close the loophole explicitly (not just restate the rule more firmly).

- [ ] **Step 3: Re-run affected test scenarios**

If gate behavior was wrong → re-run synthetic-02-incomplete gate section.
If output format was wrong → re-run synthetic-01 draft section.
Verify the fix resolved the failure without introducing new ones.

- [ ] **Step 4: Update CLAUDE.md**

Add ARCH_PROPOSER to the Phase Status table:
```
| PI V1.3 | ARCH_PROPOSER skill — built, tested | Done |
```

Update the Chain status section to reflect ARCH_PROPOSER complete.

- [ ] **Step 5: Final commit**

```bash
git add skills/project-initiator/ARCH_PROPOSER.md CLAUDE.md
git commit -m "feat(arch-proposer): v1.0 complete — gap closure after test runs"
```
