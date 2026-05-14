# PARKING_LOT — TiffinConnect
# VERSION: 1.1 | Created: 2026-05-14 | Maintained by: Claude (auto) + Lead (review)
#
# Claude — session start:
#   1. Auto-cleanup: scan all items, apply TTL rules below, delete expired items silently.
#      After cleanup, show one line: "Auto-cleaned [N] expired items from PARKING_LOT."
#      If N=0: no message.
#   2. Resolved items: delete any line marked ✅ DONE or ❌ DISCARDED immediately, no TTL needed.
#   3. Deferred Requirements expiry warning: if a → item in Section 2 is within 5 days of its
#      30-day TTL, surface it: "Deferred requirement '[title]' expires in [N] days — still relevant? YES / DISCARD"
#   4. After cleanup, if any → items remain: show "PARKING_LOT has [N] unresolved items — review first? YES / SKIP"
#
# TTL rules (measured from item date):
#   Section 1 — Retry Queue:        delete after 7 days
#   Section 2 — Deferred Req:       delete after 30 days (warn at 25 days)
#   Section 3 — Open Questions:     delete after 14 days
#   Section 4 — Vague Findings:     delete after 7 days
#
# Claude — writing: append new items to the relevant section. Use → prefix for unresolved.
#   Never delete items mid-session. Cleanup runs at session start only.
# Human — can mark any item ✅ DONE or ❌ DISCARDED at any time. Claude removes on next session start.

---

## Section 1: Retry Queue
Ticket creations that failed or were skipped during a session. Auto-deleted after 7 days.
Format: → [YYYY-MM-DD] | [ticket title] | reason: [error or skip reason] | action: RETRY / DISCARD

(empty)

---

## Section 2: Deferred Requirements
Requirements needing a product decision before a ticket can be created. Auto-deleted after 30 days.
Format: → [YYYY-MM-DD] | [requirement summary] | reason: [why deferred] | decision needed from: [who]

(empty)

---

## Section 3: Open Questions
Exploratory thoughts not ready to become tickets. Auto-deleted after 14 days.
Format: → [YYYY-MM-DD] | [question] | raised by: [who] | decision needed from: [who]

(empty)

---

## Section 4: Vague Findings
QA findings too vague to draft a ticket. Auto-deleted after 7 days.
Format: → [YYYY-MM-DD] | [raw finding text] | session: [qa-session-date] | action: RESUBMIT / DISCARD

(empty)
