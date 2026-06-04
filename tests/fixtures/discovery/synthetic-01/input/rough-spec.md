# SupplySync — Project Specification Draft
# Prepared by: Internal team
# Date: April 2026

## Overview

SupplySync is a web-based supplier management platform for Prakash Distributors, a mid-size food distribution company. The platform will replace the current Excel + WhatsApp workflow for managing supplier relationships, purchase orders, and contract tracking.

## Problem Statement

Currently, the procurement team manages ~80 suppliers using a combination of Excel spreadsheets, email threads, and WhatsApp groups. This leads to:
- Missed order deadlines because follow-ups happen manually
- No visibility into contract expiry dates until it's too late
- Invoice reconciliation takes 2–3 days per week per procurement manager
- No audit trail for supplier communication

## Proposed Solution

A centralised web platform where:
- Suppliers can be onboarded and their documents stored
- Purchase orders can be raised, tracked, and confirmed
- Contracts are stored with expiry alerts
- Invoices are matched against POs automatically

## Features

1. **Supplier Onboarding** — Supplier fills a form, uploads GST certificate and PAN card, admin approves
2. **Order Management** — Raise PO, track status (Pending → Confirmed → Dispatched → Received)
3. **Contract Management** — Upload contracts, set expiry reminders, version history
4. **Invoice Reconciliation** — Upload invoice, auto-match to PO, flag discrepancies
5. **Notification System** — Email notifications for order confirmations and contract expiry

## Tech Stack

Preferred stack: React (frontend), Node.js + Express (backend), PostgreSQL (database).
The team is open to alternatives if there are strong reasons.

## Timeline

Target delivery: Q3 2026 (July–September 2026).

## Team

Internal development team at fiftyfive-tech. Exact team size TBD.

## Out of Scope (for now)

- Mobile app (website should be mobile-friendly)
- Integration with existing ERP (deferred to Phase 2)
- Supplier-facing portal with login (all supplier interaction via form links)
