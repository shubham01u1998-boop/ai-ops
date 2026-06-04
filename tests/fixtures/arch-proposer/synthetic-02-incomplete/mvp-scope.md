# MVP Scope — StyleMart
# Generated: 2026-06-04 | Reviewed by: (consultant name not provided)
# NOTE: Deliberately incomplete fixture for testing ARCH_PROPOSER Readiness Gate.
# Expected gate result: 3 BLOCKs — missing Budget, missing Timeline, unresolved CONFLICT

## Problem Restatement
StyleMart's fashion retail platform needs a recommendation engine and checkout redesign
to reduce cart abandonment from 68% to under 30%.

## Users
**Primary:** Online shoppers — browse and purchase fashion items
**Secondary:** StyleMart merchandising team — manage product catalog

## MVP Framing
**Approach:** Value-first
**Rationale:** Conversion rate is the primary metric; highest-value features first.
**Constraint driving scope:** Conversion improvement before loyalty/social features.

## Scope: In
| Feature | Description | Confidence | Rationale |
|---|---|---|---|
| Recommendation Engine | ML-based product recommendations on PDP and cart | HIGH | Directly addresses abandonment |
| Checkout Redesign | Simplified 2-step checkout with guest option | HIGH | Removes friction at highest drop-off point |

## Scope: Out
| Feature | Why Out | Deferred to |
|---|---|---|
| Loyalty Program | Lower immediate conversion impact | Phase 2 |
| Social Sharing | No abandonment impact | Phase 2 |

## Key User Journeys
1. Shopper → views product → sees personalised recommendations → adds to cart — [Recommendation Engine]
2. Shopper → reaches checkout → completes in 2 steps as guest → order confirmed — [Checkout Redesign]

## Success Metrics
- Cart abandonment rate: 68% → <30% within 90 days of go-live

## Constraints
- Tech: CONFLICT — backend team prefers Python/FastAPI; CTO wants Node.js for stack consistency. Not resolved.
- Other: Must integrate with existing Shopify storefront via API

## Assumptions
- Shopify API supports the required recommendation injection points
- ML model can be trained on existing 6-month purchase history

## Open Questions
1. Backend framework — CONFLICT: Python/FastAPI vs Node.js (see Confidence Notes)
2. Recommendation model: collaborative filtering vs content-based — not decided

## Effort Signals
⚠ Deferred to ARCH_PROPOSER

## Confidence Notes
- Backend: CONFLICT — Python/FastAPI (backend team) vs Node.js (CTO). Not resolved.
- Recommendation model: MED — model type not decided; affects build complexity significantly

## Source Artifacts
- discovery.md — StyleMart: 2 features in scope, cart abandonment core problem, backend CONFLICT unresolved
