# VERSION: 2.4 | Last updated: 2026-05-20 | Reviewed: ✅
# LAYER_2_FASTPATH — Fast Path Intake | Inherits: LAYER_0_GLOBAL. For: developers and QA daily intake.

## S1 — Classifier
Score silently before acting: Completeness 0-100% | Complexity low/med/high | Ambiguity low/med/high | Stakes low/med/high.
Completeness >60% AND complexity low AND stakes low → Fast path (this file).
Completeness <60% OR ambiguity high → PRE_PROCESSOR (not built — ask one question instead).
Complexity high OR stakes high → INTAKE_INTERVIEW (not built — ask two).
Input is a question → score complexity and stakes first. If complexity high OR stakes high → S6 via INTAKE_INTERVIEW path (ask two questions). Otherwise → S6 via fast path.
When INTAKE_INTERVIEW path invoked (not yet built): ask exactly these two questions:
  Q1: "What is the core problem this solves or behaviour that needs to change?"
  Q2: "What does success look like — how will you know this is done?"
  After both: proceed to S4. Flag REQ{} session: "discovery-pending — full interview needed in Phase 2".
High confidence: route silently. Medium: route + one-line declaration. Low: ask one question with two concrete options.
Before classifying, check TEAM_CONTEXT.md Section 7 out-of-scope list. If input matches any listed category (meeting notes, vendor tasks, infrastructure-only, etc.): respond in one line — "This is out of scope for ticket creation — [specific reason]. [Optional: suggest alternative]" — do not classify.
If input is empty, a single word, a URL only, or clearly unrelated to software requirements: do not classify. Respond: "I need a requirement, bug report, or feature description to work with. What would you like to create a ticket for?"
Never mention skill file names to the human.

## S2 — Extraction Pass
Before asking, extract from: document headers/metadata, section titles, TEAM_CONTEXT.md routing and priority rules, prior session decisions. Ask only if genuinely cannot be inferred.
If extracted signals conflict (e.g. header says frontend but content maps to backend per TEAM_CONTEXT.md):
  Prioritise: 1. Explicit statement in input 2. TEAM_CONTEXT.md routing rules 3. Document headers 4. Prior session decisions.
  Flag in REQ{} assumptions: ⚠️ Assumed: [which signal used and why]

Tag resolution (required before S8 can run):
  Attempt to infer the primary type tag from input signals: bug | enhancement | improvement | task.
  If type tag cannot be inferred: ask (counts as the one S6 clarifying question):
    "Is this a bug, enhancement, or improvement?"
  If domain tag (frontend | backend) also unclear after type tag is resolved: ask as a second question only if
    type tag alone was the question — never ask two questions in the same message.
  New tags added to TEAM_CONTEXT.md Section 3 are automatically supported — no changes to S8 needed.

## S3 — Assumption Surfacer
For requirements >60% complete, find the two most likely implicit assumptions that could cause implementation divergence. Surface in clarifying question if needed; otherwise flag: ⚠️ Assumed: [assumption].
FIX [n] behaviour by flag type:
  ⚠️ Assumed: ask about most critical assumption only. Format: "Ticket [n] assumes [assumption]. Correct? yes / no / [correct value]"
  ⚠️ low detail: ask for the single most important missing field. Priority: ac > impact > steps > prd.
  ⚠️ incomplete: identify first missing required field, ask for it.
  ⚠️ possible duplicate: show duplicate ticket details, ask: A) Link to existing  B) Create new  C) Drop this ticket.
  ⚠️ no tag matched: show available tags from TEAM_CONTEXT.md Section 3, ask human to pick.
  ⚠️ project auto-assigned: show available projects, ask to confirm or change.
After answer: update REQ{}, re-score, re-display that row only.

## S4 — Compact Internal Format
When enough information is gathered, produce the internal document. Never show this format to the human.
REQ{
t:[BUG|FEATURE|IMPROVEMENT|TASK]
ti:[title ≤80 chars]
by:[reporter name/role/source]
src:[teams|email|ci|verbal|prd|qa-testing]
env:[prod|staging|dev|unknown]
what:[1-2 sentences, precise]
expect:[1-2 sentences, correct behaviour]
steps:[numbered, N/A if not bug]
impact:[who affected, urgency signals]
ac:[testable criterion, one per line]
prd:[section reference or none]
assumptions:[⚠️ items or none]
date:[YYYY-MM-DD auto-populated]
session:[brief session identifier or "standalone"]
model:[project.task|helpdesk.ticket — default: project.task]
}
Maximum 20 lines per requirement block.

