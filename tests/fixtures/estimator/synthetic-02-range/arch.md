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
- `sales_summary` — store_id, date, category_id, total_sales, units_sold

### Inventory Alerts
- `inventory_alerts` — sku_id, store_id, alert_type (low_stock|expiry), triggered_at, dismissed_at

### Shared
- `oracle_sync_log` — sync_id, synced_at, status, records_processed

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
**Team:** 1 FE-senior (React), 1 BE-senior (Node.js), 1 BE-junior (Node.js), 1 QA
**Timeline:** 24 working days — 6 × 2-week sprints (part-time team)
**Project start:** 2026-06-04

| Sprint | Work | Owner |
|---|---|---|
| Sprint 1-2 (D1–8) | OracleConnector | BE-senior |
| Sprint 1-2 (D1–8) | SalesAggregator + data model | BE-junior |
| Sprint 3-4 (D9–16) | SalesAPI + AlertEngine | BE-senior + BE-junior |
| Sprint 5 (D17–20) | DashboardUI + AlertsUI | FE-senior |
| Sprint 6 (D21–24) | Offline mode layer | FE-senior |

## Effort Signals
| Feature | Size | Rationale |
|---|---|---|
| OracleConnector | L | Shared connector; integration risk; method TBD |
| SalesAggregator + data model | M | Scheduled job + schema; moderate complexity |
| SalesAPI + AlertEngine | L | Two API layers + business logic |
| DashboardUI + AlertsUI | M | Two React surfaces; no backend risk |
| Offline mode layer | S | Additive; Service Worker pattern |

## Open Questions
1. Oracle Retail POS integration method not confirmed — API, DB export, or file transfer?
   Blocks: OracleConnector
2. Acceptable data staleness window for offline mode?
   Blocks: Offline mode layer

## STRAWMAN Summary
- [STRAWMAN] PostgreSQL vs Azure SQL — deferred pending Oracle POS method confirmation
- [STRAWMAN] Oracle POS integration method — API, DB export, or file transfer; HIGH risk
- [STRAWMAN] Offline mode staleness window — UX decision needed before Sprint 1
