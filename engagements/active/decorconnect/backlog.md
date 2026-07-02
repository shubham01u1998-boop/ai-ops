# Backlog — DecorConnect
# Generated: 2026-06-08 | BACKLOG_GENERATOR V1.4
# Status: DRAFT — pending Odoo creation

---

## Project Config

| Field | Value |
|---|---|
| Project name | DecorConnect |
| Odoo project ID | — (assigned when pushed to Odoo) |

### Stages

| Stage | Semantic role |
|---|---|
| Backlog | Blocked items, pre-conditions, STRAWMAN tickets |
| To Do | Newly created tickets (regular + bugs) |
| In Progress | Active development |
| Testing | QA / testing phase |
| Done | Completed |

---

## Team Mapping

| Role | arch.md label | Odoo user ID |
|---|---|---|
| BE-senior | BE-senior (6y Node.js) | — |
| BE-junior | BE-junior (3y Node.js) | — |
| Flutter-senior | Flutter-senior (5y) | — |
| Flutter-junior | Flutter-junior (2y) | — |
| QA | QA (5y) | — |

> Fill in Odoo user IDs before pushing. Run `list_metadata resource=users` to get valid IDs.

---

## Sprint Tags

| Sprint | Date range | Odoo tag ID |
|---|---|---|
| Sprint 1 | 2026-06-22 → 2026-07-03 | — |
| Sprint 2 | 2026-07-06 → 2026-07-17 | — |
| Sprint 3 | 2026-07-20 → 2026-07-31 | — |
| Sprint 4 | 2026-08-03 → 2026-08-14 | — |

---

## Parent Tickets — 11

---

### T01 · [Sprint 1] AuthService + PostgreSQL schema + Firebase project setup

| Field | Value |
|---|---|
| Sprint | Sprint 1 |
| Type | Backend |
| Stage | To Do |
| Assignee role | BE-senior |
| Deadline | 2026-07-03 |
| Priority | Normal |
| Subtasks | AuthService (Firebase Auth), PostgreSQL schema, Firebase project setup |

**Description:**

## Scope
Foundation layer for all services: Phone/OTP authentication via Firebase Auth, core PostgreSQL schema initialisation, and Firebase project provisioning (Auth, Firestore, FCM, Analytics). Every subsequent Build Order item depends on this ticket.

## Interfaces
- Firebase Auth SDK — input: Firebase project config; output: configured Auth instance used by Customer App + Designer App
- PostgreSQL schema migration — input: migration scripts; output: 8 tables (users, designer_profiles, portfolio_items, consultation_slots, bookings, ideas, chat_threads, subscriptions)
- Firebase project setup — input: service account credentials; output: provisioned project with Auth, Firestore, FCM, Analytics enabled

## Key Business Rules
- Only phone numbers with OTP verification create valid user sessions
- `users.type` must be one of: customer | designer | admin
- Schema migrations are version-controlled and idempotent

## Acceptance Criteria
- [ ] Firebase Auth Phone/OTP flow issues and verifies OTP, returning a valid session token
- [ ] All 8 PostgreSQL tables created with correct columns, types, and foreign keys
- [ ] Firebase project provisioned with Auth, Firestore, FCM, Analytics enabled
- [ ] Node.js backend verifies Firebase Auth ID tokens successfully
- [ ] Database connections pool correctly under load

## Edge Cases
- OTP SMS delivery failure — retry logic and error response defined
- Duplicate phone number registration — conflict handled at DB level (unique constraint on `users.phone`)
- Firebase project quota limits — baseline quotas reviewed before Sprint 1 begins

## Open Questions
- Azure datacenter region not confirmed (India South or Central India?) — blocks Azure App Service provisioning

---

### T02 · [Sprint 1] MediaService (Azure Blob)

| Field | Value |
|---|---|
| Sprint | Sprint 1 |
| Type | Backend |
| Stage | To Do |
| Assignee role | BE-junior |
| Deadline | 2026-07-03 |
| Priority | Normal |
| Subtasks | MediaService (Azure Blob) |

**Description:**