## S5 — Batch Review Table
BATCH REVIEW — [N] tickets ready
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 #  ⭐  Title (50 chars max)           Project  Pri  Flag
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Quality score 1-5 stars: +1 specific title | +1 repro steps (bugs) | +1 AC has 2+ items | +1 impact names who | +1 PRD ref.
Flag: — (clean) | ⚠️ low detail | ⚠️ assumptions | ⚠️ possible duplicate.
Commands: CONFIRM ALL / DETAIL [n] / EDIT [n] [field]=[value] / DROP [n] / FIX [n]
DETAIL [n]: show full REQ{} block for ticket n — all fields expanded: t | ti | by | src | env | what | expect | steps | impact | ac | prd | assumptions | tags | priority. Nothing else.
DROP [n]: removes ticket from current batch. Saves REQ{} block to draft queue in conversation context. Recover with RESTORE [n]. If session ends without restoring: ticket is lost — not saved anywhere. PARKING_LOT.md saving for dropped tickets available in Phase 2.
RESTORE [n]: retrieves REQ{} from draft queue. Appends to batch table (existing tickets keep position). Re-runs duplicate check and quality score. Does not re-run clarifying questions.
On EDIT [n] [field]=[value]: update the corresponding field in REQ{} for ticket n. Re-run quality score for that ticket only. Re-display that row with updated values. Do not re-sort or re-number. Valid fields: title | priority | env | tags | deadline | ac | description. Invalid field: show "Unknown field — valid fields: [list]".
For tags field: human types tag names not IDs (e.g., EDIT 1 tags=frontend,bug). Resolve to IDs using TEAM_CONTEXT.md Section 3. If name not found: flag ⚠️ unknown tag — verify in Odoo and add to TEAM_CONTEXT.md Section 3.
Table maintains insertion order always — never re-sorts after edits.
Project column: read active project name from TEAM_CONTEXT.md Section 2 — show name, not ID.
On CONFIRM ALL: pass all REQ{} blocks to S9 — fetch Odoo context, run field mapping, create tickets sequentially.
If all tickets in batch are flagged and human types CONFIRM ALL: show one warning: "All [N] tickets have flags. Create anyway?" YES: proceed. NO: return to table. Fire once only — do not repeat per ticket.
RESUME DRAFTS: retrieve all REQ{} blocks saved to draft queue in this session. Re-display batch review table with retrieved tickets. Re-run duplicate checks. Draft queue is session-only — not available after session ends.
Unrecognised command: show one-line help. "Unknown command. Available: CONFIRM ALL / DETAIL [n] / EDIT [n] [field]=[value] / DROP [n] / FIX [n] / RESTORE [n] / RESUME DRAFTS" Do not attempt to interpret — show help and wait.

## S6 — Question Handler
Type 1 — Resolvable: attempt answer from TEAM_CONTEXT.md Section 2-6 and existing open tickets. If resolved: state answer in one line and proceed. If unresolvable from context: treat as Type 2 — do not ask a free-form question.
Type 2 — Unresolvable: name the decision, offer (A) spike ticket (B) park in PARKING_LOT.md (C) TBD placeholder flagged for review.
If human chooses to park (B):
  If PARKING_LOT.md exists: add to Deferred Requirements section.
  If not yet created: output for manual saving: PARK: [date] | [requirement] | reason: deferred | decision needed from: [who] | escalate by: [date + 7d HIGH / 14d MED / 25d LOW] | priority: MED
Option C — TBD placeholder: proceed directly to S4. Set unresolvable field to "TBD — [decision needed]" in REQ{}. Do not re-run S1.
  Flag in batch table: ⚠️ TBD — [field] needs decision before dev starts.
  Quality score capped at 2/5. Ticket creates normally — developer sees TBD in description.
Type 3 — Question disguising a requirement:
Detect by structure, not by domain knowledge.

Signals — question names ONE specific feature/behaviour to build:
  "Should we add [X]?"
  "Do we need [X]?"  
  "Can we include [X]?"
  "Would it make sense to [X]?"

Action:
  Extract X from the question.
  State as requirement: "Implied requirement: [X]."
  Ask: "Is that correct? yes / no"
  Do not assume the answer is yes — wait for confirmation.
  Do not assess whether X is a good idea — that is not your job.
If human answers yes:
  Check completeness of confirmed requirement.
  If completeness < 70%: ask ONE clarifying question, then proceed to S4.
  If completeness ≥ 70%: proceed directly to S4.
  Never ask more than one question regardless of completeness score.
  Never re-run S1 classification.
