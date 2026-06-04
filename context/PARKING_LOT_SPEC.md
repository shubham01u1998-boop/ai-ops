# PARKING_LOT SPEC — TiffinConnect
# VERSION: 1.30 | Created: 2026-05-14 | Maintained by: Claude (auto) + QA Lead (review)
# Rules, TTL logic, and session-start checklist for PARKING_LOT.md (data file).
# Data file: context/PARKING_LOT.md
#
# CHANGELOG:
#   1.30 (2026-05-19) — Gap fix: standard dedup rule now explicitly invokes the retry threshold
#                        check after incrementing (consistent with RESUBMIT dedup path v1.27);
#                        threshold message changed from hardcoded "3 times" to dynamic "[N] times"
#                        (consistent with step 4d pre-check wording).
#   1.29 (2026-05-18) — Gap fix: step 4d now checks retries >= 3 before presenting the prompt —
#                        prepends "[title] has already failed [N] times." so the user sees the
#                        failure history before choosing RETRY; RETRY is still offered (no auto-discard).
#   1.28 (2026-05-18) — Gap fix: Writing Rules append rule now explicitly references ESCALATE-BY
#                        AUTO-COMPUTE for Section 2 items, consistent with cross-promotion rule.
#   1.27 (2026-05-18) — Gap fix: RESUBMIT dedup path now explicitly applies the retry threshold
#                        check after incrementing retries, so "failed 3 times" warning fires even
#                        when the increment was triggered by RESUBMIT rather than a fresh append.
#   1.26 (2026-05-18) — Gap fix: RESUBMIT rule now applies S1 dedup check first — if an existing
#                        S1 entry with the same ticket title is found, increments retries in-place
#                        instead of appending a duplicate; ⤴️ PROMOTED applied in both cases.
#   1.25 (2026-05-18) — Gap fix: step 6 now counts only → items without a resolution state marker,
#                        excluding items resolved during step 4 that haven't been cleaned yet.
#   1.24 (2026-05-15) — Gap fix: cross-promotion rule now specifies item date = today (promotion date)
#                        so the new entry gets a full TTL window; ESCALATE-BY AUTO-COMPUTE base date
#                        is unambiguous; original date preserved in the "promoted from:" note.
#   1.23 (2026-05-15) — Gap fix: cross-promotion rule now specifies field-mapping guidance for
#                        Section 2 targets — apply ESCALATE-BY AUTO-COMPUTE to set escalate by:,
#                        and derive missing fields (e.g. reason) from context.
#   1.22 (2026-05-15) — Gap fix: Writing Rules RESUBMIT now explicitly states to use S4 raw finding
#                        text as the S1 ticket title and to set action: RETRY on the new S1 entry.
#   1.21 (2026-05-15) — Gap fix: step 4c escalation sent instruction now says "append (if absent) or
#                        update in-place" — at x0 the field doesn't exist yet so append is correct.
#   1.20 (2026-05-15) — Gap fix: PERMITTED IN-PLACE EDITS extended field authorisation now includes
#                        step 4c (step 4c EXTEND also updates the extended: field in-place).
#   1.19 (2026-05-14) — Gap fixes: step 4d "threshold rule applies" replaced with "per-session retry
#                        limit reached" (threshold rule is a mid-session dedup concept, not relevant
#                        here); STOP clarified to explicitly include the currently-failed item in S1.
#   1.18 (2026-05-14) — Gap fix: PERMITTED IN-PLACE EDITS now explicitly covers resolution-state
#                        marking (✅ TICKETED, ❌ DISCARDED, ⤴️ PROMOTED) for session-start checklist
#                        steps and post-checklist retry execution; mid-session block clarifies
#                        resolution-state marking is permitted and not subject to appends-only rule.
#   1.17 (2026-05-14) — Gap fixes: batch deduplication rule added for items triggering both 4a and
#                        4c (collected once, labelled ESCALATION DUE + EXPIRED, resolved in sequence);
#                        step 4b documented as safety net for non-standard S2 items only.
#   1.16 (2026-05-14) — Gap fixes: dedup rule now skips 4b only (not 4c) when 4a fires; items past
#                        both escalate-by and 30-day TTL get both 4a notification and 4c required
#                        decision; step 4c x0 EXTEND absence explained; cumulative escalation cap
#                        documented.
#   1.15 (2026-05-14) — Step 4c EXTEND now resets escalate by: field to today + priority offset;
#                        escalate by: reset added to Permitted In-Place Edits.
#   1.14 (2026-05-14) — Gap fixes: escalation sent field added to S2 format spec with (xN) counter;
#                        ESCALATE AGAIN capped at x3; step 4c EXTEND explicitly checks extended:
#                        counter; EXTEND from expired state documents full 30-day restart.
#   1.13 (2026-05-14) — Step 4c expanded: escalation sent tracking, ESCALATE / ESCALATE AGAIN /
#                        EXTEND / DISCARD options, follow-up messaging; escalation sent added to
#                        Permitted In-Place Edits.
#   1.12 (2026-05-14) — Gap fixes: successful retry marks S1 item ✅ TICKETED #[id]; step 4c
#                        reworded to match 4a human-action pattern; [n] blocking rule handles
#                        multiple 4c items; "force SKIP or STOP" clarified as hard stop.
#   1.11 (2026-05-14) — Added step 4d retry execution flow; [n] selection blocks on 4c EXPIRED items.
#   1.10 (2026-05-14) — Gap fixes: step 1 exempts expired S2 items from silent deletion; step 4a
#                        as human-action prompt; SKIP NON-REQUIRED when 4c items present.
#   1.9  (2026-05-14) — Gap fixes: removed undefined end-of-session path; flattened 4d sub-prompt;
#                        batch sorted by severity then priority; 4a notification-only; step 6 fixed.
#   1.8  (2026-05-14) — Added batching rule; step 4d LATER; step 6 compact table; 4f/4g paths.
#   1.7  (2026-05-14) — Gap fixes: S4 RESUBMIT pre-expiry warning; extension field update rule;
#                        extension count base case; session-start in-place edits documented.
#   1.6  (2026-05-14) — Gap fixes: step 1 S4 RESUBMIT exemption; step 4d/4f/4g paths; extension
#                        cap (max 2); extended field on S2/S3; TTL from extended date.
#   1.5  (2026-05-14) — Gap fixes: S4 pre-expiry warning; RESUBMIT S4 exemption; S2/S3 extension
#                        mechanic; RESUBMIT marks original as PROMOTED.
#   1.4  (2026-05-14) — Gap fixes: retry increment rule; escalate-by auto-compute; RESUBMIT behavior;
#                        S1 pre-expiry warning; overlap dedup; S1 dedup; TEAM_CONTEXT fallback;
#                        [role] vs [who]; S3 pre-expiry warning; vacant PM fallback.
#   1.3  (2026-05-14) — Gap fixes: PROMOTED prefix; escalate-by check; max-retries threshold;
#                        priority criteria reference; role-based escalation.
#   1.2  (2026-05-14) — Added: TICKETED state, retry counter, raised-by on S1/S4, priority field,
#                        context field on S3, escalate-by on S2, escalation path, max-items guardrail,
#                        cross-promotion, mid-session announce, last-reviewed cadence check.
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
# --- SESSION-START CHECKLIST (Claude runs in order against PARKING_LOT.md) ------
#   1. Auto-cleanup: scan all items, apply TTL rules below, delete expired items silently.
#      Exceptions — do NOT silently delete:
#        - Section 2 items at or past their TTL — handle via step 4c.
#        - Section 4 items with action: RESUBMIT — handle via step 4g.
#      After cleanup, show one line: "Auto-cleaned [N] expired items from PARKING_LOT."
#      If N=0: no message.
#   2. Resolved items: delete any line marked ✅ DONE, ✅ TICKETED, ❌ DISCARDED, or ⤴️ PROMOTED.
#   3. Last-reviewed check:
#      - Read "Last Reviewed" from line 3 of PARKING_LOT.md.
#      - If "(unset)": surface "PARKING_LOT has never been reviewed — schedule a review with QA Lead."
#      - If > 14 days ago: surface "PARKING_LOT has not been reviewed in [N] days — schedule a review with QA Lead."
#   4. Pre-expiry warnings — BATCHING RULE:
#      Collect ALL warnings from steps 4a–4g before showing any of them.
#      Deduplication: if a single S2 item qualifies for both step 4a and step 4c, add it to the
#        batch ONCE, labelled "ESCALATION DUE + EXPIRED". When addressed, resolve 4a (notification,
#        auto-advance) then 4c (required decision) in sequence for the same item.
#      Sort collected warnings in this order before display:
#        1st — ESCALATION DUE (step 4a), EXPIRED (step 4c), and ESCALATION DUE + EXPIRED (combined)
#        2nd — HIGH priority items
#        3rd — MED priority items
#        4th — LOW priority items
#        Within each tier, preserve document order (S1 → S2 → S3 → S4).
#      Before presenting choices, check if any collected warnings are EXPIRED or ESCALATION DUE + EXPIRED:
#        - If YES: replace SKIP ALL with SKIP NON-REQUIRED and prepend:
#          "Note: [N] expired item(s) require a decision and cannot be skipped."
#        - If NO: offer SKIP ALL as normal.
#      Show batch block:
#        "[N] PARKING_LOT items need attention:
#         [1] S[section]: '[title/summary]' — [reason]
#         [2] S[section]: '[title/summary]' — [reason]"
#        "Address now? YES (go through each) / SKIP ALL or SKIP NON-REQUIRED / [n] (address item n only)"
#      On SKIP ALL: proceed with session — no items addressed, warnings fire again next session start.
#      On SKIP NON-REQUIRED: skip all items except EXPIRED and ESCALATION DUE + EXPIRED ones —
#        address those now, then proceed.
#      On YES: address items one by one in sorted order using response options in steps 4a–4g.
#      On [n]: address only that numbered item, skip the rest.
#        Exception: if any EXPIRED or ESCALATION DUE + EXPIRED items exist in the batch and human
#        selects a non-expired item via [n], resolve all expired items first, in listed order:
#        "Expired item(s) must be resolved before item [n]. Addressing [N] expired item(s) first."
#        Present 4c options for each in turn, wait for resolution, then continue with [n].
#      If 0 warnings: proceed silently to step 5.
#
#      Per-item rules:
#        - If step 4a fires for a Section 2 item, skip 4b for that same item — do NOT skip 4c.
#        - If both 4a and 4c apply to the same item, collect as one combined "ESCALATION DUE + EXPIRED"
#          entry (deduplication rule above). When addressed: 4a notification fires first (auto-advance),
#          then 4c required decision fires immediately for the same item.
#      a. Any Section 2 item with an explicit "escalate by: [YYYY-MM-DD]" field at or past that date:
#         Type: NOTIFICATION — no user choice required.
#         Claude states: "ACTION NEEDED: notify [role per item] (or QA Lead if PM is vacant) that
#           '[title]' requires a decision — escalate-by date has passed."
#         Claude advances automatically: to 4c if the item is also expired, else to the next batch item.
#      b. Any Section 2 item within 5 days of its TTL deadline where escalate-by is not yet triggered
#         (i.e., escalate-by date is still in the future, or no escalate-by field present):
#         Note: this step is a safety net for non-standard items — items created via the auto-compute
#         formula always have escalate-by <= day 25, so step 4a fires before this window is reached.
#         Step 4b activates for: manually-added items without an escalate-by field, or items where
#         a human manually set escalate-by beyond day 25 of the item's life.
#         "Deferred requirement '[title]' expires in [N] days — still relevant? YES / DISCARD"
#         On YES:
#           - If no extended field present, count = 0.
#           - If count < 2: if count = 0, append "extended: [today] (x1)" to the line;
#             otherwise update the existing extended field in-place to "extended: [today] (x[count+1])". TTL resets from today.
#           - If count >= 2: "This item has been extended twice — escalate to [role] or DISCARD. No further extension available."
#         On DISCARD: mark ❌ DISCARDED.
#      c. Any Section 2 item at or past its TTL with no decision — DO NOT silently delete:
#         Note: escalation history persists across extensions — the x3 cap applies cumulatively
#         across the item's lifetime, not per TTL window.
#         Read escalation count from "escalation sent: [date] (xN)" field (absent = x0).
#         - If escalation count < 3:
#           If x0 (no escalation yet):
#             "ACTION NEEDED: '[title]' has expired with no decision received.
#               Escalate to [role per item] (or QA Lead if PM is vacant), or DISCARD?"
#             Note: EXTEND not available at x0 — at least one escalation must be sent first.
#               Escalate first; EXTEND becomes available at x1.
#           If x1 or x2:
#             "FOLLOW-UP: '[title]' was escalated [N] days ago (x[count] total).
#               Escalate again / DISCARD / EXTEND (if extensions remain, see below)"
#           On ESCALATE / ESCALATE AGAIN: append (if absent) or update in-place the escalation sent
#               field to "escalation sent: [today] (x[count+1])". Surface: "Notify [role] directly — '[title]' requires a decision."
#           On DISCARD: mark ❌ DISCARDED.
#           On EXTEND (x1 or x2 only; only if extended: count < 2):
#               Resets TTL to today + 30 days (full 30-day window from today — use only if item is
#               still genuinely active). Update extended field in-place to "extended: [today] (x[count+1])".
#               Reset escalate by: field to today + priority offset (use ESCALATE-BY AUTO-COMPUTE formula).
#               Item exits expired state; step 4b governs it on future sessions.
#           On EXTEND (if extended: count >= 2): option not available — remove from choices.
#         - If escalation count = 3:
#           "FINAL NOTICE: '[title]' has been escalated 3 times with no decision.
#             Maximum escalations reached — EXTEND (if extensions remain) or DISCARD only."
#           On EXTEND (only if extended: count < 2): apply same EXTEND rule above.
#           On EXTEND (if extended: count >= 2): option not available.
#           On DISCARD: mark ❌ DISCARDED.
#           ESCALATE AGAIN is no longer offered.
#         Cannot be skipped via SKIP ALL or SKIP NON-REQUIRED. Must be resolved before proceeding.
#      d. Any Section 1 item at day 5 or later (within 2 days of 7-day TTL):
#         Pre-check: if retries >= 3, prepend "[title] has already failed [N] times." before the
#           prompt — do not auto-discard, still offer all options.
#         "Retry item '[title]' expires in [N] days — RETRY / DISCARD / LATER"
#         On RETRY: increment retries: [N] in-place and reset item date to today (session-start
#           in-place edits). Ticket creation is queued and runs after the full checklist completes.
#           After checklist, if any RETRY items are queued, show before proceeding with session:
#             "Running [N] queued retry/retries from PARKING_LOT S1:"
#             For each: ✅ #[id] '[title]' created → [url] — mark item ✅ TICKETED #[id] immediately.
#                    OR ❌ '[title]' failed — [error]. RETRY again / SKIP / STOP
#           RETRY again: one further attempt only. If it fails again: no further RETRY offered
#             (item has now failed twice in this session — per-session retry limit reached).
#             Present SKIP / STOP only.
#           SKIP: mark item ❌ DISCARDED in S1, continue with remaining queued retries.
#           STOP: leave the current failed item and all remaining unprocessed items in S1 queue,
#             proceed with session without retrying further.
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
#   6. After all cleanup, if any → items remain without a resolution state marker (i.e. no
#      ✅ DONE, ✅ TICKETED, ❌ DISCARDED, or ⤴️ PROMOTED on the line): show
#      "PARKING_LOT has [N] unresolved items — review first? YES / SKIP"
#      On YES: show compact table of all such → items grouped by section:
#        S1 Retry Queue: [N] items | S2 Deferred: [N] | S3 Questions: [N] | S4 Vague: [N]
#        Then list each item as: [section] [date] [title/summary] [priority]
#        Display only — no action taken here. Human acts by marking items directly in the file
#        (✅ DONE, ✅ TICKETED #<ID>, or ❌ DISCARDED) or at next session start.
#      On SKIP: proceed with session. PARKING_LOT not surfaced again until next session start.
#
# --- TTL RULES (measured from item date, or from extended date if present) ------
#   Section 1 — Retry Queue:    delete after  7 days (warn at day 5; date resets on RETRY)
#   Section 2 — Deferred Req:   delete after 30 days (check escalate-by first; warn at day 25
#                                via step 4b safety net; escalate if expired — no silent delete;
#                                max 2 extensions, max 3 escalations cumulative;
#                                EXTEND from expired = full 30-day restart)
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
#   Session-start (checklist steps and post-checklist retry execution — not mid-session):
#     - Increment retries: [N] on Section 1 item when user chooses RETRY (step 4d)
#     - Reset item date on Section 1 item when user chooses RETRY (step 4d)
#     - Append or update extended field on Section 2/3 item when user chooses YES or EXTEND (steps 4b/4c/4e)
#     - Append or update escalation sent field on Section 2 item when user chooses ESCALATE (step 4c)
#     - Reset escalate by: field on Section 2 item when user chooses EXTEND (step 4c)
#     - Mark items ✅ TICKETED #[id], ❌ DISCARDED, or ⤴️ PROMOTED when user confirms a resolution
#       during any checklist step (4b, 4c, 4d, 4e, 4f, 4g) or during post-checklist retry execution
#   Mid-session (Writing Rules — during active work):
#     - Increment retries: [N] on Section 1 item on retry attempt (dedup rule)
#     - Mark items ✅ DONE, ✅ TICKETED #[id], ❌ DISCARDED, or ⤴️ PROMOTED when user confirms a
#       resolution conversationally — these are in-place edits and are NOT subject to the appends-only rule
#   All other changes mid-session are appends only. Never delete items mid-session.
#
# --- WRITING RULES (Claude mid-session) -----------------------------------------
#   - Append new items to the relevant section in PARKING_LOT.md using → prefix.
#     When appending to Section 2: apply ESCALATE-BY AUTO-COMPUTE to set escalate by: on the new entry.
#   - Announce inline when adding: "Added to PARKING_LOT S[N]: [brief title]"
#   - Deduplication (Section 1): before appending, check if an entry with the same ticket title
#     exists. If found, increment retries: [N] on that existing line in-place instead of appending,
#     then apply the retry threshold check.
#   - Section 1 retry threshold: if retries >= 3, surface "[title] has failed [N] times — DISCARD recommended."
#     Do not auto-discard — wait for human confirmation.
#   - RESUBMIT (Section 4): when a vague finding is clarified and marked action: RESUBMIT,
#     apply the S1 dedup check first — if an existing S1 entry with the same ticket title is found,
#     increment retries: [N] on that entry in-place instead of appending a new one, then apply the
#     retry threshold check (if retries >= 3 after incrementing, surface the DISCARD recommendation).
#     If no match, append a new Section 1 entry using the S4 [raw finding text] as the ticket title,
#     retries: 0, same raised-by and priority, action: RETRY,
#     reason: "resubmitted from S4 on [YYYY-MM-DD]", and announces inline.
#     In both cases, simultaneously mark the original S4 item ⤴️ PROMOTED — it will be removed at
#     next session start.
#   - Cross-promotion: to move an item between sections, append it to the target section with
#     "promoted from: S[N] on [YYYY-MM-DD]" at the end of the line. Mark the original ⤴️ PROMOTED —
#     it will be removed at next session start.
#     Item date: use today (the promotion date) — gives the new entry a full TTL window in its target
#     section. The "promoted from:" note preserves the audit trail of the original item's origin.
#     Field mapping: carry over matching fields (raised by, priority, decision needed from where
#     present). Derive missing fields from context (e.g. reason = "promoted from S[N]").
#     If target is Section 2: apply ESCALATE-BY AUTO-COMPUTE to set escalate by: on the new entry.
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
#   Human — update "Last Reviewed:" in line 3 of PARKING_LOT.md after each Lead review session.
