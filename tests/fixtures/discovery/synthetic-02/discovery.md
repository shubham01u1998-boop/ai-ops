# Discovery — StyleMart
# Generated: 2026-06-03 | Reviewed by: (consultant name not provided)

## Project Context
StyleMart is a branded e-commerce platform for Kavya Fashions Pvt Ltd, an Indian apparel retailer
currently operating through 3 physical stores and Instagram DMs. The platform replaces a manual
sales workflow — Instagram DM orders, manual UPI QR code payments, no order tracking — with a
structured online channel. The engagement is delivered by fiftyfive-tech.

## Users
**Primary:** End customers — browse, purchase, track orders
**Secondary:** Store admin (Kavya Fashions staff) — manage products, orders, inventory, returns;
Delivery partner staff — update order delivery status

## Core Problem
Kavya Fashions has no online sales infrastructure. Orders are placed via Instagram comments and DMs,
payments collected via manual UPI QR code, and there is no order tracking or customer data captured.
This creates operational load, limits growth beyond walk-in customers, and makes scale impossible
without a proper platform.

## Features Mentioned
- Product Catalog — categories, search, filters, size guides [HIGH]
- Cart and Checkout — COD, UPI, cards, net banking; payment gateway: Razorpay [HIGH]
- Order Tracking — real-time delivery status [HIGH]
- Customer Accounts — purchase history, saved addresses [HIGH]
- Returns and Exchange Management — workflow TBD (deferred) [HIGH]
- Admin Dashboard — product, order, and inventory management [HIGH]
- Email + SMS Notifications — order confirmations and delivery updates [HIGH]
- Loyalty Points — earn 1pt per ₹100 spent; redeem 100pts = ₹50 discount; expire after 12 months inactivity [HIGH — confirmed with rules, client-followup]
- COD (Cash on Delivery) — confirmed essential; ~60–70% of orders expected at launch [HIGH — client-followup]

## Constraints
- Timeline: Soft launch (web only, ~500 SKUs) by October 20, 2026 (hard deadline — before Diwali); full launch (web + app, full catalog) Q1–Q2 2027
- Budget: TBD — CFO (Priya Sharma) to share separately; NDA required before documentation
- Tech: React (frontend) + Node.js + PostgreSQL (backend/DB); React Native for mobile (web and app launched together)
- Payments: Razorpay; COD confirmed essential — logistics and return flows must be designed for COD from day 1
- Inventory: Tally POS (existing) — no integration Phase 1; ~500 SKUs at launch; Phase 2 integration planned
- Other: Single brand only — no marketplace (confirmed, client-followup); mobile + web launched simultaneously

## Open Questions
1. Budget — CFO (Priya Sharma) to share approved figures separately; NDA must be signed first.
2. Tally/inventory integration — confirmed deferred to Phase 2; manual inventory process for Phase 1; no details defined.
3. Return/exchange workflow — feature confirmed in scope but no process, SLA, or refund timeline defined.

## Confidence Notes
- Tech: CONFLICT resolved — custom React + Node.js + PostgreSQL confirmed (Shopify ruled out; loyalty customisation and future flexibility cited by Arjun Mehta)
- Mobile: CONFLICT resolved — web + app launched together confirmed (product-brief Phase 2 position overridden by client-followup)
- Timeline: CONFLICT resolved — October 20, 2026 confirmed as hard soft-launch deadline (Q3 2026 in product-brief was a draft estimate)
- Multi-vendor: CONFLICT resolved in docs — single brand only confirmed by client-followup (rough-notes "maybe marketplace" was exploratory, overridden before question loop)
- Budget: LOW — no figures available; gated on NDA + CFO call

## Source Docs
- rough-notes.md — Initial discovery call notes (Vikram + Neha): core problem, Instagram workflow, Diwali deadline, Shopify suggestion, budget not discussed
- product-brief.txt — Internal brief draft: feature list, Shopify preference, Phase 2 mobile, Q3 soft launch (superseded by client-followup on multiple points)
- dev-team-email.md — Arjun Mehta's tech assessment: React+Node.js recommendation, React Native rationale, COD and multi-vendor architectural concerns
- client-followup.txt — Neha's authoritative follow-up: decisions on multi-vendor, mobile, loyalty rules, COD volume, timeline, budget NDA requirement
