# Architecture — DecorConnect
# Generated: 2026-06-08 | Reviewed by: Shubham
# Source: mvp-scope.md (Time-boxed framing, 9 features in scope)

## Client Summary
DecorConnect is a two-sided mobile marketplace connecting homeowners in Jaipur with
verified interior designers. The build runs across 4 two-week sprints: the first
establishes accounts and admin approval, the second onboards designers and launches
the inspiration browser, the third brings customer search and real-time chat, and the
fourth adds consultation booking with payments and go-live readiness. The primary
schedule risk is Sprint 3 — chat and customer-side features ramp up simultaneously;
if designer onboarding slips in Sprint 2, Sprint 3 will compress. The platform targets
10 verified Jaipur designers and 25 paid consultation bookings within the first 30 days
of launch.

## Tech Stack
| Layer | Decision | Status |
|---|---|---|
| Mobile | Flutter (iOS + Android) | ✓ Confirmed |
| Backend | Node.js + Express | ✓ Confirmed |
| Database (core) | PostgreSQL | ✓ Confirmed |
| Database (chat) | Firebase Firestore | ✓ Confirmed |
| Auth | Firebase Auth (Phone/OTP) | ✓ Confirmed |
| Media Storage | Azure Blob Storage | ✓ Confirmed |
| Push Notifications | Firebase FCM | ✓ Confirmed |
| Payment Gateway | Razorpay | ✓ Confirmed |
| Analytics | Firebase Analytics | ✓ Confirmed |
| Infra | Azure App Service | ✓ Confirmed |
| Admin Panel | React (web) | ✓ Confirmed |

## Components

### Ideas/Inspiration Section
- InspirationScreen (Flutter) — category browse, image slider, dimension toggle
- InspirationAPI (Node.js) — /ideas/categories, /ideas/by-category
- *(uses MediaService — shared)*

### Designer Profiles
- DesignerProfileScreen (Flutter) — portfolio slider, plans, booking CTA
- DesignerProfileAPI (Node.js) — /designers/:id, /designers/:id/portfolio
- *(uses MediaService — shared)*

### Customer App
- CustomerDashboardScreen (Flutter) — nearby designers, top performers, social proof
- ExploreScreen (Flutter) — search + filters (city/price/type/budget)
- CompareScreen (Flutter) — side-by-side 2–3 designer comparison
- CustomerAPI (Node.js) — /designers/nearby, /designers/top, /designers/search,
  /designers/compare
- *(uses LocationService — shared)*

### Designer App
- OnboardingFlow (Flutter) — 4-step wizard: About → Skills → Experience → Documents
- DesignerDashboardScreen (Flutter) — profile views/saves analytics
- ProfileSetupScreen (Flutter) — portfolio, plans, consultation types
- SubscriptionScreen (Flutter) — tier management, auto-pay, cancel/resume
- SubmitIdeaScreen (Flutter) — idea card creation: images, description, tags
- DesignerAppAPI (Node.js) — /designer/onboard, /designer/profile,
  /designer/subscription, /designer/ideas, /designer/analytics

### Chat Support
- ChatScreen (Flutter) — pre-chat questions, attach designs, block/delete
- ChatService (Firebase Firestore) — real-time message sync
- ChatAPI (Node.js) — /chat/request, /chat/accept, /chat/block (metadata only)

### Push Notifications
- NotificationService (Firebase FCM) — chat, booking, approval triggers
- NotificationAPI (Node.js) — /notifications/register-device, /notifications/send

### In-app Payments
- BookingScreen (Flutter) — consultation type + slot selection + payment
- PaymentService (Node.js + Razorpay) — payment initiation + webhook handling
- BookingDB (PostgreSQL) — bookings, consultation slots, payment records

### Admin Panel
- AdminWeb (React) — designer approval queue, platform management
- AdminAPI (Node.js) — /admin/designers/pending, /admin/approve, /admin/reject

### Analytics
- AnalyticsService (Firebase Analytics) — event tracking across both apps
- DesignerAnalyticsAPI (Node.js) — /designer/analytics/views-saves

### Shared Components
- **AuthService** — Firebase Auth Phone/OTP; used by Customer App + Designer App
- **MediaService** — Azure Blob upload + CDN; used by Designer Profiles, Ideas, Chat
- **LocationService** — geolocation for nearby ranking; used by Customer Dashboard
  + Explore

## Data Model Hints

### Core (PostgreSQL)
- `users` — user_id, type (customer|designer|admin), phone, otp_verified, created_at
- `designer_profiles` — designer_id, name, company_type (freelancer|company), city,
  bio, license_url, subscription_tier, approval_status
- `portfolio_items` — item_id, designer_id, title, description, media_urls[], tags[]
- `consultation_slots` — slot_id, designer_id, day_of_week, start_time, end_time,
  type (office|video|home), charge
- `bookings` — booking_id, customer_id, designer_id, slot_id, payment_id, amount,
  status, created_at
- `ideas` — idea_id, designer_id, title, description, media_urls[], tags[], category
- `chat_threads` — thread_id, customer_id, designer_id, status
  (pending|active|blocked)
- `subscriptions` — subscription_id, designer_id, tier (free|basic|premium),
  start_date, end_date, auto_pay, status

