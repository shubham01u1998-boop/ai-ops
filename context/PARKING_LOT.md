# PARKING_LOT — TiffinConnect
# VERSION: 1.11 | Created: 2026-05-14 | Last Reviewed: (unset) | Maintained by: Claude (auto) + QA Lead (review)
#
# CHANGELOG:
#   1.10 (2026-05-14) — Gap fixes: step 1 now explicitly exempts expired S2 items from silent
#                        deletion (same as S4 RESUBMIT); step 4a reworded as human-action prompt
#                        (not autonomous escalation); batch pre-checks for 4c items and replaces
#                        SKIP ALL with SKIP NON-REQUIRED when required items are present.
#   1.9  (2026-05-14) — Gap fixes: removed undefined "attempt at end of session" from step 4d LATER
#                        path; flattened nested sub-prompt in 4d (RETRY now single-level); batch
#                        warnings sorted by severity then priority; 4a items marked as
#                        notification-only in batch (no user choice); step 6 "commands" replaced
#                        with explicit file-marking instruction.
#   1.8  (2026-05-14) — Added batching rule for step 4 warnings; step 4d LATER option; step 6
#                        compact table on YES; response paths for 4f/4g.
#   1.7  (2026-05-14) — Gap fixes: RESUBMIT-tagged S4 pre-expiry warning at day 5-6 (step 4g split);
#                        extension field update rule clarified (append on first, update in-place after);
#                        extension count base case defined (absent field = count 0); session-start
#                        permitted in-place edits documented; (unset) last-reviewed message fixed.
#   1.6  (2026-05-14) — Gap fixes: step 1 exempts RESUBMIT-tagged S4 items from silent deletion;
#                        step 4d response path defined; step 4f/4g response paths defined; extension
#                        cap (max 2 per item); extended field added to S2/S3 format; TTL measures
#                        from extended date if present; original item date preserved.
#   1.5  (2026-05-14) — Gap fixes: S4 pre-expiry warning (step 4f), RESUBMIT-tagged S4 exempt from
#                        silent deletion (step 4g), extension mechanic for S2/S3 YES responses,
#                        RESUBMIT marks original as PROMOTED.
#   1.4  (2026-05-14) — Gap fixes: retry increment rule, escalate-by auto-compute formula, RESUBMIT
#                        behavior defined, S1 pre-expiry warning, checklist step overlap dedup,
#                        S1 deduplication rule, TEAM_CONTEXT fallback, [role] vs [who] clarified,
#                        S3 pre-expiry warning, vacant PM fallback.
#   1.3  (2026-05-14) — Gap fixes: PROMOTED prefix, escalate-by date check, max-retries threshold,
#                        priority criteria reference, role-based escalation.
#   1.2  (2026-05-14) — Added: TICKETED state, retry counter, raised-by on S1/S4, priority field,
#                        context field on S3, escalate-by on S2, escalation path, max-items guardrail,
#                        cross-promotion, mid-session announce rule, last-reviewed cadence check.
#   1.1  (2026-05-14) — Added TTL-based auto-cleanup logic.
#   1.0  (2026-05-14) — Initial structure.
#
# --- FIELD CONVENTIONS ----------------------------------------------------------
#   [who]  — person who raised the item. Use name or Odoo user ID from TEAM_CONTEXT.md Section 1.
#   [role] — person whose decision is needed. Use role title from TEAM_CONTEXT.md System Roles
#            (e.g. "QA Lead", "PM"). Do not use a person's name — roles can change hands.
#
# --- RESOLUTION STATES ----------------------------------------------------------
#   ✅ DONE           — resolved without a ticket
#   ✅ TICKETED #<ID> — resolved; Odoo ticket ID must be included
#   ❌ DISCARDED      — intentionally dropped
#   ⤴️ PROMOTED       — moved to another section; original line kept until next session start
#   Claude removes all resolved and promoted lines on next session start. No TTL needed.
#
# --- SESSION-START CHECKLIST (Claude runs in order) -----------------------------
#   1. Auto-cleanup: scan all items, apply TTL rules below, delete expired items silently.
#      Exceptions — do NOT silently delete:
#        - Section 2 items at or past their TTL — handle via step 4c.
#        - Section 4 items with action: RESUBMIT — handle via step 4g.
#      After cleanup, show one line: "Auto-cleaned [N] expired items from PARKING_LOT."
#      If N=0: no message.
#   2. Resolved items: delete any line marked ✅ DONE, ✅ TICKETED, ❌ DISCARDED, or ⤴️ PROMOTED.
#   3. Last-reviewed check:
#      - If "Last Reviewed" in line 2 is "(unset)": surface "PARKING_LOT has never been reviewed — schedule a review with QA Lead."
#      - If > 14 days ago: surface "PARKING_LOT has not been reviewed in [N] days — schedule a review with QA Lead."
#   4. Pre-expiry warnings — BATCHING RULE:
#      Collect ALL warnings from steps 4a–4g before showing any of them.
#      Sort collected warnings in this order before display:
#        1st — ESCALATION DUE (step 4a) and EXPIRED (step 4c), regardless of priority
#        2nd — HIGH priority items
#        3rd — MED priority items
#        4th — LOW priority items
#        Within each tier, preserve document order (S1 → S2 → S3 → S4).
#      Before presenting choices, check if any collected warnings are EXPIRED (step 4c):
#        - If YES: replace SKIP ALL with SKIP NON-REQUIRED and prepend:
#          "Note: [N] expired item(s) require a decision and cannot be skipped."
#        - If NO: offer SKIP ALL as normal.
#      Show batch block:
#        "[N] PARKING_LOT items need attention:
#         [1] S[section]: '[title/summary]' — [reason]
#         [2] S[section]: '[title/summary]' — [reason]"
#        "Address now? YES (go through each) / SKIP ALL or SKIP NON-REQUIRED / [n] (address item n only)"
#      On SKIP ALL: proceed with session — no items addressed, warnings fire again next session start.
#      On SKIP NON-REQUIRED: skip all items except EXPIRED (step 4c) ones — address those now, then proceed.
#      On YES: address items one by one in sorted order using response options in steps 4a–4g.
#      On [n]: address only that numbered item, skip the rest.
#        Exception: if a 4c EXPIRED item exists in the batch and human selects a non-4c item via [n],
#        resolve the 4c item first: "Expired item '[title]' must be resolved first."
#        Present 4c options (Escalate / DISCARD), wait for resolution, then continue with [n].
#      If 0 warnings: proceed silently to step 5.
#
#      Per-item rules (if step 4a fires for a Section 2 item, skip 4b/4c for that same item):
#      a. Any Section 2 item with an explicit "escalate by: [YYYY-MM-DD]" field at or past that date:
#         Type: NOTIFICATION — no user choice required.
#         Claude states: "ACTION NEEDED: notify [role per item] (or QA Lead if PM is vacant) that
#           '[title]' requires a decision — escalate-by date has passed."
#         Claude advances to the next batch item automatically.
#      b. Any Section 2 item within 5 days of its TTL deadline (no escalate-by date, or escalate-by not yet triggered):
#         "Deferred requirement '[title]' expires in [N] days — still relevant? YES / DISCARD"
#         On YES:
#           - If no extended field present, count = 0.
#           - If count < 2: if count = 0, append "extended: [today] (x1)" to the line;
#             otherwise update the existing extended field in-place to "extended: [today] (x[count+1])". TTL resets from today.
#           - If count >= 2: "This item has been extended twice — escalate to [role] or DISCARD. No further extension available."
#         On DISCARD: mark ❌ DISCARDED.
#      c. Any Section 2 item at or past its TTL with no decision — DO NOT silently delete:
#         "EXPIRED: '[title]' — no decision received. Escalate to [role per item] or DISCARD?"
#         Cannot be skipped via SKIP ALL or SKIP NON-REQUIRED. Must be resolved before proceeding.
#         Wait for human input before removing.
#      d. Any Section 1 item at day 5 or later (within 2 days of 7-day TTL):
#         "Retry item '[title]' expires in [N] days — RETRY / DISCARD / LATER"
#         On RETRY: increment retries: [N] in-place and reset item date to today (session-start
#           in-place edits). Ticket creation is queued and runs after the full checklist completes.
#           After checklist, if any RETRY items are queued, show before proceeding with session:
#             "Running [N] queued retry/retries from PARKING_LOT S1:"
#             For each: ✅ #[id] '[title]' created → [url]
#                    OR ❌ '[title]' failed — [error]. RETRY again / SKIP / STOP
#           RETRY again: one further attempt only. If it fails again: force SKIP or STOP.
#           SKIP: mark item ❌ DISCARDED in S1, continue.
#           STOP: leave remaining retries in S1 queue, proceed with session without retrying.
#         On DISCARD: mark ❌ DISCARDED.
#         On LATER: no change — item remains, warning fires again next session start.
#      e. Any Section 3 item at day 12 or later (within 2 days of 14-day TTL):
#         "Open question '[question]' expires in [N] days — still relevant? YES / DISCARD"
#         On YES:
#           - If no extended field present, count = 0.
#           - If count < 2: if count = 0, append "extended: [today] (x1)" to the line;
#             otherwise update the existing extended field in-place to "extended: [today] (x[count+1])". TTL resets from today.
#           - If count >= 2: "This item has been extended twice — escalate to [role] or DISCARD. No further extension available."
#         On DISCARD: mark ❌ DISCARDED.
#      f. Any Section 4 item at day 5 or later with action: DISCARD or no action set:
#         "Vague finding '[text]' expires in [N] days — RESUBMIT or DISCARD?"
#         On RESUBMIT: follow RESUBMIT rule in Writing Rules (append S1 entry, mark original ⤴️ PROMOTED).
#         On DISCARD: mark ❌ DISCARDED.
#      g. Any Section 4 item with action: RESUBMIT:
#         - At day 5 or 6 (pre-expiry warning):
#           "Vague finding '[text]' tagged RESUBMIT expires in [N] days — RESUBMIT now / LATER"
#           On RESUBMIT now: follow RESUBMIT rule in Writing Rules.
#           On LATER: no change — item remains, step 4g fires again next session start.
#         - At day 7 or later (expiry — DO NOT silently delete):
#           "Vague finding '[text]' tagged RESUBMIT is expiring — RESUBMIT now or DISCARD?"
#           On RESUBMIT now: follow RESUBMIT rule in Writing Rules (append S1 entry, mark original ⤴️ PROMOTED).
#           On DISCARD: mark ❌ DISCARDED.
#   5. Max-items check: if any section has >= 10 unresolved items, surface:
#      "WARNING: Section [N] has [X] items — process may be stalled. Review recommended."
#   6. After all cleanup, if any → items remain: show
#      "PARKING_LOT has [N] unresolved items — review first? YES / SKIP"
#      On YES: show compact table of all → items grouped by section:
#        S1 Retry Queue: [N] items | S2 Deferred: [N] | S3 Questions: [N] | S4 Vague: [N]
#        Then list each item as: [section] [date] [title/summary] [priority]
#        Display only — no action taken here. Human acts by marking items directly in the file
#        (✅ DONE, ✅ TICKETED #<ID>, or ❌ DISCARDED) or at next session start.
#      On SKIP: proceed with session. PARKING_LOT not surfaced again until next session start.
#
# --- TTL RULES (measured from item date, or from extended date if present) ------
#   Section 1 — Retry Queue:    delete after  7 days (warn at day 5; date resets on RETRY)
#   Section 2 — Deferred Req:   delete after 30 days (check escalate-by first; warn at day 25;
#                                escalate if expired — no silent delete; max 2 extensions)
#   Section 3 — Open Questions: delete after 14 days (warn at day 12; max 2 extensions)
#   Section 4 — Vague Findings: delete after  7 days (warn at day 5 for all; RESUBMIT-tagged
#                                items exempt from silent deletion — see step 4g)
#   Extended items: TTL measured from the date in the extended field (updated in-place on each
#   extension). Original item date at start of line is never modified — preserved for audit.
#
# --- PRIORITY CRITERIA ----------------------------------------------------------
#   Apply TEAM_CONTEXT.md Section 4 priority rules. Mapping to PARKING_LOT priority field:
#     urgent (1) -> HIGH  |  normal (0) -> MED  |  nice-to-have -> LOW
#   When in doubt: MED.
#   If TEAM_CONTEXT.md is unavailable: default to MED; escalate to QA Lead (Shubham Upadhyay, Odoo ID: 42).
#
# --- PERMITTED IN-PLACE EDITS ---------------------------------------------------
#   Session-start (checklist steps only — not mid-session):
#     - Increment retries: [N] on Section 1 item when user chooses RETRY (step 4d)
#     - Reset item date on Section 1 item when user chooses RETRY (step 4d)
#     - Append or update extended field on Section 2/3 item when user chooses YES (steps 4b/4e)
#   Mid-session (Writing Rules — during active work):
#     - Increment retries: [N] on Section 1 item on retry attempt (dedup rule)
#   All other changes mid-session are appends only. Never delete items mid-session.
#
# --- WRITING RULES (Claude mid-session) -----------------------------------------
#   - Append new items to the relevant section using → prefix.
#   - Announce inline when adding: "Added to PARKING_LOT S[N]: [brief title]"
#   - Deduplication (Section 1): before appending, check if an entry with the same ticket title
#     exists. If found, increment retries: [N] on that existing line in-place instead of appending.
#   - Section 1 retry threshold: if retries >= 3, surface "[title] has failed 3 times — DISCARD recommended."
#     Do not auto-discard — wait for human confirmation.
#   - RESUBMIT (Section 4): when a vague finding is clarified and marked action: RESUBMIT,
#     Claude appends a corresponding entry to Section 1 with retries: 0, same raised-by and priority,
#     reason: "resubmitted from S4 on [YYYY-MM-DD]", and announces inline.
#     Simultaneously mark the original S4 item ⤴️ PROMOTED — it will be removed at next session start.
#   - Cross-promotion: to move an item between sections, append it to the target section with
#     "promoted from: S[N] on [YYYY-MM-DD]" at the end of the line. Mark the original ⤴️ PROMOTED —
#     it will be removed at next session start.
#
# --- ESCALATION PATH ------------------------------------------------------------
#   Read current role holders from TEAM_CONTEXT.md System Roles section.
#   Default path: QA Lead → PM (when joined).
#   If PM is vacant: QA Lead handles all escalations until PM role is filled.
#   If TEAM_CONTEXT.md is unavailable: escalate to QA Lead (Shubham Upadhyay, Odoo ID: 42).
#
# --- ESCALATE-BY AUTO-COMPUTE (Section 2 new items) -----------------------------
#   When appending a new Section 2 item, compute escalate-by from item date + priority offset:
#     HIGH -> item date + 7 days
#     MED  -> item date + 14 days
#     LOW  -> item date + 25 days
#
# --- HUMAN REVIEW ---------------------------------------------------------------
#   Human — mark any item ✅ DONE, ✅ TICKETED #<ID>, or ❌ DISCARDED at any time.
#   Human — update "Last Reviewed:" in line 2 after each Lead review session.

