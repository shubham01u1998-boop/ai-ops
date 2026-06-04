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
If that line contains "resolved" as affirmation (not negated as in "Not resolved"), it PASSes.
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

Recommendation: A — fits budget and timeline; B only if Oracle POS SDK is .NET-specific.
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

---

## Draft Artifact

Produce the Client Summary last, after all technical sections are complete. Show the
full draft to the consultant. Use this exact format:

```markdown
# Architecture — <engagement name>
# Generated: <YYYY-MM-DD> | Reviewed by: <consultant name if known>
# Source: mvp-scope.md (<framing> framing, <N> features in scope)

## Client Summary
<Plain-language, non-technical view — 3-5 sentences.>
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
| 1-2 | ... | ... |

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

## Write Session State

After saving arch.md, silently:

1. Write/update `session_state.md` in the current directory:

```markdown
# Session State — <engagement name>
# Updated: <YYYY-MM-DD>

## Current Stage
Last completed: ARCH_PROPOSER
Status: complete

## Next Step
Run: DOC_GENERATOR
From: <current directory path>

## Open Items
<list each item from arch.md ## Open Questions section, one per line as:
- <question> — source: ARCH_PROPOSER — blocks: <component or ticket noted in arch.md>>
If no open questions: (none)

## Notes
```

2. If `project.md` exists in the current directory, update two fields only:
   - `Stage: ARCH_PROPOSER complete`
   - `Last session: <today: YYYY-MM-DD>`
   Leave all other fields unchanged.

3. If `project.md` does not exist: skip step 2 silently.

Output one line: `session_state.md updated.`

---

## Rules for this skill

1. Never create Odoo tickets — that is BACKLOG_GENERATOR's job (V1.4).
2. Never auto-select stack or architecture — surface proposed call with rationale, consultant confirms.
3. Readiness Gate: never skip it. Every invocation.
4. Client Summary produced last — after all technical sections are complete.
5. Client Summary summarises, does not replace — technical sections stay intact and complete.
6. STRAWMAN auto-applies for: close calls, MED/LOW confidence sources, tentative decisions, XL estimates, HIGH integration risk.
7. Effort signals: S/M/L/XL only. No day-count ranges in arch.md (use sprint mapping for that).
8. LAYER_0_GLOBAL Rule 4 output limits and Rule 5 (no narration) apply.