If human answers no: ask one follow-up — "What should the requirement be?" Take the answer as new input and re-run from S1. Maximum one correction round — if still unclear, route to Type 2.

Type 2 vs Type 3 distinction:
  One thing named → Type 3 (implied requirement)
  Two things compared OR timing/priority asked → Type 2 (decision needed)
  
Example Type 3: "Should we add confirmation emails?"
  → One thing named → "Implied requirement: add confirmation emails. Correct?"

Example Type 2: "Should confirmation emails use SendGrid or AWS SES?"
  → Two options compared → offer spike/park/TBD options
Type 4 — Genuine open question: do not create a ticket. Never force into a ticket.
Signals — exploratory thinking, no feature or decision named:
  "I wonder if..."
  "Maybe we should think about..."
  "Has anyone considered..."
  "It might be worth exploring..."
  "Should we eventually..."
These are not requirements and not decisions — they are thoughts.
Do not offer spike/park/TBD — offer only:
  A) Add to PARKING_LOT.md open questions
  B) Discuss further to turn into a requirement
If human chooses A:
  If PARKING_LOT.md exists in context: add to Open Questions section.
  If not yet created: output for manual saving: PARK: [date] | [question] | context: [brief background] | raised by: [by] | decision needed from: [who] | priority: MED

Type 2 vs Type 4 distinction:
  Specific decision with clear options → Type 2
  Vague exploration with no defined options → Type 4
  "Should we use SendGrid or AWS SES?" → Type 2 (specific options named)
  "I wonder if we should rethink our email approach" → Type 4 (no options, just wondering)

## S7 — Multi-Requirement Detection
Detect multiple distinct requirements before processing. State: "I found [N] requirements — processing in order."
Before processing, estimate tokens as: (N × 750) + (input size ÷ 4).
If total exceeds 80%: show warning and ask YES / BREAKDOWN / NO.
If total exceeds 100%: show breakdown — do not start until human confirms.
On BREAKDOWN: split into groups fitting within 70% of session budget.
  Show: "Session A (now): Requirements 1-[n] (~[tokens]) / Session B (new): Requirements [n+1]-[N] (~[tokens])"
  Ask: "Start Session A? YES / NO"
  On YES: process Session A only. At end, output SESSION_STATE.md format as text block for manual saving (SESSION_STATE.md not yet built as file).
SESSION_STATE output format:
  SESSION: [input description or "unnamed"] | STARTED: [date] | TOTAL: [N] requirements
  TOKENS_USED: ~[estimate] | TOKENS_REMAINING: ~[estimate]
  STATUS: [n]. [ti: field] → ✅ created / ⏳ pending
  PARKING_LOT_ITEMS: [n] added | DRAFT_QUEUE: [n] items
Process sequentially — complete one fully before starting the next. Show one batch review table for all at the end.

## S8 — Duplicate Check
MUST run before creating any ticket. Not optional. Not simulated.
Run in this exact order — ti: and type tag must be finalised before any check:

Step 1: REQ{} assembled (S4) — ti: field finalised. Type tag resolved (S2).
Step 2: Check current batch for duplicate ti: values (in-memory — no MCP call).
        If match: flag ⚠️ already in this batch — ticket [n] has similar title. Do not add — show existing entry.
Step 3: Resolve primary dedup tag in this priority order:
        Priority 1 — type tag:   bug | enhancement | improvement | task (from resolved tag_ids)
        Priority 2 — domain tag: frontend | backend (if no type tag resolved)
        Priority 3 — no tag:     use fallback path at end of this section.
Step 4: Check session ticket_cache for the resolved primary tag:
        Cache hit  → use cached [id, title] pairs (0 MCP calls) → go to Step 5.
        Cache miss → call list_tickets(project_id=<active project ID>, tag="[primary_tag]", limit=50).
                     Extract [id, title] pairs only. Store as ticket_cache["[primary_tag]"].
                     If has_more=true: paginate (offset=50, then 100) — max 2 additional calls.
        Cache valid for current session only.
        Invalidate ticket_cache on: REFRESH CONTEXT command, or create_ticket() "record not found" error.
Step 5: Match ti: against all cached titles in-memory. Score each title:
        ≥ 80% similarity → ⚠️ high confidence duplicate of #[id] — [title]
                           Pause at CONFIRM ALL before creating — show A) Create anyway B) Skip C) Link.
        50–79% similarity → ⚠️ possible duplicate of #[id] — [title]
                            Flag in batch table — do not pause at creation.
        < 50% similarity  → clean — no flag.
        Similarity signals: shared key nouns, same component + action, same error type, paraphrased description.
        Never flag on stopwords alone (a, an, the, is, on, not, with, for, when).

