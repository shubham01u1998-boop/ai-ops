# Discovery — RetailEdge
# Generated: 2026-06-04 | Reviewed by: (consultant name not provided)
# NOTE: This is a deliberately incomplete fixture for testing the MVP_SYNTHESIZER Readiness Gate.
# Expected gate result: 2 BLOCKs (missing Core Problem, unresolved CONFLICT on integrations)

## Project Context
RetailEdge is a retail analytics platform for FreshMart, a chain of 12 grocery stores in
Maharashtra. The platform provides sales dashboards, inventory alerts, and staff scheduling tools.

## Users
**Primary:** Store managers (12 stores) — daily use for inventory and scheduling
**Secondary:** FreshMart HQ operations team — weekly performance review

## Features Mentioned
- Sales Dashboard — daily/weekly/monthly sales by store and category [HIGH]
- Inventory Alerts — low-stock and expiry alerts by SKU [HIGH]
- Staff Scheduling — shift planning with approval flow [MED]
- Supplier Reorder — auto-suggest reorder quantities based on sales velocity [LOW — mentioned once in email]
- Mobile App — store managers access on phone; native app preferred [MED]

## Constraints
- Budget: ₹18 lakhs total approved
- Tech: React (frontend per spec), Vue.js (frontend per CTO email) — CONFLICT unresolved
- Cloud: Azure (preferred — existing Microsoft EA)
- Other: Must integrate with existing POS system (Oracle Retail); offline mode required for stores with poor connectivity

## Confidence Notes
- Tech frontend: CONFLICT — spec says React, CTO email says Vue.js. Not resolved.
- Mobile App: MED — native app mentioned but no platform decision (iOS/Android/both)
- Supplier Reorder: LOW — single mention, no stakeholder confirmation

## Source Docs
- project-brief.txt — FreshMart internal brief: overview, dashboards, inventory alerts, React stack, no timeline
- cto-email.md — CTO post-call notes: Vue.js preference, Azure cloud, offline requirement, mobile app
