# Scope Agreement — RetailEdge
**Client:** FreshMart
**Prepared by:** fiftyfive technologies
**Date:** 2026-06-04
**Status:** Pending client sign-off

---

## 1. Problem Statement

FreshMart store managers currently lack unified, real-time visibility into store operations.
Sales data, inventory status, and scheduling exist as disconnected tools — leading to missed
stockouts, expiry incidents, and scheduling gaps across 12 stores. RetailEdge is a single
integrated platform to close this gap.

---

## 2. Users

| Role | Usage |
|---|---|
| Store Managers (12 stores) | Daily — inventory monitoring, sales review |
| FreshMart HQ Operations | Weekly — cross-store performance review |

---

## 3. MVP Scope

### In Scope

| Feature | Description |
|---|---|
| Sales Dashboard | Daily, weekly, and monthly sales by store and category |
| Inventory Alerts | Real-time low-stock and expiry alerts by SKU |

### Out of Scope (Phase 2)

| Feature | Reason |
|---|---|
| Staff Scheduling | Deferred — lower immediate impact; included in Phase 2 |
| Mobile App | Deferred — platform decision (iOS/Android) not confirmed; web-responsive delivery for MVP |
| Supplier Reorder | Deferred — insufficient requirements; depends on Inventory Alerts being live |

---

## 4. Key User Journeys

The MVP must support these end-to-end flows at go-live:

1. Store manager views daily/weekly sales dashboard → identifies underperforming categories → acts
2. Store manager receives low-stock alert → acts on reorder before stockout occurs
3. HQ operations team reviews weekly cross-store performance → identifies and escalates underperforming locations

---

## 5. Constraints

- **Budget:** ₹18 lakhs (total approved)
- **Infrastructure:** Azure (existing Microsoft EA)
- **Frontend:** React (confirmed 2026-06-04)
- **Integrations:** Oracle Retail POS system (existing); offline mode required for stores with poor connectivity
- **Timeline:** 2026-09-04 (3-month window confirmed 2026-06-04)

---

## 6. Assumptions

The following are treated as true for scoping purposes. If any assumption is incorrect,
scope and budget may need to be revisited.

1. Web-responsive delivery is sufficient for MVP — native mobile app is Phase 2.
2. Oracle Retail POS integration is technically feasible within the ₹18 lakhs budget.
3. Offline mode requirement is achievable within approved budget.

---

## 7. Pre-Conditions for Project Start

The following must be confirmed before architecture and development begins:

| # | Item | Owner |
|---|---|---|
| 1 | ~~Target go-live date~~ **2026-09-04 confirmed** | ✓ Resolved |
| 2 | ~~Frontend framework decision~~ **React confirmed** 2026-06-04 | ✓ Resolved |
| 3 | Success metric baseline: current manual effort in hours/process to establish 75% reduction target | FreshMart |

---

## 8. Success Metric

**Target:** 75% reduction in manual operational effort for store managers
**Baseline:** To be confirmed with client before go-live measurement

---

## 9. What This Agreement Does Not Cover

- Architecture decisions (to follow in ARCH_PROPOSER phase)
- Sprint plan or delivery timeline (requires Pre-Conditions above to be resolved)
- Phase 2 features (Staff Scheduling, Mobile App, Supplier Reorder)

---

## 10. Sign-Off

By signing below, both parties confirm that the scope described in this document is agreed
and that work will not commence until Pre-Conditions in Section 7 are resolved.

| | |
|---|---|
| **FreshMart** | |
| Name: | __________________ |
| Title: | __________________ |
| Date: | __________________ |
| | |
| **fiftyfive technologies** | |
| Name: | __________________ |
| Title: | __________________ |
| Date: | __________________ |
