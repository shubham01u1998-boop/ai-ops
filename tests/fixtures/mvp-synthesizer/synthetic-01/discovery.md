# Discovery — SupplySync
# Generated: 2026-06-03 | Reviewed by: (consultant name not provided)

## Project Context
SupplySync is a web-based supplier management platform for Prakash Distributors, a mid-size
food distribution company managing ~80 suppliers. The platform replaces a manual workflow
across Excel spreadsheets, email threads, and WhatsApp groups. The engagement is being
delivered by fiftyfive-tech.

## Users
**Primary:** 4 procurement managers (internal, Prakash Distributors) — daily operational use
**Secondary:** Rohan Kapoor, Head of Procurement — dashboard / oversight view
**Suppliers:** Receive notifications via links only; no login access in v1

## Core Problem
The procurement team has no single source of truth — managers work in separate Excel files,
leading to double-order incidents and missed contract renewals. Invoice reconciliation consumes
2–3 days per week per manager. There is no audit trail for supplier communication.

## Features Mentioned
- Supplier Onboarding — supplier form with GST/PAN upload, admin approval flow [HIGH]
- Order Management — raise PO, track status (Pending → Confirmed → Dispatched → Received) [HIGH]
- Contract Management — upload contracts, set expiry reminders, version history [HIGH]
- Invoice Reconciliation — auto-match invoice to PO, flag discrepancies [HIGH]
- Email Notifications — order confirmations, contract expiry alerts [HIGH]
- WhatsApp Notifications — PO confirmation messages to suppliers; Hindi language support confirmed [MED — confirmed v1, not in original spec]
- Supplier Rating / Scoring — rate suppliers on delivery time, quality, responsiveness (each /5) after order Received; quarterly scorecard for contract renewal decisions [MED — CFO email only]
- Procurement Dashboard — overview view for Procurement Head [MED — meeting notes only]

## Constraints
- Timeline: December 1, 2026 go-live (hard deadline, CFO); Q3 2026 soft/internal launch acceptable
- Budget: TBD — CFO to share in separate call; NDA required before documentation
- Tech: React (frontend), Node.js + Express (backend), PostgreSQL (database); WhatsApp Business API required for notifications
- Cloud: AWS Mumbai (hard requirement — India data residency mandated by CFO for government client compliance)
- Other: No supplier login in v1 (link-based only); mobile-responsive website, no native app; ERP (SAP B1) integration deferred to Phase 2

## Open Questions
1. Budget — CFO (Anita Sharma) to share approved figures in a separate call; NDA must be signed first. Do not include numbers in any document until then.
2. ERP integration (SAP B1) — confirmed deferred to Phase 2; no timeline or scope commitment exists yet.

## Confidence Notes
- Tech: CONFLICT resolved — React confirmed (Vue.js was Rohan's preference for maintainability; not adopted)
- Timeline: CONFLICT resolved — December 1, 2026 confirmed by CFO as authoritative deadline (Q3 in spec was a draft estimate)
- Budget: LOW — no figures available; gated on NDA + CFO call
- WhatsApp notifications: critical gap resolved — confirmed as v1 requirement with Hindi language support
- Data residency: critical gap resolved — AWS Mumbai confirmed as hard architectural constraint

## Source Docs
- rough-spec.md — Internal spec draft: project overview, initial features, React/Node.js/PostgreSQL stack, Q3 timeline (both timeline and React since superseded/confirmed)
- meeting-notes-1.md — Discovery call with Rohan Kapoor: user breakdown, scale (~80 suppliers, 200–300 POs/month), Vue.js preference, December deadline, WhatsApp requirement, budget unknown
- stakeholder-email.txt — CFO Anita Sharma post-meeting: supplier rating requirement, India data residency (hard), December 1 go-live (authoritative), budget NDA requirement
