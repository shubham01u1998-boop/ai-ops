# Project Proposal — DecorConnect
**Client:** 55tech (Internal Product)
**Prepared by:** fiftyfive technologies
**Date:** 2026-06-08
**Status:** Draft — pending client review

---

## 1. Executive Summary
Homeowners in India currently lack a single destination to browse interior design
inspiration, estimate project costs, and connect with verified local designers. At the
same time, interior designers — whether freelancers or established firms — have no
structured platform to showcase their work, manage client enquiries, and book
consultations. DecorConnect solves both sides of this problem with a two-sided mobile
marketplace, launching first in Jaipur, that makes it easy for homeowners to find the
right designer and for designers to grow their client base.

---

## 2. MVP Scope

### In Scope
| Capability | Description |
|---|---|
| Design Inspiration Browser | A browsable library of interior design ideas across 30+ room categories (kitchen, living room, bedroom, outdoor, and more), with image galleries and adjustable room dimensions. Available to all users without registration. |
| Designer Profiles & Listings | Fully detailed profiles for every designer — portfolio (up to 20 projects), consultation availability, pricing plans, hire rates, ratings, reviews, and license verification. |
| Customer App | A mobile app for homeowners with a personalised home screen showing nearby and top-rated designers, full search and filter (by city, price, hire type, budget), and a side-by-side designer comparison tool. |
| Designer App | A mobile app for designers with a guided sign-up flow, profile management, subscription management, and a personal analytics dashboard (profile views and saves). |
| In-App Chat | Direct messaging between homeowners and designers. Designers set screening questions before accepting a chat request. Both parties can share files, delete conversations, and block users. |
| Consultation Booking & Payment | Homeowners can book paid consultations in-app — choosing type (office, video, or home visit), selecting a slot, and paying securely via Razorpay. |
| Real-Time Notifications | Push notifications for both homeowners and designers: new chat messages, booking confirmations, and designer approval status updates. |
| Admin Approval Panel | A web-based interface for the platform team to review and approve designer applications before they go live. |
| Product Analytics | In-app behaviour tracking across both apps to enable data-driven product improvements from day one. |

### Out of Scope (Phase 2+)
| Capability | Reason |
|---|---|
| AI-Powered Cost Planner & Designer Matching | Requires AI model development — planned for Phase 2 |
| AI 3D Renovation Modelling | High-complexity AI feature — Phase 2 paid pro plan |
| AI Interior Design Assistant | Requires trained conversational model — Phase 2 |
| Vendor & Materials Marketplace | Separate supply-side onboarding — Phase 3 |

---

## 3. Key User Journeys
1. Customer → browses inspiration by room category → discovers styles and saves ideas
2. Customer → searches designers by city/filters → views profile → books consultation
   (in-app payment) → chats with designer
3. Designer → completes sign-up → admin approves → sets up profile + subscription →
   receives and responds to chat requests
4. Designer → submits design idea publicly → customers discover via Inspiration section
   → customer views designer profile

---

## 4. High-Level Solution Overview
DecorConnect is a two-sided mobile marketplace connecting homeowners in Jaipur with
verified interior designers. The build runs across 4 two-week phases: the first
establishes user accounts and the admin approval system, the second onboards designers
and launches the inspiration browser, the third delivers the customer search experience
and real-time chat, and the fourth adds consultation booking with payments and prepares
for go-live. The primary schedule risk is the third phase, where the highest volume of
features are developed simultaneously — if designer onboarding from phase two is not
complete on time, the customer-facing features will compress. The platform targets 10
verified Jaipur designers and 25 paid consultation bookings within the first 30 days
of launch.

---

## 5. Delivery Timeline

```gantt
    title Delivery Timeline — DecorConnect
    dateFormat  YYYY-MM-DD
    section Phase 1 — Foundation & Admin
        Accounts, Media & Admin Approval Panel      : 2026-06-09, 2026-06-20
    section Phase 2 — Designer & Inspiration
        Designer Onboarding & Inspiration Browser   : 2026-06-23, 2026-07-04
    section Phase 3 — Customer & Chat
        Customer App, Search & Real-Time Chat       : 2026-07-07, 2026-07-18
    section Phase 4 — Bookings & Launch
        Payments, Analytics & Go-Live               : 2026-07-21, 2026-08-01
```
*Dates assume kick-off 2026-06-09. Confirm with the project team before distributing.*

---

## 6. Budget
⚠ To be confirmed — not specified in scoping documents.

---

## 7. Risks & Assumptions

### Key Risks
- **Phase 3 delivery risk:** The third build phase has the highest concentration of
  parallel work. If designer onboarding features from Phase 2 are not fully complete
  on time, the customer search, chat, and comparison features in Phase 3 may be
  compressed or delayed.

### Assumptions
- AI-enabled development tools are used throughout, expected to significantly reduce
  build time compared to a traditional approach.
- The platform launches exclusively in Jaipur for Phase 1. Multi-city expansion is
  planned for Phase 2.
- Payment for designer services (beyond consultations) is arranged directly between
  homeowner and designer outside the platform. Only consultation bookings are
  processed in-app.
- Designer subscription billing is managed within the app.

---

## 8. Success Metrics
- ≥10 verified interior designers onboarded in Jaipur within 30 days of launch
- ≥25 paid consultation bookings completed within 30 days of launch
- Product analytics instrumented at launch to capture baseline data for Phase 2

---

## 9. Delivery Team
- 1 × Senior Flutter Mobile Engineer (5 years)
- 1 × Flutter Mobile Engineer (2 years)
- 1 × Senior Backend Engineer (6 years)
- 1 × Backend Engineer (3 years)
- 1 × QA Engineer (5 years)

---

## 10. Sign-Off

By signing below, both parties confirm that this proposal accurately reflects the agreed
scope and that work will commence following the resolution of any outstanding
pre-conditions.

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
