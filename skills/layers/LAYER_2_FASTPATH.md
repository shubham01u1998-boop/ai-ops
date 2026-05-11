# LAYER_2_FASTPATH — Fast Path Intake | Inherits: LAYER_0_GLOBAL. For: developers and QA daily intake.

## S1 — Classifier
Score silently before acting: Completeness 0-100% | Complexity low/med/high | Ambiguity low/med/high | Stakes low/med/high.
Completeness >60% AND complexity low AND stakes low → Fast path (this file).
Completeness <60% OR ambiguity high → PRE_PROCESSOR (not built — ask one question instead).
Complexity high OR stakes high → DISCOVERY (not built — ask two). Input is a question → S6.
High confidence: route silently. Medium: route + one-line declaration. Low: ask one question with two concrete options.
Never mention skill file names to the human.

## S2 — Extraction Pass
Before asking, extract from: document headers/metadata, section titles, TEAM_CONTEXT.md routing and priority rules, prior session decisions. Ask only if genuinely cannot be inferred.

## S3 — Assumption Surfacer
For requirements >60% complete, find the two most likely implicit assumptions that could cause implementation divergence. Surface in clarifying question if needed; otherwise flag: ⚠️ Assumed: [assumption].

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
Table maintains insertion order always — never re-sorts after edits.
On CONFIRM ALL: pass all REQ{} blocks to S9 — fetch Odoo context, run field mapping, create tickets sequentially.

## S6 — Question Handler
Type 1 — Resolvable: answer from TEAM_CONTEXT.md or existing tickets. State answer and proceed. Ask only if unresolvable.
Type 2 — Unresolvable: name the decision, offer (A) spike ticket (B) park in PARKING_LOT.md (C) TBD placeholder flagged for review.
Type 3 — Disguised requirement: state the implied requirement explicitly, ask for one-word confirmation before proceeding.
Type 4 — Genuine open question: do not create a ticket. Offer PARKING_LOT.md or discussion. Never force into a ticket.

## S7 — Multi-Requirement Detection
Detect multiple distinct requirements before processing. State: "I found [N] requirements — processing in order."
Process sequentially — complete one fully before starting the next. Show one batch review table for all at the end.

## S8 — Duplicate Check
Run AFTER REQ{} is fully assembled. Use finalised ti: as primary search term.
Run in parallel: search_tickets(first 3 words of ti:, limit=5) | search_tickets(most distinctive noun from what:, limit=5) | search_tickets(component name if identifiable, limit=5).
Also check: was a ticket with this title created earlier in this session? If yes: flag ⚠️ already created this session — #[id]. Do not create again.
If overlap found: flag ⚠️ possible duplicate of #[id] — [title]. Do not block — flag only, human decides.
If MCP unavailable: flag ⚠️ duplicate check skipped — verify manually.

## S9 — Inline Mapper (Phase 1)
On CONFIRM ALL, for each REQ{} block:
1. Fetch once per session: list_projects() | list_tags() | list_users() | list_stages(project_id=58) — cache, do not re-fetch per ticket.
2. Map fields using TEAM_CONTEXT.md routing rules. assignee_ids: always empty at creation — never use reporter as assignee. Assignment happens during triage only.
3. Set stage_id to Backlog (ID: 347) always.
4. Set priority: apply TEAM_CONTEXT.md Section 4 rules.
5. Build description HTML from REQ{} fields before calling create_ticket():
   <h3>What is happening</h3><p>[what]</p>
   <h3>Expected behaviour</h3><p>[expect]</p>
   <h3>Steps to reproduce</h3><p>[steps]</p>
   <h3>Acceptance criteria</h3><p>[ac]</p>
   <h3>Impact</h3><p>[impact]</p>
   <h3>Context</h3><p>Reported by: [by] | Source: [src] | Env: [env]</p>
   Always pass this pre-built HTML string as the description parameter. Never pass raw text.
6. Show compact preview: #[n] [title] | [project] | [priority] | [tags] — CONFIRM / EDIT / SKIP
7. On CONFIRM: call create_ticket(). Report: ✅ #[odoo_id] created → [url]
   On fail: save to PARKING_LOT.md. Report: ❌ #[n] failed → saved to parking lot.
8. After all tickets: [N] created | [N] skipped | [N] failed — check PARKING_LOT.md
