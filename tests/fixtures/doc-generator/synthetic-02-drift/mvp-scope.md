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
- Tech: Azure (preferred — existing Microsoft EA); React (frontend — confirmed 2026-06-04); .NET Core (backend — confirmed 2026-06-04)
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
- Tech backend: RESOLVED — .NET Core confirmed 2026-06-04.
- Mobile App: MED — native app mentioned but no platform decision (iOS/Android/both)
- Supplier Reorder: LOW — single mention, no stakeholder confirmation

## Source Artifacts
- discovery.md — RetailEdge: 5 features extracted, 2 users identified, tech CONFLICT resolved,
  timeline confirmed; core problem confirmed in MVP_SYNTHESIZER session
