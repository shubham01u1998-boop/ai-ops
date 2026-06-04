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
