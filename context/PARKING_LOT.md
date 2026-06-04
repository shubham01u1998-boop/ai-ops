# PARKING_LOT — TiffinConnect | Data File
# Rules, TTL logic, and session-start checklist: see PARKING_LOT_SPEC.md
# VERSION: 1.30 | Created: 2026-05-14 | Last Reviewed: (unset)

---

## Section 1: Retry Queue
Ticket creations that failed or were skipped during a session. Auto-deleted after 7 days (warn at day 5; date resets on RETRY).
Format: → [YYYY-MM-DD] | [ticket title] | reason: [error or skip reason] | retries: [N] | raised by: [who] | priority: LOW/MED/HIGH | action: RETRY / DISCARD

(empty)

---

## Section 2: Deferred Requirements
Requirements needing a product decision before a ticket can be created. Auto-deleted after 30 days from item date or most recent extended date (max 2 extensions, max 3 escalations cumulative; escalated, not silently deleted, if expired).
Format: → [YYYY-MM-DD] | [requirement summary] | reason: [why deferred] | decision needed from: [role] | escalate by: [YYYY-MM-DD] | priority: LOW/MED/HIGH | extended: [YYYY-MM-DD] (xN) [optional] | escalation sent: [YYYY-MM-DD] (xN) [optional]

(empty)

---

## Section 3: Open Questions
Exploratory thoughts not ready to become tickets. Auto-deleted after 14 days from item date or most recent extended date (max 2 extensions; warn at day 12).
Format: → [YYYY-MM-DD] | [question] | context: [brief background] | raised by: [who] | decision needed from: [role] | priority: LOW/MED/HIGH | extended: [YYYY-MM-DD] (xN) [optional]


---

## Section 4: Vague Findings
QA findings too vague to draft a ticket. Auto-deleted after 7 days (warn at day 5 for all; RESUBMIT-tagged items exempt from silent deletion).
Format: → [YYYY-MM-DD] | [raw finding text] | session: [qa-session-date] | raised by: [who] | priority: LOW/MED/HIGH | action: RESUBMIT / DISCARD

