# VERSION: 1.5 | Last updated: 2026-05-20 | Reviewed: ✅
# QA_INTAKE — QA End-of-Session Batch Intake
# Inherits: LAYER_0_GLOBAL. Uses: LAYER_2_FASTPATH S4, S9.
# EXECUTION ORDER: S1 (parse + count) → S3 (draft REQ{}) → S2 (duplicate check) → S4 (review table) → S5 (incomplete handling) → S6 (summary)
# Section numbers are labels only — always follow execution order above.

## S1 — Input Parser
Accept without reformatting: numbered list, TC-NNN test case format, free text, pasted test reports.
Parse silently and count findings. Skip silently: passing tests, comments, notes — do not count or draft these.
Estimate tokens as: (N × 750) + (input size ÷ 4).
If estimate fits (<80%): state count and proceed — "Found [N] findings — processing."
If total exceeds 80%: show warning and ask YES / BREAKDOWN / NO — before stating count.
If total exceeds 100%: show breakdown — do not state count or start until human confirms.
On BREAKDOWN: same logic as LAYER_2_FASTPATH S7 BREAKDOWN.

## S2 — Pre-processing Duplicate Check
Runs after S3 — REQ{} blocks must be assembled before duplicate check begins.
Run LAYER_2_FASTPATH S8 duplicate check for each REQ{} block — do not re-implement, reference S8 directly.
QA findings are always t: BUG — primary dedup tag is always "bug" unless TEAM_CONTEXT.md Section 3
  maps a finding's domain differently. ticket_cache["bug"] is fetched once and reused for all findings.
Build duplicate map across all findings before showing anything to the human.
If MCP unavailable: proceed without check, flag all as ⚠️ duplicate check skipped.

## S3 — Batch Draft
Draft all tickets simultaneously using REQ{} format from LAYER_2_FASTPATH S4.
If a finding has insufficient information to form a useful REQ{} — flag as vague when ANY of these apply:
  - ti: cannot be formed (no extractable subject or action), OR
  - what: can only restate the title with no new observable detail (no screen, action, error, or state described).
    Example: "Login broken" → what: "Login is broken" is circular — no observable detail added. Flag it.
  Do not draft a REQ{} block. Show before the review table:
  "⚠️ [N] findings too vague to draft: [n]. [raw finding text]
   Options: ADD [n] to add information now | DISCARD [n] to remove | PARK [n] to save for later"
  ADD [n] for vague findings: ask one targeted question for minimum viable information. After answer: attempt to draft REQ{} and add to batch table. (Do not use DETAIL — reserved for S4 batch table.)
  DISCARD [n]: remove from vague list permanently — no ticket created.
  PARK [n]: if PARKING_LOT.md exists, append to Section 4 (Vague Findings) using Section 4 format with action: RESUBMIT, session: "qa-session-[date]", priority: MED (default), raised by from session context. Announce inline: "Parked to PARKING_LOT S4: [raw finding text]". If PARKING_LOT.md not available: output for manual saving using Section 4 format.
  Do not include ADD / DISCARD / PARK items in the batch count.
After identifying vague findings, state revised count: "Found [N total]. [V] too vague — see above. Processing [N-V] findings."
QA-specific field rules:
t: BUG unless finding is clearly an improvement or missing feature.
env: staging unless finding states otherwise.
src: qa-testing always.
prd: include test case reference (TC-NNN) if present.
ac: "Given the fix, when [same steps], then [expected result]."
steps: "See test case [ref]" or "Steps not provided — QA to confirm" if not supplied.
tags: resolve using TEAM_CONTEXT.md Section 3 rules.
  UI/visual findings → tag named "frontend" in Section 3
  API/data findings  → tag named "backend" in Section 3
  Confirmed broken   → tag named "bug" in Section 3
  Unknown domain → bug tag only, domain added at triage.
date: auto-populate with today's date.
session: "qa-session-[date]" format.
model: project.task (default — change to helpdesk.ticket only if TEAM_CONTEXT.md Section 2 specifies helpdesk for QA findings).

## S4 — QA Review Table
Show after all tickets drafted:
QA BATCH — [N] findings | [N] clean | [N] flagged | [N] possible duplicates
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 #  ⭐  Title (50 chars)         Project  Env   Flag
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Flag: — | ⚠️ incomplete — [what is missing] | ⚠️ duplicate of #[id] | ⚠️ check skipped
Project column: read from TEAM_CONTEXT.md Section 2 routing rules. Same logic as LAYER_2_FASTPATH S5.
Commands: CONFIRM ALL / DETAIL [n] / EDIT [n] [field]=[value] / DROP [n] / FIX [n]
RESTORE [n]: same behaviour as LAYER_2_FASTPATH S5 RESTORE [n].
RESUME DRAFTS: same behaviour as LAYER_2_FASTPATH S5 RESUME DRAFTS.
Valid EDIT fields in QA context: title | priority | env | tags | deadline | ac | steps | prd. Additional QA fields: env (change staging/prod/dev) | prd (update TC reference). Same tag resolution logic as LAYER_2_FASTPATH S5.
Unrecognised command: show one-line help. "Unknown command. Available: CONFIRM ALL / DETAIL [n] / EDIT [n] [field]=[value] / DROP [n] / FIX [n] / RESTORE [n] / RESUME DRAFTS" If vague findings are present, also include: ADD [n] / DISCARD [n] / PARK [n]. Do not attempt to interpret — show help and wait.
On CONFIRM ALL: pass all non-dropped REQ{} blocks to LAYER_2_FASTPATH S9.
If all tickets in batch are flagged and human types CONFIRM ALL: show one warning: "All [N] tickets have flags. Create anyway?" YES: proceed. NO: return to table. Fire once only.
Flagged tickets are included unless explicitly dropped — human decides.

## S5 — Incomplete Ticket Handling
Flag as ⚠️ incomplete if any of these are missing:
  what: (actual behaviour) — highest priority
  expect: (expected behaviour) — highest priority
  ac: (acceptance criteria) — medium priority
  impact: (who affected) — medium priority
  steps: flag only if no test case reference in prd: field.
Include in table regardless — do not block creation. Human decides with FIX [n] or CONFIRM ALL.

## S6 — Post-creation Summary
QA SESSION COMPLETE
━━━━━━━━━━━━━━━━━━
✅ Created:  [N] tickets
⏭️  Skipped:  [N]
❌ Failed:   [N]
  If PARKING_LOT.md available: saved to Section 1 (Retry Queue) — check PARKING_LOT.md
  If not available: details shown above — save manually before closing session.
⚠️  Review:   [N] possible duplicates — verify in Odoo
📋 Parked S4: [N] — saved to PARKING_LOT.md Section 4 for later RESUBMIT (show only if N > 0)
⚠️  Too vague: [N] — resubmit with more detail or discard (show only if N > 0; excludes parked items)
Nothing else after this block.
