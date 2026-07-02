# MVP Scope — SupplySync
# Generated: 2026-06-04 | Reviewed by: (consultant name not provided)
# Framing: Time-boxed — scope constrained to what can ship by December 1, 2026 (CFO hard deadline)

## Problem Restatement
Prakash Distributors' procurement team operates without a shared system — each manager works
from separate Excel files, leading to double-order incidents and missed contract renewals.
Invoice reconciliation alone consumes 2–3 days per week per manager. There is no audit trail
for supplier communication, creating accountability and compliance gaps. SupplySync exists to
replace this fragmented workflow with a single source of truth.

## Users
**Primary:** 4 procurement managers (Prakash Distributors) — daily operational use
**Secondary:** Rohan Kapoor, Head of Procurement — dashboard and oversight view
**Suppliers:** Notification recipients only; no login access in v1

## MVP Framing
**Approach:** Time-boxed
**Rationale:** The December 1, 2026 go-live is a hard CFO deadline driven by a government
client compliance requirement. Scope must fit the timeline — features that cannot land by
December 1 are deferred, not descoped.
**Constraint driving scope:** December 1, 2026 hard deadline (CFO, Anita Sharma)

## Scope: In
| Feature | Description | Confidence | Rationale |
|---|---|---|---|
| Supplier Onboarding | Supplier form with GST/PAN upload, admin approval flow | HIGH | Core requirement; prerequisite for order flow |
| Order Management | Raise PO, track status (Pending → Confirmed → Dispatched → Received) | HIGH | Central workflow; solves ordering chaos and creates audit trail |
| Contract Management | Upload contracts, set expiry reminders, version history | HIGH | Directly solves missed renewal problem |
| Invoice Reconciliation | Auto-match invoice to PO, flag discrepancies | HIGH | Directly solves core problem (2–3 days/week loss) |
| Email Notifications | Order confirmations, contract expiry alerts | HIGH | Required for order flow; table stakes |
| WhatsApp Notifications | PO confirmation messages to suppliers; Hindi language support | MED | Confirmed as v1 requirement during DISCOVERY |
| Procurement Dashboard | Overview view for Head of Procurement | MED | Secondary user need; visibility into operations |

## Scope: Out
| Feature | Why Out | Deferred to |
|---|---|---|
| Supplier Rating / Scoring | Single-source (CFO email only); quarterly cycle non-essential for go-live; deferred per consultant | Phase 2 |

## Key User Journeys
1. Procurement Manager → onboards new supplier with GST/PAN docs → supplier approved and live for ordering — [Supplier Onboarding, Contract Management]
2. Procurement Manager → raises purchase order → supplier notified via email + WhatsApp → PO tracked to Received with full audit trail — [Order Management, Email Notifications, WhatsApp Notifications]
3. Procurement Manager → receives invoice → system auto-matches to PO → discrepancies surfaced for review — [Invoice Reconciliation, Order Management]
4. Head of Procurement → opens dashboard → views current supplier and order status at a glance — [Procurement Dashboard]

## Success Metrics
- Invoice reconciliation time: baseline 2–3 days/week per manager → target <4 hours/week
- Double-order incidents: baseline unknown → target zero in first 30 days post-launch

## Constraints
- Timeline: December 1, 2026 go-live (hard deadline, CFO Anita Sharma); Q3 2026 internal soft launch acceptable
- Budget: TBD — CFO to share after NDA signed; do not include figures in any document until confirmed
- Tech: React (frontend), Node.js + Express (backend), PostgreSQL (database); WhatsApp Business API required; AWS Mumbai (India data residency, hard requirement)
- Other: No supplier login in v1 (link-based notification only); mobile-responsive website, no native app; ERP (SAP B1) integration deferred to Phase 2

## Assumptions
- WhatsApp Business API can be integrated and tested within the December 1 timeline — source: assumed from DISCOVERY confirmation
- India data residency (AWS Mumbai) is achievable within the (yet-unknown) budget — source: inferred from CFO hard constraint
- 4 procurement managers can be onboarded and trained before December 1 go-live — source: inferred from team size

## Open Questions
1. Budget — CFO (Anita Sharma) to share approved figures after NDA signed. Do not include numbers in any document until confirmed.
2. ERP integration (SAP B1) — confirmed deferred to Phase 2; no timeline or scope commitment exists yet.

## Effort Signals
⚠ Deferred to ARCH_PROPOSER — sizing without architecture context produces noise, not signal.
ARCH_PROPOSER will add sizing once tech stack and build order are determined.

## Confidence Notes
- Budget: LOW — no figures available; gated on NDA + CFO call (carried from discovery.md)

## Source Artifacts
- discovery.md — completed DISCOVERY session covering 8 features across 3 source docs; hard constraints include December 1 deadline and AWS Mumbai data residency