## Scope
Shared media upload and delivery service. Accepts image/video uploads from any client (Designer Profiles, Inspiration ideas, Chat attachments) and returns CDN-accessible URLs. Standalone Node.js upload endpoint backed by Azure Blob Storage. Used by all subsequent Build Order items that handle media.

## API Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| POST | /media/upload | Upload image or video; returns CDN URL |
| DELETE | /media/:blob_id | Remove uploaded media (admin or uploader only) |

## Key Business Rules
- Accepted types: image/jpeg, image/png, image/webp, video/mp4 — all others rejected with 400
- Maximum file size: 10 MB per upload
- Blob URLs are permanent CDN links; clients store the URL, not the blob ID
- Delete requires authenticated user ID matching the uploader, or admin role

## Acceptance Criteria
- [ ] POST /media/upload returns a valid CDN URL for accepted file types
- [ ] Oversized or invalid file types return 400 with descriptive error message
- [ ] Blob accessible via CDN URL within 5 seconds of upload
- [ ] DELETE /media/:blob_id removes blob from Azure Blob Storage and returns 204

## Edge Cases
- Upload interrupted mid-stream — partial blob cleaned up, client receives error
- Azure Blob Storage rate limit hit — returns 503 with retry-after header
- Delete called for non-existent blob — returns 404, no error logged

## Open Questions
- Azure datacenter region not confirmed (India South or Central India?) — blocks Azure Blob Storage provisioning

---

### T03 · [Sprint 1] AdminAPI + AdminWeb

| Field | Value |
|---|---|
| Sprint | Sprint 1 |
| Type | Backend [type inferred — verify] |
| Stage | To Do |
| Assignee role | BE-senior |
| Deadline | 2026-07-03 |
| Priority | Normal |
| Subtasks | AdminAPI (Node.js), AdminWeb (React) |

**Description:**

## Scope
Designer approval gate. Web-based admin panel (React) backed by a Node.js API allowing the 55tech team to review pending designer applications and approve or reject them. Triggers push notifications to the designer on approval/rejection. Required before any designer can go live on the platform.

## API Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| GET | /admin/designers/pending | List all pending designer applications |
| POST | /admin/approve/:designer_id | Approve designer; sets approval_status → approved |
| POST | /admin/reject/:designer_id | Reject with reason; sets approval_status → rejected |

## Screens & Components
- AdminWeb (React) — designer approval queue with application details (documents, portfolio, bio), approve/reject actions, reason input on rejection