### Chat (Firebase Firestore)
- `chat_messages/{thread_id}/messages` — sender_id, content, attachments[],
  timestamp, type (text|design_attachment)

## Integration Points
| System | Approach | Risk | Open Questions |
|---|---|---|---|
| Firebase Auth (Phone) | OTP via Firebase Auth SDK | LOW | None |
| Razorpay | Flutter SDK + Node.js webhook | LOW | None |
| Firebase Firestore | FlutterFire plugin, real-time chat | LOW | None |
| Firebase FCM | FlutterFire plugin, server triggers | LOW | None |
| Azure Blob Storage | Node.js upload endpoint + CDN URLs | LOW | None |
| Firebase Analytics | FlutterFire plugin, event tracking | LOW | None |

## Build Order
1. AuthService + PostgreSQL schema + Firebase project setup — foundation for all
2. MediaService (Azure Blob) — needed by profiles, ideas, chat attachments
3. AdminAPI + AdminWeb — designer approval gate before any designer goes live
4. OnboardingFlow + DesignerAppAPI — supply side before demand side
5. InspirationAPI + InspirationScreen — free-tier value, early launch hook
6. CustomerAPI + LocationService + ExploreScreen + CompareScreen +
   CustomerDashboardScreen — demand side
7. DesignerProfileScreen + ProfileSetupScreen + SubscriptionScreen — profile +
   billing
8. ChatService + ChatScreen + ChatAPI + NotificationService — engagement layer
9. BookingScreen + PaymentService + BookingDB — consultation revenue
10. DesignerDashboardScreen + AnalyticsService + SubmitIdeaScreen — analytics +
    ideas publishing
11. Full QA sweep + UAT + go-live prep

## Sprint Mapping
**Team:** 2 Flutter (5y + 2y), 2 BE Node.js (6y + 3y), 1 QA (5y)
**Timeline:** 40 working days — 4 × 2-week sprints

| Sprint | Work | Owner |
|---|---|---|
| 1 (D1–10) | AuthService + PostgreSQL schema + Firebase setup | BE-senior |
| | MediaService + Azure Blob setup | BE-junior |
| | AdminAPI + AdminWeb | BE-senior + Flutter-senior |
| | QA: test environment setup | QA |
| 2 (D11–20) | OnboardingFlow + DesignerAppAPI | Flutter-senior + BE-junior |
| | InspirationAPI + InspirationScreen | Flutter-junior + BE-senior |
| | NotificationService + NotificationAPI setup | BE-junior |
| | QA: Auth + Admin approval flow | QA |
| 3 (D21–30) | ExploreScreen + CompareScreen + CustomerDashboardScreen + CustomerAPI + LocationService | Flutter-senior + BE-senior |
| | DesignerProfileScreen + ProfileSetupScreen + SubscriptionScreen | Flutter-junior + BE-junior |
| | ChatService + ChatScreen + ChatAPI | Flutter-senior + BE-senior |
| | QA: Designer onboarding + Inspiration | QA |
| 4 (D31–40) | BookingScreen + PaymentService (Razorpay) + BookingDB | Flutter-senior + BE-senior |
| | AnalyticsService + DesignerDashboardScreen + SubmitIdeaScreen | Flutter-junior + BE-junior |
| | Push notification end-to-end integration | BE-junior |
| | Full QA sweep + UAT + go-live prep | QA |

## Effort Signals
| Feature | Size | Rationale |
|---|---|---|
| Ideas/Inspiration section | M | 2 components + content seeding; no integration risk |
| Designer Profiles | M | 3 components; reuses MediaService |
| Customer App (dashboard, explore, compare) | L | 5 components; LocationService; compare logic |
| Designer App (onboarding, profile, subscription, ideas, analytics) | L | 7 components; subscription billing; admin approval dependency |
| Chat Support | L | Dual-tech (Firestore + Node.js); attachment flow; block/delete |
| Push Notifications | S | FCM + trigger hooks; additive |
| In-app Payments | M | Razorpay SDK + webhooks; booking slot logic |
| Admin Panel | M | React web + approval API; contained scope |
| Analytics | S | Firebase Analytics + designer API; additive |
| AuthService (shared) | S | Firebase Auth Phone; well-understood pattern |
| MediaService (shared) | S | Azure Blob + upload endpoint; standard pattern |

## Open Questions
1. Azure datacenter region not confirmed — India South or Central India?
   Blocks: Azure App Service provisioning
2. Apple Developer + Google Play accounts — ready for publishing?
   Blocks: Sprint 4 go-live prep

## STRAWMAN Summary
All tentative decisions — challenge before Sprint 1:
- [STRAWMAN] Sprint 3 parallel BE + Flutter tracks — assumes designer onboarding
  completes cleanly by Day 20; if it slips, Sprint 3 chat + customer tracks compress

## Confidence Notes
- Budget: WARN — internal product, not specified; no cost ceiling defined for
  Azure + Firebase usage
- Timeline: WARN — 40 working days is consultant estimate; no formal milestone
  sign-off yet

## Source Artifacts
- mvp-scope.md — time-boxed MVP, 9 Phase 1 features, 40-day constraint,
  Jaipur launch, 2 primary users
