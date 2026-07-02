# Discovery — DecorConnect
# Generated: 2026-06-08 | Reviewed by: Shubham

## Project Context
DecorConnect is an internal 55tech product — a two-sided interior design marketplace
mobile application targeting the Indian market, launching first in Jaipur. The platform
connects homeowners with verified interior designers (freelancers and companies) through
a structured marketplace with inspiration, discovery, consultation booking, and
communication tools. The product has a defined 3-phase roadmap; Phase 1 is the
buildable MVP.

## Users
**Primary — Customers:** Homeowners browsing design inspiration, estimating costs,
finding and hiring designers. Can explore basic features without login; chat and premium
features require registration (OTP auth).

**Primary — Interior Designers:** Freelancers and companies seeking clients. Pay a
subscription (Free/Basic/Premium) to be listed on the platform. Require admin approval
before going live. Service fees are collected directly from customers outside the platform.

**Secondary — Admin:** Internal 55tech operators. Web-based panel for designer
onboarding approval and platform management.

## Core Problem
Homeowners lack a single platform to discover interior design inspiration, estimate
project costs, and connect with verified local designers. Designers lack a structured
channel to showcase work, manage consultations, and acquire clients. DecorConnect
addresses both sides with a single marketplace.

## Features Mentioned

**Phase 1 — Core Marketplace**
- Ideas/Inspiration section — 30+ room categories, image sliders, dimension adjustment [HIGH]
- Designer Profiles — consultation booking/schedule, portfolio (max 20 projects),
  ratings, subscription plans, hire types (hourly/daily/project), verified license badge [HIGH]
- Chat Support — request-based; designer sets pre-chat screening questions; attach
  designs; block/delete; registered users only [HIGH]
- Customer App — OTP auth, dashboard (nearby + top designers, social proof stats),
  explore with filters (city/price/type/budget), estimation & compare (2–3 designers),
  accounts section [HIGH]
- Designer App — OTP auth, 4-step onboarding with admin approval gate, profile setup,
  subscription management, submit ideas publicly, profile analytics
  (views/saves) [HIGH]
- Admin Panel — web-based, designer approval workflow, platform management [MED]
- In-app payments — consultation bookings only; all other service fees handled
  directly between customer and designer outside the platform [HIGH]
- Push notifications — critical triggers: new chat, booking confirmation,
  designer approval status [HIGH]

**Phase 2 — AI Layer**
- Cost Planner — template-based estimation + AI designer matching with pros/cons
  (paid basic plan) [MED]
- AI 3D Modelling — upload images/videos, generate renovation 3D models across
  Basic/Standard/Premium tiers (paid pro plan) [MED]
- AI Expert Assistance — interior design chatbot, limited on free plan [MED]

**Phase 3 — Vendor Marketplace**
- Vendor onboarding — materials, furniture, carpenter/service providers [LOW]

## Constraints
- Timeline: not specified
- Budget: internal product — not specified
- Tech: iOS + Android, cross-platform mobile; framework not yet decided
- Geography: India; Phase 1 city — Jaipur
- Other: Admin approval required before any designer is visible; in-app payments scoped
  to consultations only; service payments are off-platform

## Open Questions
1. Cross-platform framework — Flutter vs React Native? — not specified in docs
2. Delivery timeline and milestones — not mentioned
3. Payment gateway preference — Razorpay, Cashfree, etc.? — not discussed
4. Push notification service — Firebase FCM or other? — not discussed

## Confidence Notes
- Problem: MED — inferred from feature set, not explicitly stated in doc
- Tech: LOW — cross-platform confirmed but framework and backend stack unspecified
- Timeline: LOW — not mentioned anywhere
- Monetization: CONFLICT resolved — designer subscription (to be listed) +
  customer per-service pricing (off-platform); in-app payments limited to consultations

## Source Docs
- Interior_Design_Platform_Concept.pdf — 11-page concept doc covering all Phase 1/2/3
  features with wireframe sketches and designer pricing examples