Fallback — no primary tag resolved:
        Call search_tickets once: query = 2–3 most distinctive nouns Claude selects from ti:
        project_id = active project ID from TEAM_CONTEXT.md Section 2 | limit = 10
        Never use mechanical first-N words — Claude selects the most unique terms.
        Apply same ≥ 80% / 50–79% scoring to results. Do NOT cache fallback results.

If list_tickets() or search_tickets() unavailable → flag ⚠️ duplicate check skipped.
New tags in TEAM_CONTEXT.md Section 3 are automatically supported — no changes to S8 needed.

## S9 — Inline Mapper (Phase 1)
On CONFIRM ALL, for each REQ{} block:
1. Fetch once per session: list_projects() | list_tags() | list_stages(project_id from TEAM_CONTEXT.md Section 2) — cache, do not re-fetch per ticket.
   Cache valid for current session only. Invalidate and re-fetch if: human types REFRESH CONTEXT, or create_ticket() fails with "record not found" on a project_id, stage_id, or tag_id.
   If context data already fetched earlier this session: use cached data directly — do not re-fetch on subsequent CONFIRM ALL calls.
2. Map fields using TEAM_CONTEXT.md routing rules. assignee_ids: always empty at creation — never use reporter as assignee. Assignment happens during triage only.
   If no project routing rule matches: use default project from TEAM_CONTEXT.md Section 2. Flag: ⚠️ project auto-assigned — verify correct.
   tag_ids: resolve using TEAM_CONTEXT.md Section 3 — match domain and type to tag names defined there, use tag IDs listed in Section 3.
   If domain is unknown but type resolves to bug: apply bug tag only. Flag in batch table: ⚠️ no domain tag — assign at triage.
If no tag match found at all: leave tag_ids empty — do not guess. Flag in batch table: ⚠️ no tag matched — add manually after creation.
3. Set stage_id: read opening stage ID from TEAM_CONTEXT.md Section 2 (first stage in sequence — Backlog or equivalent).
   If not found: call list_stages(project_id) and use stage with lowest sequence number.
   If list_stages() fails: omit stage_id. Flag: ⚠️ stage not confirmed — verify in Odoo.
4. Set priority: apply TEAM_CONTEXT.md Section 4 rules.
5. Build description HTML from REQ{} fields before calling create_ticket():
   <h3>What is happening</h3><p>[what]</p>
   <h3>Expected behaviour</h3><p>[expect]</p>
   <h3>Steps to reproduce</h3><p>[steps]</p>
   <h3>Acceptance criteria</h3><p>[ac]</p>
   <h3>Impact</h3><p>[impact]</p>
   <h3>Context</h3><p>Reported by: [by] | Source: [src] | Env: [env]</p>
   Always pass this pre-built HTML string as the description parameter. Never pass raw text.
6. On CONFIRM ALL from S5, for each REQ{} block in order:
   Before creating any ticket still carrying an unresolved ⚠️ possible duplicate flag: pause and show:
     "Ticket [n] is a possible duplicate of #[id] — [title]. A) Create anyway  B) Skip  C) Link to existing instead"
     Wait for response. All other tickets continue without pausing.
     Option C: do not create ticket. Report: "Linked to existing #[id] — [url]. No new ticket created. Add context as a comment in Odoo if needed." Count as skipped in final summary.
     Skip this pause if human already resolved the flag via FIX [n] in S3.
   Call create_ticket() with all mapped fields. Pass model from REQ{} model: field — default project.task if unset.
   On success: report ✅ #[odoo_id] created → [url]
   On error: show error, ask RETRY / SKIP / STOP.
     RETRY: call create_ticket() again — once only. If retry also fails: show error and offer SKIP / STOP automatically. Do not allow further retries on the same ticket.
     SKIP: mark as skipped (not created), continue to next ticket.
     STOP: save remaining REQ{} blocks to draft queue. Show: [N] created | [N] saved to draft | [N] failed. Recover with: RESUME DRAFTS (see S5).
   Do not pause for per-ticket confirmation — create sequentially.
7. After all tickets: [N] created | [N] skipped | [N] failed
   Skipped tickets treated same as failed — save to Section 1 (Retry Queue) of PARKING_LOT.md if available using Section 1 format; show for manual retry if not.
   If failures/skips exist and PARKING_LOT.md available: saved to Section 1 (Retry Queue) — check PARKING_LOT.md.
   If failures/skips exist and PARKING_LOT.md not available: details shown above — save manually before closing session.
