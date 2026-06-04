# Architecture — RetailEdge
# Generated: 2026-06-04 | Reviewed by: Shubham Upadhyay
# Source: mvp-scope.md (Value-first framing, 2 features in scope)

## Client Summary
RetailEdge will give FreshMart store managers a unified operational platform covering daily
sales visibility and real-time inventory alerts across all 12 stores. The build runs in two
parallel tracks — a data integration layer first to connect the existing point-of-sale system,
followed by the dashboard and alerting surfaces that store managers will use day to day. The
biggest business risk is the POS integration: the technical method has not yet been confirmed
and could extend the timeline if the system does not expose a direct connection. The platform
is designed for web access from any store, with offline support for locations with poor
connectivity, targeting go-live by 2026-09-04.

## Tech Stack
| Layer | Decision | Status |
|---|---|---|
| Frontend | React | ✓ Confirmed |
| Backend | Node.js + Express | ✓ Confirmed |
| Database | PostgreSQL | [STRAWMAN] — Azure SQL viable if Oracle POS requires it |
| Infra | Azure App Service | ✓ Confirmed |
| Cloud | Azure | ✓ Confirmed |

## Components
### Sales Dashboard
- DashboardUI (React) — store/date/category filters + chart views
- SalesAPI (Node.js) — /sales/summary, /sales/by-store, /sales/by-category
- SalesAggregator (Node.js) — scheduled job: aggregates raw POS data → summary tables
- OracleConnector (Node.js) — reads from Oracle Retail POS **(shared)**

### Inventory Alerts
- AlertsUI (React) — low-stock/expiry alert feed per store
- AlertsAPI (Node.js) — /alerts/active, /alerts/dismiss
- AlertEngine (Node.js) — scheduled job: evaluates SKU thresholds → writes alert records
- OracleConnector — reused (shared)

**Shared:** OracleConnector — used by [Sales Dashboard, Inventory Alerts]. Built once in Sprint 1 before either feature's API layer.

## Data Model Hints
### Sales Dashboard
- `sales_summary` — store_id, date, category_id, total_sales, units_sold; relationships: store → sales_summary (1:many)

### Inventory Alerts
- `inventory_alerts` — sku_id, store_id, alert_type (low_stock|expiry), triggered_at, dismissed_at; relationships: sku → alerts (1:many)

### Shared
- `oracle_sync_log` — sync_id, synced_at, status, records_processed; used by SalesAggregator and AlertEngine

## Integration Points
| System | Approach | Risk | Open Questions |
|---|---|---|---|
| Oracle Retail POS | Method TBD — deferred | HIGH [STRAWMAN] | API available? DB export? Auth method? |
| Offline mode | Service Worker + IndexedDB cache; sync on reconnect | MED [STRAWMAN] | Acceptable data staleness window when offline? |

## Build Order
1. OracleConnector — shared; both Sales Dashboard and Inventory Alerts depend on it
2. SalesAggregator + data model — backend aggregation layer before API
3. SalesAPI + AlertEngine — business logic before UI surfaces
4. DashboardUI + AlertsUI — frontend surfaces last
5. Offline mode layer — additive on top of DashboardUI

## Sprint Mapping
**Team:** 2 FE (5y + 2y), 2 BE (6y + 3y), 1 QA (5y)
**Timeline:** 2026-06-04 → 2026-09-04 (13 weeks / 6 × 2-week sprints)

| Sprint | Work | Owner |
|---|---|---|
| 1–2 | OracleConnector + data model + SalesAggregator | BE-senior (6y) + BE-junior (3y) |
| 3–4 | SalesAPI + AlertEngine + AlertsUI | BE-senior + FE-senior (5y) |
| 5 | DashboardUI + QA integration testing | FE-senior + FE-junior (2y) + QA |
| 6 | Offline mode + UAT + go-live prep | FE-senior + QA |

[STRAWMAN] — assumes parallel BE+FE tracks from Sprint 3. Adjust if dependencies shift.

## Effort Signals
| Feature | Size | Rationale |
|---|---|---|
| Sales Dashboard | L | 4 components; OracleConnector dependency; data aggregation complexity |
| Inventory Alerts | M | 3 components; reuses OracleConnector; alert logic simpler than aggregation |
| OracleConnector (shared) | L | HIGH risk [STRAWMAN] — integration method unconfirmed; could become XL if REST API unavailable |
| Offline mode | M | Service Worker + IndexedDB; additive; requires scale testing |

## Open Questions
1. Oracle Retail POS integration method — REST API available, or DB/file export? Affects build order and OracleConnector design — blocks: OracleConnector, Sprint 1 scope
2. Offline mode staleness window — what's the acceptable data age when connectivity is lost? Affects Service Worker cache strategy — blocks: Offline mode (Sprint 6)
3. Core Problem baseline — "75% reduction in manual effort" requires a confirmed current-state baseline before go-live — blocks: Success Metrics validation
4. Success metric definition — "manual effort" needs scoping: which workflows, which roles, measured how? — blocks: UAT acceptance criteria

## STRAWMAN Summary
All tentative decisions — challenge these before dev Sprint 1:
- [STRAWMAN] PostgreSQL — Azure SQL viable if Oracle POS integration requires MSSQL compatibility; confirm once POS integration method is known
- [STRAWMAN] Oracle Retail POS integration method — deferred; REST API, DB export, or file-based sync all possible; HIGH risk to build order and effort
- [STRAWMAN] OracleConnector effort = L — could become XL if Oracle does not expose a direct API and custom extraction is required
- [STRAWMAN] Offline mode staleness window — MED risk; Service Worker + IndexedDB approach confirmed, but acceptable cache age not yet validated with client
- [STRAWMAN] Sprint 3–6 parallel BE+FE tracks — assumes dependencies are resolved by Sprint 3; shift if OracleConnector integration is delayed

## Confidence Notes
- Oracle POS integration: WARN — method unconfirmed; single highest-risk item in build plan
- Offline mode scope: WARN — budget and staleness assumptions inferred, not validated with client
- Success metrics baseline: WARN — 75% reduction target needs current-state measurement before UAT
- All other scope items: HIGH — confirmed in MVP_SYNTHESIZER session

## Source Artifacts
- mvp-scope.md — RetailEdge: 2 features in MVP scope (Sales Dashboard, Inventory Alerts), Value-first framing, timeline 2026-09-04 confirmed, ₹18L budget confirmed
