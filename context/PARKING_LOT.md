# PARKING_LOT — TiffinConnect
# VERSION: 1.3 | Created: 2026-05-14 | Last Reviewed: (unset) | Maintained by: Claude (auto) + QA Lead (review)
#
# CHANGELOG:
#   1.3 (2026-05-14) — Gap fixes: PROMOTED prefix for cross-promotion, escalate-by date check in
#                       session-start checklist, max-retries threshold (≥3 → DISCARD recommended),
#                       priority criteria reference, role-based escalation via TEAM_CONTEXT System Roles.
#   1.2 (2026-05-14) — Added: TICKETED resolution state, retry counter on S1, raised-by on S1/S4,
#                       priority field on all sections, context field on S3, escalate-by on S2,
#                       escalation path for expired S2 items, max-items guardrail, cross-promotion
#                       instructions, mid-session announce rule, last-reviewed cadence check.
#   1.1 (2026-05-14) — Added TTL-based auto-cleanup logic.
#   1.0 (2026-05-14) — Initial structure.
#
# ─── RESOLUTION STATES ──────────────────────────────────────────────────────────
#   ✅ DONE           — resolved without a ticket
#   ✅ TICKETED #<ID> — resolved; Odoo ticket ID must be included
#   ❌ DISCARDED      — intentionally dropped
#   ⤴️ PROMOTED       — moved to another section; original line kept until next session start
#   Claude removes all resolved and promoted lines on next session start. No TTL needed.
#
# ─── SESSION-START CHECKLIST (Claude runs in order) ─────────────────────────────
#   1. Auto-cleanup: scan all items, apply TTL rules below, delete expired items silently.
#      After cleanup, show one line: "Auto-cleaned [N] expired items from PARKING_LOT."
#      If N=0: no message.
#   2. Resolved items: delete any line marked ✅ DONE, ✅ TICKETED, ❌ DISCARDED, or ⤴️ PROMOTED.
#   3. Last-reviewed check: if "Last Reviewed" in line 2 is "(unset)" or > 14 days ago, surface:
#      "PARKING_LOT has not been reviewed in [N] days — schedule a review with QA Lead."
#   4. Expiry warnings:
#      a. Any Section 2 item with an explicit "escalate by: [YYYY-MM-DD]" field — check that date
#         against today. If at or past that date, surface immediately regardless of 30-day TTL:
#         "ESCALATION DUE: '[title]' — escalate by date reached. Notify PM (when joined) or QA Lead."
#      b. Any Section 2 item within 5 days of its 30-day TTL (no escalate-by date) → surface:
#         "Deferred requirement '[title]' expires in [N] days — still relevant? YES / DISCARD"
#      c. Any Section 2 item at or past 30 days with no decision → DO NOT silently delete.
#         Surface: "EXPIRED: '[title]' — no decision received. Escalate to PM (when joined) / QA Lead, or DISCARD?"
#         Wait for human input before removing.
#   5. Max-items check: if any section has ≥ 10 unresolved items, surface:
#      "WARNING: Section [N] has [X] items — process may be stalled. Review recommended."
#   6. After all cleanup, if any → items remain: show
#      "PARKING_LOT has [N] unresolved items — review first? YES / SKIP"
#
# ─── TTL RULES (measured from item date) ────────────────────────────────────────
#   Section 1 — Retry Queue:    delete after  7 days
#   Section 2 — Deferred Req:   delete after 30 days (check escalate-by date first; escalate if expired, no silent delete)
#   Section 3 — Open Questions: delete after 14 days
#   Section 4 — Vague Findings: delete after  7 days
#
# ─── PRIORITY CRITERIA ──────────────────────────────────────────────────────────
#   Apply TEAM_CONTEXT.md Section 4 priority rules. When in doubt: MED.
#   HIGH = matches urgent (1) conditions | MED = normal (0) | LOW = nice-to-have
#
# ─── WRITING RULES (Claude mid-session) ─────────────────────────────────────────
#   - Append new items to the relevant section using → prefix.
#   - Never delete items mid-session. Cleanup runs at session start only.
#   - When adding a new item mid-session, announce inline: "Added to PARKING_LOT S[N]: [brief title]"
#   - Section 1 retry threshold: if retries ≥ 3, surface: "[title] has failed 3 times — DISCARD recommended."
#     Do not auto-discard — wait for human confirmation.
#   - Cross-promotion: to move an item between sections, append it to the target section with
#     "promoted from: S[N] on [YYYY-MM-DD]" at the end of the line. Mark the original ⤴️ PROMOTED —
#     it will be removed at next session start.
#
# ─── ESCALATION PATH ─────────────────────────────────────────────────────────────
#   Read current role holders from TEAM_CONTEXT.md System Roles section.
#   Default path: QA Lead → PM (when joined).
#
# ─── HUMAN REVIEW ────────────────────────────────────────────────────────────────
#   Human — mark any item ✅ DONE, ✅ TICKETED #<ID>, or ❌ DISCARDED at any time.
#   Human — update "Last Reviewed:" in line 2 after each Lead review session.

---

## Section 1: Retry Queue
Ticket creations that failed or were skipped during a session. Auto-deleted after 7 days.
Format: → [YYYY-MM-DD] | [ticket title] | reason: [error or skip reason] | retries: [N] | raised by: [who] | priority: LOW/MED/HIGH | action: RETRY / DISCARD

(empty)

---

## Section 2: Deferred Requirements
Requirements needing a product decision before a ticket can be created. Auto-deleted after 30 days (escalated, not silently deleted, if expired).
Format: → [YYYY-MM-DD] | [requirement summary] | reason: [why deferred] | decision needed from: [role] | escalate by: [YYYY-MM-DD] | priority: LOW/MED/HIGH

(empty)

---

## Section 3: Open Questions
Exploratory thoughts not ready to become tickets. Auto-deleted after 14 days.
Format: → [YYYY-MM-DD] | [question] | context: [brief background] | raised by: [who] | decision needed from: [role] | priority: LOW/MED/HIGH

(empty)

---

## Section 4: Vague Findings
QA findings too vague to draft a ticket. Auto-deleted after 7 days.
Format: → [YYYY-MM-DD] | [raw finding text] | session: [qa-session-date] | raised by: [who] | priority: LOW/MED/HIGH | action: RESUBMIT / DISCARD

(empty)
