# Sprint Plan — DecorConnect
**Prepared by:** fiftyfive technologies
**Date:** 2026-06-08
**Source:** arch.md Sprint Mapping

---

## Team
- 1 × Senior Flutter Mobile Engineer (5 years)
- 1 × Flutter Mobile Engineer (2 years)
- 1 × Senior Backend Engineer (6 years)
- 1 × Backend Engineer (3 years)
- 1 × QA Engineer (5 years)

---

## Sprint Calendar

```gantt
    title Sprint Plan — DecorConnect
    dateFormat  YYYY-MM-DD
    section BE Senior
        Auth + DB Schema + Firebase Setup          : 2026-06-22, 2026-07-03
        InspirationAPI                             : 2026-07-06, 2026-07-17
        CustomerAPI + ChatAPI                      : 2026-07-20, 2026-07-31
        PaymentService (Razorpay)                  : 2026-08-03, 2026-08-14
    section BE Junior
        MediaService (Azure Blob)                  : 2026-06-22, 2026-07-03
        DesignerAppAPI + NotificationService       : 2026-07-06, 2026-07-17
        DesignerProfileAPI + SubscriptionAPI       : 2026-07-20, 2026-07-31
        Push Notification Integration              : 2026-08-03, 2026-08-14
    section Flutter Senior
        AdminWeb (React)                           : 2026-06-22, 2026-07-03
        OnboardingFlow                             : 2026-07-06, 2026-07-17
        ExploreScreen + CompareScreen + ChatScreen : 2026-07-20, 2026-07-31
        BookingScreen                              : 2026-08-03, 2026-08-14
    section Flutter Junior
        InspirationScreen                          : 2026-07-06, 2026-07-17
        DesignerProfileScreen + ProfileSetupScreen + SubscriptionScreen : 2026-07-20, 2026-07-31
        DesignerDashboardScreen + SubmitIdeaScreen : 2026-08-03, 2026-08-14
    section QA
        Test Environment Setup                     : 2026-06-22, 2026-07-03
        Auth + Admin Approval Flow                 : 2026-07-06, 2026-07-17
        Designer Onboarding + Inspiration          : 2026-07-20, 2026-07-31
        Full QA Sweep + UAT + Go-Live Prep         : 2026-08-03, 2026-08-14
```

---

## Sprint Breakdown

| Sprint | Dates | Deliverables | Owner |
|---|---|---|---|
| 1 | 2026-06-22 → 2026-07-03 | AuthService + PostgreSQL schema + Firebase setup | BE-senior |
| | | MediaService + Azure Blob setup | BE-junior |
| | | AdminAPI + AdminWeb | BE-senior + Flutter-senior |
| | | QA: test environment setup | QA |
| 2 | 2026-07-06 → 2026-07-17 | OnboardingFlow + DesignerAppAPI | Flutter-senior + BE-junior |
| | | InspirationAPI + InspirationScreen | Flutter-junior + BE-senior |
| | | NotificationService + NotificationAPI setup | BE-junior |
| | | QA: Auth + Admin approval flow | QA |
| 3 | 2026-07-20 → 2026-07-31 | ExploreScreen + CompareScreen + CustomerDashboardScreen + CustomerAPI + LocationService | Flutter-senior + BE-senior |
| | | DesignerProfileScreen + ProfileSetupScreen + SubscriptionScreen | Flutter-junior + BE-junior |
| | | ChatService + ChatScreen + ChatAPI | Flutter-senior + BE-senior |
| | | QA: Designer onboarding + Inspiration section | QA |
| 4 | 2026-08-03 → 2026-08-14 | BookingScreen + PaymentService (Razorpay) + BookingDB | Flutter-senior + BE-senior |
| | | AnalyticsService + DesignerDashboardScreen + SubmitIdeaScreen | Flutter-junior + BE-junior |
| | | Push notification end-to-end integration | BE-junior |
| | | Full QA sweep + UAT + go-live prep | QA |

**Go-live target: 2026-08-14**

---

## Dependency Map

| Component / Deliverable | Must complete before |
|---|---|
| AuthService + PostgreSQL schema | Everything — all components require auth and DB |
| MediaService (Azure Blob) | Designer Profiles, Ideas section, Chat attachments |
| AdminAPI + AdminWeb | Designer Onboarding (designers need approval before going live) |
| OnboardingFlow + DesignerAppAPI | Customer App, Chat (both sides must be registered) |
| InspirationAPI | InspirationScreen |
| CustomerAPI + LocationService | ExploreScreen, CompareScreen, CustomerDashboardScreen |
| DesignerProfileAPI | DesignerProfileScreen, BookingScreen |
| DesignerAppAPI (subscription) | SubscriptionScreen, designer visibility on platform |
| ChatAPI + ChatService | ChatScreen, push notification integration |
| NotificationService | End-to-end push notifications across chat, booking, approval |
| PaymentService + BookingDB | BookingScreen |
| All Sprint 3 features | Full QA sweep (Sprint 4 QA requires all features integrated) |

---

## Risk Flags

| Sprint | STRAWMAN Risk | Impact if Unresolved |
|---|---|---|
| 3 | Sprint 3 parallel BE + Flutter tracks assume designer onboarding is complete by July 17 | If Sprint 2 designer features slip, Sprint 3 customer-facing and chat work will compress, risking incomplete features before Sprint 4 |
