# Scope Agreement — DecorConnect
**Client:** 55tech (Internal Product)
**Prepared by:** fiftyfive technologies
**Date:** 2026-06-08
**Status:** Pending client sign-off

---

## 1. Problem Statement
Homeowners in India lack a single platform to discover interior design inspiration,
estimate project costs, and connect with verified local designers. Designers —
both freelancers and companies — lack a structured channel to showcase their work,
manage consultation bookings, and acquire clients. DecorConnect addresses both sides
with a two-sided marketplace launching first in Jaipur.

---

## 2. Users

| Role | Usage |
|---|---|
| Customers (Homeowners) | Browse design inspiration without registration; register to chat, book consultations, and compare designers |
| Interior Designers | Register and subscribe to be listed on the platform; manage profile, portfolio, and consultations; communicate with clients via chat |
| Admin (55tech team) | Review and approve designer applications via a web-based panel before designers go live |

---

## 3. MVP Scope

### In Scope
| Capability | Description |
|---|---|
| Design Inspiration Browser | Browsable library of interior design ideas across 30+ room categories, with image galleries and adjustable room dimensions. Available without registration. |
| Designer Profiles & Listings | Full designer profiles — portfolio (up to 20 projects), consultation availability and pricing, hire rates, ratings, reviews, and license verification badge. |
| Customer App | Mobile app for homeowners: personalised home screen with nearby and top-rated designers, search with filters (city, price, hire type, budget), and side-by-side designer comparison. |
| Designer App | Mobile app for designers: guided sign-up flow with admin approval gate, profile and portfolio management, subscription plan management, and a personal analytics dashboard. |
| In-App Chat | Direct messaging between homeowners and designers. Designers may set screening questions before accepting a chat. Both parties can share files, delete conversations, and block users. |
| Consultation Booking & Payment | Homeowners book paid consultations in-app — choosing consultation type (office, video, or home visit), available time slot, and paying securely via Razorpay. |
| Real-Time Notifications | Push notifications for: new chat messages, consultation booking confirmations, and designer approval status updates. |
| Admin Approval Panel | Web-based management interface for the 55tech team to review and approve designer applications. |
| Product Analytics | In-app usage tracking across both apps to support data-driven improvements from day one. |

### Out of Scope (Phase 2+)
| Capability | Reason |
|---|---|
| AI-Powered Cost Planner & Designer Matching | Requires AI model development — planned Phase 2 |
| AI 3D Renovation Modelling | High-complexity AI feature — Phase 2 paid pro plan |
| AI Interior Design Assistant | Requires trained conversational model — Phase 2 |
| Vendor & Materials Marketplace | Separate supply-side onboarding — Phase 3 |

---

## 4. Key User Journeys
1. Customer → browses inspiration by room category → discovers styles and saves ideas
2. Customer → searches designers by city/filters → views profile → books consultation
   (in-app payment) → chats with designer
3. Designer → completes sign-up → admin approves → sets up profile + subscription →
   receives and responds to chat requests
4. Designer → submits design idea publicly → customers discover via Inspiration section
   → customer views designer profile

---

## 5. Delivery Timeline

```gantt
    title Delivery Timeline — DecorConnect
    dateFormat  YYYY-MM-DD
    section Phase 1 — Foundation & Admin
        Accounts, Media & Admin Approval Panel      : 2026-06-22, 2026-07-03
    section Phase 2 — Designer & Inspiration
        Designer Onboarding & Inspiration Browser   : 2026-07-06, 2026-07-17
    section Phase 3 — Customer & Chat
        Customer App, Search & Real-Time Chat       : 2026-07-20, 2026-07-31
    section Phase 4 — Bookings & Launch
        Payments, Analytics & Go-Live               : 2026-08-03, 2026-08-14
```

**Go-live target: 2026-08-14**

---

## 6. Constraints
- **Budget:** To be confirmed — not specified in scoping documents
- **Infrastructure:** Cloud-hosted on Microsoft Azure; media files stored on Azure
  cloud storage
- **Mobile platform:** iOS and Android (cross-platform)
- **Payment processing:** In-app payments cover consultation bookings only;
  all other payments between homeowners and designers are arranged directly
  outside the platform
- **Geography:** Phase 1 launch city — Jaipur, India
- **Timeline:** 40 working days, kick-off 2026-06-22, go-live 2026-08-14

---

## 7. Assumptions
- AI-enabled development tools are used throughout, expected to significantly reduce
  build time compared to a traditional approach.
- The platform launches exclusively in Jaipur for Phase 1. Multi-city expansion is
  planned for Phase 2.
- Service payments (beyond consultation bookings) are arranged directly between
  homeowners and designers — not processed through the platform.
- Designer subscription billing is managed within the app.
- It is assumed that the cloud infrastructure region (India) is feasible for all
  required services — this will be confirmed before Sprint 1 begins.
- It is assumed that the Sprint 3 parallel development tracks are achievable given
  that designer onboarding is completed on schedule in Sprint 2. This will be
  reviewed at the end of Sprint 2.

---

## 8. Pre-conditions for Project Start

| # | Pre-condition | Owner | Status |
|---|---|---|---|
| 1 | Azure datacenter region confirmed (India South or Central India) | fiftyfive | Open |
| 2 | Apple Developer account available for iOS publishing | 55tech | Open |
| 3 | Google Play Developer account available for Android publishing | 55tech | Open |

---

## 9. Success Metrics
- ≥10 verified interior designers onboarded in Jaipur within 30 days of launch
- ≥25 paid consultation bookings completed within 30 days of launch
- Product analytics instrumented at launch to capture baseline data for Phase 2

---

## 10. Sign-Off

By signing below, both parties confirm that the scope described in this document is
agreed and that work will not commence until pre-conditions in Section 8 are resolved.

| | |
|---|---|
| **55tech** | |
| Name: | __________________ |
| Title: | __________________ |
| Date: | __________________ |
| | |
| **fiftyfive technologies** | |
| Name: | __________________ |
| Title: | __________________ |
| Date: | __________________ |
