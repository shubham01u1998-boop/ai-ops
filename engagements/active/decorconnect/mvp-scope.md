# MVP Scope — DecorConnect
# Generated: 2026-06-08 | Reviewed by: Shubham
# Framing: Time-boxed — 40 working days, AI-enabled development

## Problem Restatement
Homeowners in India lack a single platform to discover interior design inspiration,
estimate project costs, and connect with verified local designers. Designers —
both freelancers and companies — lack a structured channel to showcase their work,
manage consultation bookings, and acquire clients. DecorConnect addresses both sides
with a two-sided marketplace launching first in Jaipur.

## Users
**Primary — Customers:** Homeowners browsing design inspiration, finding and hiring
designers. Basic browsing without login; chat and consultation require registration.
**Primary — Interior Designers:** Freelancers and companies seeking clients via
subscription-based listing (Free/Basic/Premium). Service fees collected off-platform.
**Secondary — Admin:** Internal 55tech operators managing designer approvals via
web panel.

## MVP Framing
**Approach:** Time-boxed
**Rationale:** Internal product with a defined delivery target; scope must fit
within 40 working days of AI-enabled development.
**Constraint driving scope:** 40 working days from kick-off

## Scope: In
| Feature | Description | Confidence | Rationale |
|---|---|---|---|
| Ideas/Inspiration section | 30+ room categories, image sliders, dimension adjustment | HIGH | Core free-tier value, drives customer acquisition |
| Designer Profiles | Booking, portfolio (max 20), ratings, subscription plans, hire types, verified badge | HIGH | Supply-side showcase — central to marketplace value |
| Customer App | OTP auth, dashboard, explore with filters, estimation & compare, accounts | HIGH | Primary user entry point |
| Designer App | OTP auth, 4-step onboarding, profile setup, subscription mgmt, submit ideas, profile analytics | HIGH | Supply-side onboarding and management |
| Chat Support | Request-based; pre-chat screening questions; attach designs; block/delete | HIGH | Primary engagement channel |
| Push Notifications | New chat, booking confirmation, designer approval status | HIGH | Required for chat + booking flows |
| In-app Payments | Consultation bookings only; service fees off-platform | HIGH | Revenue-enabling, clearly scoped |
| Admin Panel | Web-based; designer approval workflow, platform management | MED | Gate for supply-side quality control |
| Analytics | Audience capture — user behavior, retention signals for product iteration | MED | Success metric dependency |

## Scope: Out
| Feature | Why Out | Deferred to |
|---|---|---|
| Cost Planner + AI Designer Matching | AI complexity; beyond 40-day window | Phase 2 |
| AI 3D Modelling | High AI/infra complexity; paid pro feature | Phase 2 |
| AI Expert Assistance | Requires trained model; post-MVP | Phase 2 |
| Vendor Marketplace | Separate supply-side onboarding; Phase 3 vision | Phase 3 |

## Key User Journeys
1. Customer → browses inspiration by room category → discovers styles and saves ideas
   [Ideas/Inspiration section]
2. Customer → searches designers by city/filters → views profile → books consultation
   (in-app payment) → chats with designer
   [Customer App, Designer Profiles, In-app Payments, Chat, Push Notifications]
3. Designer → completes 4-step onboarding → admin approves → sets up profile +
   subscription → receives and responds to chat requests
   [Designer App, Admin Panel, Chat, Push Notifications]
4. Designer → submits design idea publicly → customers discover via Ideas section
   → customer views designer profile
   [Designer App, Ideas section, Designer Profiles]

## Success Metrics
- ≥10 designers onboarded in Jaipur within 30 days of launch
- ≥25 consultation bookings within 30 days of launch
- Analytics instrumented at launch — baseline data captured for Phase 2 decisions

## Constraints
- Timeline: 40 working days (AI-enabled development)
- Budget: internal product — not specified
- Tech: iOS + Android, cross-platform; framework not yet decided
- Geography: India; Phase 1 city — Jaipur
- Other: Admin approval gates all designer visibility; service payments off-platform

## Assumptions
- AI-enabled development reduces build time significantly vs traditional approach
- Platform launches in Jaipur only for Phase 1; multi-city is Phase 2+
- Service payments remain off-platform for Phase 1 (no escrow/transaction fee model)
- Designer subscription billing is handled in-app

## Open Questions
1. Cross-platform framework — Flutter vs React Native? — not specified
2. Payment gateway preference — Razorpay, Cashfree, etc.? — not discussed
3. Push notification service — Firebase FCM or other? — not discussed
4. Analytics platform — Firebase Analytics, Mixpanel, or custom? — not discussed
5. Sprint/milestone breakdown — how are 40 days structured? — not discussed

## Effort Signals
⚠ Deferred to ARCH_PROPOSER — sizing without architecture context produces noise,
not signal. ARCH_PROPOSER will add sizing once tech stack and build order are determined.

## Confidence Notes
- Timeline: WARN — 40 working days stated by consultant; no formal milestone
  breakdown yet
- Tech: LOW — cross-platform confirmed but framework and backend stack unspecified
- Analytics: MED — added during MVP_SYNTHESIZER session, not in original doc

## Source Artifacts
- discovery.md — two-sided interior design marketplace concept, 3-phase roadmap,
  Phase 1 feature set, monetization model, constraints