## Key Business Rules
- Only users with `admin` type can call /admin/* endpoints — all others receive 403
- Approved designers become visible on the platform immediately
- Rejection reason stored in designer_profiles and surfaced to designer via push notification
- Approve/reject actions are idempotent — acting on an already-approved designer is a no-op

## Acceptance Criteria
- [ ] Admin user can log in and view the pending designer queue in AdminWeb
- [ ] Approving a designer updates approval_status to approved and triggers push notification
- [ ] Rejecting a designer stores reason and triggers push notification
- [ ] Non-admin users receive 403 on all /admin/* endpoints
- [ ] AdminWeb is deployable as a standalone React app

## Edge Cases
- Designer applies twice — duplicate application prevented at DB level
- Admin approves already-approved designer — idempotent, no duplicate notification
- Notification failure during approval — approval committed; notification retried async

## Open Questions
(none)

---

### T04 · [Sprint 2] OnboardingFlow + DesignerAppAPI

| Field | Value |
|---|---|
| Sprint | Sprint 2 |
| Type | Frontend |
| Stage | To Do |
| Assignee role | Flutter-senior |
| Deadline | 2026-07-17 |
| Priority | Normal |
| Subtasks | OnboardingFlow (Flutter), DesignerAppAPI (Node.js) |
| Shared (reuse) | AuthService — built T01 · Sprint 1 |
| Shared (reuse) | MediaService — built T02 · Sprint 1 |

**Description:**

## Scope
Supply-side onboarding for interior designers. A 4-step Flutter wizard collects all designer information and submits for admin approval. DesignerAppAPI handles all designer-side data operations: profile management, subscription, idea submission, and designer analytics.

## Screens & Components
- OnboardingFlow (Flutter) — 4-step wizard: Step 1 (About: name, company type, city, bio), Step 2 (Skills: specialisations, room types), Step 3 (Experience: years, portfolio items — min 1, max 20), Step 4 (Documents: license upload, verification photo). Wizard state persisted locally — resumable if app is closed.

## API Endpoints (DesignerAppAPI)
| Method | Path | Purpose |
|--------|------|---------|
| POST | /designer/onboard | Submit completed onboarding application |
| GET/PUT | /designer/profile | Get or update designer profile |
| GET/PUT | /designer/subscription | Get or manage subscription tier |
| POST | /designer/ideas | Submit a new inspiration idea |
| GET | /designer/analytics | Get profile views and saves |

## Key Business Rules
- Designer cannot go live until admin approves application (approval_status = approved)
- All 4 steps must be complete before submission — no partial submits
- Portfolio items at onboarding: minimum 1, maximum 20
- Designer subscription tier defaults to Free on approval

## Acceptance Criteria
- [ ] Designer completes all 4 steps and submits — designer_profiles record created with approval_status = pending
- [ ] Incomplete submission blocked with inline validation messages
- [ ] Submitted application appears in AdminWeb pending queue
- [ ] Designer receives push notification on approval or rejection
- [ ] App closed mid-wizard — state persisted and wizard resumes from last completed step

## Edge Cases
- Duplicate phone number — caught at AuthService level before onboarding begins
- Document upload failure during Step 4 — retry allowed without resetting prior steps
- Designer submits then reopens wizard — application locked once submitted; no re-submission

## Open Questions
(none)

---

### T05 · [Sprint 2] InspirationAPI + InspirationScreen

| Field | Value |
|---|---|
| Sprint | Sprint 2 |
| Type | Frontend |
| Stage | To Do |
| Assignee role | Flutter-junior |
| Deadline | 2026-07-17 |
| Priority | Normal |
| Subtasks | InspirationAPI (Node.js), InspirationScreen (Flutter) |
| Shared (reuse) | MediaService — built T02 · Sprint 1 |

**Description:**

## Scope
Free-tier value proposition — browsable library of interior design ideas available without registration. InspirationScreen provides a category-based gallery with room dimension toggles. InspirationAPI serves idea data by category. Primary hook for new visitors before they register.

## Screens & Components
- InspirationScreen (Flutter) — category grid (30+ room categories), image slider per category, adjustable room dimension toggle (e.g. 10×12 ft). Accessible without login.

## API Endpoints (InspirationAPI)
| Method | Path | Purpose |
|--------|------|---------|
| GET | /ideas/categories | List all available room categories |
| GET | /ideas/by-category | Paginated idea cards for a given category |

## Key Business Rules
- InspirationScreen is accessible without authentication — no login gate
- Ideas submitted by designers surface here after admin approval
- Category list and idea cards are read-only — no user interaction beyond browsing
- Images served via MediaService CDN URLs

## Acceptance Criteria
- [ ] InspirationScreen loads category grid without requiring login
- [ ] Selecting a category displays paginated image gallery for that category
- [ ] Room dimension toggle updates display correctly
- [ ] /ideas/categories returns all available categories
- [ ] /ideas/by-category returns paginated idea cards with valid CDN media URLs

## Edge Cases
- No ideas in a category yet — empty state shown with clear user prompt
- Image load failure — placeholder shown, no crash
- Slow network — loading skeleton shown while ideas fetch

## Open Questions
(none)

---

### T06 · [Sprint 3] CustomerAPI + LocationService + ExploreScreen + CompareScreen + CustomerDashboardScreen

| Field | Value |
|---|---|
| Sprint | Sprint 3 |
| Type | Frontend |
| Stage | To Do |
| Assignee role | Flutter-senior |
| Deadline | 2026-07-31 |
| Priority | Normal |
| Subtasks | CustomerAPI (Node.js), LocationService (Flutter), ExploreScreen (Flutter), CompareScreen (Flutter), CustomerDashboardScreen (Flutter) |
| Shared (reuse) | AuthService — built T01 · Sprint 1 |

**Description:**

## Scope
Demand side of the marketplace — the full customer discovery experience. CustomerDashboardScreen shows personalised nearby and top-rated designers. ExploreScreen provides filtered search. CompareScreen enables side-by-side comparison of 2–3 designers. LocationService is a shared component first built in this ticket.

## Screens & Components
- CustomerDashboardScreen (Flutter) — personalised home: nearby designers (location-ranked), top performers (rating-ranked), social proof stats. Requires login.
- ExploreScreen (Flutter) — search bar + filters: city, price range, hire type (freelancer/company), budget. Infinite scroll results.
- CompareScreen (Flutter) — side-by-side comparison of 2–3 selected designers: portfolio thumbnails, hire rates, ratings, consultation availability.
- LocationService (Flutter, **shared — first built here**) — device geolocation; provides lat/lng to CustomerAPI for proximity ranking. Shared by CustomerDashboardScreen + ExploreScreen.

## API Endpoints (CustomerAPI)
| Method | Path | Purpose |
|--------|------|---------|
| GET | /designers/nearby | Designers sorted by proximity to lat/lng |
| GET | /designers/top | Top-rated designers |
| GET | /designers/search | Filtered designer search (city/price/type/budget) |
| GET | /designers/compare | Side-by-side data for 2–3 designer IDs |

## Key Business Rules
- CustomerDashboardScreen requires authenticated customer session
- Search results show only approved designers (approval_status = approved)
- /designers/nearby uses device lat/lng; falls back to city-based ranking if location permission denied
- CompareScreen allows maximum 3 designers at once

## Acceptance Criteria
- [ ] CustomerDashboardScreen loads nearby and top designers after login
- [ ] ExploreScreen filters return correct results for all filter combinations
- [ ] CompareScreen renders side-by-side for 2–3 selected designers
- [ ] /designers/nearby returns proximity-sorted designers for given lat/lng
- [ ] LocationService requests device permission and returns lat/lng on grant
- [ ] Location permission denied — dashboard falls back to city-based ranking without crash

## Edge Cases
- No designers match search filters — empty state with clear user prompt
- Less than 2 designers selected for compare — compare blocked with inline prompt
- Location permission denied — graceful fallback, no crash or error screen

## Open Questions
(none)

---

### T07 · [Sprint 3] DesignerProfileScreen + ProfileSetupScreen + SubscriptionScreen

| Field | Value |
|---|---|
| Sprint | Sprint 3 |
| Type | Frontend |
| Stage | To Do |
| Assignee role | Flutter-junior |
| Deadline | 2026-07-31 |
| Priority | Normal |
| Subtasks | DesignerProfileScreen (Flutter), DesignerProfileAPI (Node.js), ProfileSetupScreen (Flutter), SubscriptionScreen (Flutter) |
| Shared (reuse) | MediaService — built T02 · Sprint 1 |
| Shared (reuse) | AuthService — built T01 · Sprint 1 |
| Shared (reuse) | DesignerAppAPI — built T04 · Sprint 2 |

**Description:**

## Scope
Designer-facing profile and billing management. DesignerProfileScreen is the customer-visible designer profile (portfolio, plans, booking CTA). ProfileSetupScreen allows designers to manage their own profile and portfolio. SubscriptionScreen handles tier selection and billing lifecycle.

## Screens & Components
- DesignerProfileScreen (Flutter) — customer-visible: portfolio slider (up to 20 items), consultation plan listing and pricing, hire rates, ratings and reviews, booking CTA.
- DesignerProfileAPI (Node.js) — /designers/:id, /designers/:id/portfolio
- ProfileSetupScreen (Flutter) — designer self-service: update bio, add/remove/reorder portfolio items, set consultation types (office/video/home), charges and availability.
- SubscriptionScreen (Flutter) — tier selection (Free / Basic / Premium), auto-pay toggle, cancel and resume subscription lifecycle.

## API Endpoints (DesignerProfileAPI)
| Method | Path | Purpose |
|--------|------|---------|
| GET | /designers/:id | Full designer profile for customer view |
| GET | /designers/:id/portfolio | Paginated portfolio items for a designer |

## Key Business Rules
- DesignerProfileScreen visible to all authenticated customers
- Subscription tier determines portfolio visibility caps (exact caps TBD — product team to define)
- Auto-pay enabled by default at subscription activation; designer must explicitly disable
- Subscription cancellation moves to Free tier at period end — not immediately

## Acceptance Criteria
- [ ] DesignerProfileScreen renders full profile with portfolio slider and booking CTA
- [ ] /designers/:id returns correct designer data
- [ ] ProfileSetupScreen saves portfolio item changes correctly
- [ ] SubscriptionScreen shows current tier and allows upgrade/downgrade/cancel
- [ ] Cancellation sets status to `cancelling`; tier changes at period end, not immediately

## Edge Cases
- Designer has no portfolio items — profile shows empty-portfolio state with prompt to add
- Portfolio image fails to load — placeholder shown, no crash
- Subscription payment failure — designer notified, tier stays current until retry

## Open Questions
(none)

---

### T08 · [Sprint 3] ChatService + ChatScreen + ChatAPI + NotificationService

| Field | Value |
|---|---|
| Sprint | Sprint 3 |
| Type | Backend |
| Stage | To Do |
| Assignee role | Flutter-senior |
| Deadline | 2026-07-31 |
| Priority | Normal |
| Subtasks | ChatService (Firebase Firestore), ChatScreen (Flutter), ChatAPI (Node.js), NotificationService (Firebase FCM), NotificationAPI (Node.js) |
| Shared (reuse) | MediaService — built T02 · Sprint 1 |
| Shared (reuse) | AuthService — built T01 · Sprint 1 |

**Description:**

## Scope
Engagement layer — real-time chat between customers and designers with pre-chat screening, media attachments, and block/delete controls. NotificationService and NotificationAPI complete the push notification infrastructure, handling all push triggers (chat, booking, approval) across both apps.

## API Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| POST | /chat/request | Customer initiates chat with designer |
| POST | /chat/accept | Designer accepts chat request |
| POST | /chat/block | Either party blocks the other |
| POST | /notifications/register-device | Register FCM device token for authenticated user |
| POST | /notifications/send | Send push notification (internal server-to-server) |

## Screens & Components
- ChatScreen (Flutter) — pre-chat screening questions (designer-set), real-time messaging thread, design image attachments, block and delete conversation controls.
- ChatService (Firebase Firestore) — real-time message sync per thread (`chat_messages/{thread_id}/messages`).

## Key Business Rules
- Designer sets screening questions; customer must answer before thread opens
- Only designer can accept or reject a chat request — customer cannot force open a thread
- Block action hides thread and prevents further messages from the blocked party
- All push triggers (chat message, booking confirmation, designer approval) route through /notifications/send

## Acceptance Criteria
- [ ] Customer sends chat request; designer receives push notification
- [ ] Designer accepts; thread active in ChatScreen for both parties with real-time Firestore sync
- [ ] Chat media attachments upload via MediaService and display inline in thread
- [ ] Block action hides thread for both parties immediately
- [ ] Push notification delivered for: new message, booking confirmation, approval status change
- [ ] /notifications/register-device stores FCM token for authenticated user

## Edge Cases
- Designer offline — FCM notification queued, delivered on reconnect
- Message sent while app offline — queued locally, synced to Firestore on reconnect
- Both parties block simultaneously — thread closed cleanly, no error state

## Open Questions
(none)

---

### T09 · [Sprint 4] BookingScreen + PaymentService + BookingDB

| Field | Value |
|---|---|
| Sprint | Sprint 4 |
| Type | Backend |
| Stage | To Do |
| Assignee role | Flutter-senior |
| Deadline | 2026-08-14 |
| Priority | Normal |
| Subtasks | BookingScreen (Flutter), PaymentService (Node.js + Razorpay SDK), BookingDB (PostgreSQL) |

**Description:**

## Scope
Consultation revenue layer. BookingScreen allows customers to select consultation type, choose an available time slot, and pay in-app via Razorpay. PaymentService handles Razorpay order creation, webhook processing, and booking record writes. BookingDB is the PostgreSQL schema for bookings and slot availability.

## API Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| GET | /bookings/:designer_id/slots | List available future consultation slots |
| POST | /bookings/create | Create Razorpay order; return orderId + key to client |
| POST | /bookings/webhook | Razorpay payment.captured webhook handler |

## Screens & Components
- BookingScreen (Flutter) — consultation type selector (office / video / home visit), available slot picker, Razorpay checkout integration, booking confirmation screen.

## Key Business Rules
- Only consultation bookings are paid in-app — all other payments between designers and customers are off-platform
- Booking confirmed only after `payment.captured` webhook — not at order creation
- Slot locked on payment confirmation — no double-booking
- Razorpay webhook signature validated before any state change

## Acceptance Criteria
- [ ] Customer selects slot, pays via Razorpay — booking record created with status = confirmed
- [ ] Webhook signature validation rejects tampered payloads with 400
- [ ] Double-booking same slot returns conflict error to second attempt
- [ ] Booking confirmation push notification sent to both customer and designer
- [ ] /bookings/:designer_id/slots returns only future, unbooked slots

## Edge Cases
- User closes app after payment but before webhook fires — booking stays pending; resolved on next webhook delivery or manual timeout
- Razorpay webhook arrives before order record exists — idempotent write, no duplicate booking
- Slot cancelled by designer after payment — out of scope for MVP; flagged as manual resolution process

## Open Questions
(none)

---

### T10 · [Sprint 4] DesignerDashboardScreen + AnalyticsService + SubmitIdeaScreen

| Field | Value |
|---|---|
| Sprint | Sprint 4 |
| Type | Frontend |
| Stage | To Do |
| Assignee role | Flutter-junior |
| Deadline | 2026-08-14 |
| Priority | Normal |
| Subtasks | DesignerDashboardScreen (Flutter), AnalyticsService (Firebase Analytics), DesignerAnalyticsAPI (Node.js), SubmitIdeaScreen (Flutter) |
| Shared (reuse) | MediaService — built T02 · Sprint 1 |
| Shared (reuse) | AuthService — built T01 · Sprint 1 |
| Shared (reuse) | DesignerAppAPI — built T04 · Sprint 2 |

**Description:**

## Scope
Analytics and content publishing for designers. DesignerDashboardScreen shows profile view and save counts from DesignerAnalyticsAPI. AnalyticsService instruments Firebase Analytics event tracking across both mobile apps. SubmitIdeaScreen lets designers publish new design ideas to the Inspiration section.

## Screens & Components
- DesignerDashboardScreen (Flutter) — profile views count, saves count, trend indicators. Designer role only, post-login.
- AnalyticsService (Firebase Analytics, **shared — first built here**) — FlutterFire event tracking across Customer App + Designer App. Events: designer_profile_view, idea_view, booking_initiated, booking_completed.
- SubmitIdeaScreen (Flutter) — idea card creation: image upload (1–10 images via MediaService), description, tags, category selector. Submitted ideas appear in InspirationScreen.
- DesignerAnalyticsAPI (Node.js) — /designer/analytics/views-saves

## API Endpoints (DesignerAnalyticsAPI)
| Method | Path | Purpose |
|--------|------|---------|
| GET | /designer/analytics/views-saves | Aggregated profile views and saves for requesting designer |

## Key Business Rules
- Analytics events are fire-and-forget — app does not block or crash on Firebase Analytics failure
- DesignerDashboardScreen is only visible to authenticated designer users
- Submitted ideas require at least 1 image
- Idea category must match an existing category from /ideas/categories

## Acceptance Criteria
- [ ] DesignerDashboardScreen shows correct view and save counts from DesignerAnalyticsAPI
- [ ] Firebase Analytics events confirmed firing: designer_profile_view, idea_view, booking_initiated, booking_completed
- [ ] SubmitIdeaScreen uploads images via MediaService and creates idea record via DesignerAppAPI
- [ ] Submitted idea appears in InspirationScreen under the selected category
- [ ] /designer/analytics/views-saves returns aggregated counts for the requesting designer only

## Edge Cases
- Firebase Analytics event failure — app continues silently, event dropped
- Designer submits idea with no images — blocked by inline validation
- Idea submitted with category not in /ideas/categories — DesignerAppAPI returns 400

## Open Questions
(none)

---

### T11 · [Sprint 4] Full QA sweep + UAT + go-live prep

| Field | Value |
|---|---|
| Sprint | Sprint 4 |
| Type | Backend |
| Stage | To Do |
| Assignee role | QA |
| Deadline | 2026-08-14 |
| Priority | Normal |
| Subtasks | (none) |

**Description:**

## Scope
Sprint 4 QA milestone: end-to-end QA sweep across all 10 Build Order items, User Acceptance Testing, and go-live preparation (app store submissions, infra checklist, STRAWMAN resolution confirmation).

## Interfaces
- Input: all Sprint 1–4 features integrated and deployed to staging environment
- Output: QA sign-off report, UAT feedback resolved, go-live checklist completed

## Key Business Rules
- No feature may be marked Done until all its Acceptance Criteria are verified on staging
- STRAWMAN pre-condition ticket must be resolved and signed off before go-live
- App Store (iOS) and Google Play (Android) submissions require production builds from Sprint 4

## Acceptance Criteria
- [ ] All parent ticket Acceptance Criteria verified on staging environment
- [ ] UAT conducted with ≥2 test customers and ≥2 test designers
- [ ] Firebase Analytics events confirmed firing in production build
- [ ] App Store and Google Play submissions prepared and submitted
- [ ] Go-live infra checklist completed (Azure region confirmed, FCM prod keys, Razorpay prod keys)
- [ ] STRAWMAN ticket resolved and signed off before submission

## Edge Cases
- UAT feedback requires code changes — Sprint 4 buffer consumed; track impact against 2026-08-14 deadline
- App Store review rejected — resubmit within Sprint 4 buffer window

## Open Questions
- Apple Developer account available for iOS publishing? — blocks Sprint 4 go-live prep
- Google Play Developer account available for Android publishing? — blocks Sprint 4 go-live prep

---

## STRAWMAN Tickets — 1

---

### S01 · ⚠ STRAWMAN: Verify Sprint 3 parallel BE + Flutter tracks before Sprint 1

| Field | Value |
|---|---|
| Stage | Backlog |
| Deadline | 2026-06-22 (Sprint 1 start) |
| Assignee | (unassigned — triage owner assigns) |
| Priority | High |
| Tags | (none) |

**Description:**

## Decision
Sprint 3 runs parallel Backend and Flutter tracks simultaneously: customer-side features (ExploreScreen, CompareScreen, CustomerDashboardScreen, CustomerAPI, LocationService) and designer profile/billing features (DesignerProfileScreen, DesignerProfileAPI, ProfileSetupScreen, SubscriptionScreen) are built concurrently.

## Why Tentative
This parallelism assumes designer onboarding (Sprint 2: OnboardingFlow + DesignerAppAPI) completes cleanly by Day 20 (2026-07-17). If Sprint 2 slips, Sprint 3 customer-facing and chat work will compress, risking incomplete features before Sprint 4.

## What Resolves This
Confirm Sprint 2 scope is achievable before Sprint 1 ends (by 2026-07-03). Review at Sprint 1 retrospective: is OnboardingFlow + DesignerAppAPI realistic for the Flutter-senior + BE-junior pairing in 10 working days?

## Impact if Wrong
Sprint 3 chat + customer tracks compress — T06, T07, T08 may be incomplete entering Sprint 4. BookingScreen and PaymentService (T09) depend on CustomerAPI and DesignerProfileAPI being complete.

---

## Push to Odoo

To create these tickets in Odoo when MCP is available:

1. Fill in **Odoo user IDs** in the Team Mapping table above — run `list_metadata resource=users` to get valid IDs
2. Run BACKLOG_GENERATOR in this folder — it will detect `backlog.md` and offer push mode
3. BACKLOG_GENERATOR will:
   - `create_project` (DecorConnect, stages: Backlog / To Do / In Progress / Testing / Done)
   - `create_tag` × 4 (Sprint 1 / Sprint 2 / Sprint 3 / Sprint 4)
   - `list_tickets` (duplicate check)
   - `bulk_create_tickets` × 11 parent tickets
   - `add_subtasks` × 10 parent tickets (T11 has no subtasks)
   - `bulk_create_tickets` × 1 STRAWMAN ticket (stage: Backlog, priority: high)
