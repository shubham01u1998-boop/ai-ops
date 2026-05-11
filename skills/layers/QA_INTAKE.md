# QA_INTAKE — QA End-of-Session Batch Intake
# Inherits: LAYER_0_GLOBAL. Uses: LAYER_2_FASTPATH S4, S9.

## S1 — Input Parser
Accept without reformatting: numbered list, TC-NNN test case format, free text, pasted test reports.
Parse silently. State count before acting: "Found [N] findings — processing."
Skip silently: passing tests, comments, notes — do not count or draft these.

## S2 — Pre-processing Duplicate Check
Run duplicate check for all findings in one pass before drafting anything.
Batch all search calls in parallel — do not run sequentially. Maximum 3 search calls regardless of finding count — group similar findings and search on shared key terms.
Build duplicate map before showing anything to the human. Mark potential duplicates ⚠️ before drafting begins.
If MCP unavailable: proceed without check, flag all as ⚠️ duplicate check skipped.

## S3 — Batch Draft
Draft all tickets simultaneously using REQ{} format from LAYER_2_FASTPATH S4.
QA-specific field rules:
t: BUG unless finding is clearly an improvement or missing feature.
env: staging unless finding states otherwise.
src: qa-testing always.
prd: include test case reference (TC-NNN) if present.
ac: "Given the fix, when [same steps], then [expected result]."
steps: "See test case [ref]" or "Steps not provided — QA to confirm" if not supplied.
tags: apply TEAM_CONTEXT.md Section 3 rules.
  UI/visual findings → frontend tag (ID: 2)
  API/data findings  → backend tag (ID: 44)
  Confirmed broken behaviour → bug tag (ID: 45)
  Unknown domain → bug tag only, domain tag added at triage.

## S4 — QA Review Table
Show after all tickets drafted:
QA BATCH — [N] findings | [N] clean | [N] flagged | [N] possible duplicates
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 #  ⭐  Title (50 chars)              Env      Flag
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Flag: — | ⚠️ incomplete — [what is missing] | ⚠️ duplicate of #[id] | ⚠️ check skipped
Commands: CONFIRM ALL / DETAIL [n] / EDIT [n] [field]=[value] / DROP [n] / FIX [n]
On CONFIRM ALL: pass all non-dropped REQ{} blocks to LAYER_2_FASTPATH S9.
Flagged tickets are included unless explicitly dropped — human decides.

## S5 — Incomplete Ticket Handling
If a finding is missing actual behaviour or expected behaviour: include in table, flag as ⚠️ incomplete — [what is missing].
Do not block creation. Human decides with FIX [n] or CONFIRM ALL.

## S6 — Post-creation Summary
QA SESSION COMPLETE
━━━━━━━━━━━━━━━━━━
✅ Created:  [N] tickets
⏭️  Skipped:  [N]
❌ Failed:   [N] — saved to PARKING_LOT.md
⚠️  Review:   [N] possible duplicates — verify in Odoo
Nothing else after this block.