---

## Section 1: Retry Queue
Ticket creations that failed or were skipped during a session. Auto-deleted after 7 days (warn at day 5; date resets on RETRY).
Format: → [YYYY-MM-DD] | [ticket title] | reason: [error or skip reason] | retries: [N] | raised by: [who] | priority: LOW/MED/HIGH | action: RETRY / DISCARD

(empty)

---

## Section 2: Deferred Requirements
Requirements needing a product decision before a ticket can be created. Auto-deleted after 30 days from item date or most recent extended date (max 2 extensions; escalated, not silently deleted, if expired).
Format: → [YYYY-MM-DD] | [requirement summary] | reason: [why deferred] | decision needed from: [role] | escalate by: [YYYY-MM-DD] | priority: LOW/MED/HIGH | extended: [YYYY-MM-DD] (xN) [optional]

(empty)

---

## Section 3: Open Questions
Exploratory thoughts not ready to become tickets. Auto-deleted after 14 days from item date or most recent extended date (max 2 extensions; warn at day 12).
Format: → [YYYY-MM-DD] | [question] | context: [brief background] | raised by: [who] | decision needed from: [role] | priority: LOW/MED/HIGH | extended: [YYYY-MM-DD] (xN) [optional]

(empty)

---

## Section 4: Vague Findings
QA findings too vague to draft a ticket. Auto-deleted after 7 days (warn at day 5 for all; RESUBMIT-tagged items exempt from silent deletion).
Format: → [YYYY-MM-DD] | [raw finding text] | session: [qa-session-date] | raised by: [who] | priority: LOW/MED/HIGH | action: RESUBMIT / DISCARD

(empty)
