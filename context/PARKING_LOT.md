# PARKING_LOT — TiffinConnect
# VERSION: 1.0 | Created: 2026-05-14 | Maintained by: Claude (auto) + Lead (review)
#
# Claude — session start: if this file exists, scan all sections for unresolved items (→).
#   Surface them before proceeding: "PARKING_LOT has [N] unresolved items — review first? YES / SKIP"
# Claude — writing: append new items to the relevant section. Use → prefix for unresolved.
#   Never delete entries. Never resolve items without human approval.
# Human — review weekly: mark items ✅ DONE or ❌ DISCARDED. Prune resolved items monthly.

---

## Section 1: Retry Queue
Ticket creations that failed or were skipped during a session. Retry or discard.
Format: → [date] | [ticket title] | reason: [error or skip reason] | action: RETRY / DISCARD

(empty)

---

## Section 2: Deferred Requirements
Requirements that need a product decision before a ticket can be created.
Format: → [date] | [requirement summary] | reason: [why deferred] | decision needed from: [who]

(empty)

---

## Section 3: Open Questions
Exploratory thoughts not ready to become tickets. Discuss further or discard.
Format: → [date] | [question] | raised by: [who] | decision needed from: [who]

(empty)

---

## Section 4: Vague Findings
QA findings too vague to draft a ticket. Resubmit with more detail or discard.
Format: → [date] | [raw finding text] | session: [qa-session-date] | action: RESUBMIT / DISCARD

(empty)
