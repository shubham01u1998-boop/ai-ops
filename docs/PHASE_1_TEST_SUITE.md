# Phase 1 Test Suite — Ticket Intake (TiffinConnect)
# Where to run: Claude Enterprise → "Ticket Intake — TiffinConnect" project
# Version: 1.0 | Date: 2026-05-19
# Format: TC-NNN | Input → Expected behaviour → Pass criterion

---

## How to Run

1. Open Claude Enterprise → **Ticket Intake — TiffinConnect** project
2. Start a fresh conversation for each group (avoids session state bleed)
3. Paste the **Input** exactly as written
4. Compare Claude's response to **Expected** and mark Pass / Fail
5. For creation tests: verify the ticket appears in Odoo at project ID 58

---

## GROUP 1 — Routing & Classification (S1 + LAYER_0_GLOBAL Routing Rule)

---

**TC-001** | Single complete bug → fast path, no questions
```
Input:
  Login button unresponsive on iPhone 14 iOS 17.
  Works on Android. All iOS users blocked. Reported by Shubham.

Expected:
  Routes silently to LAYER_2_FASTPATH.
  Shows batch review table — does NOT ask any clarifying questions first.

Pass if: Table appears without any question being asked.
```

---

**TC-002** | Vague single bug → asks exactly one question
```
Input:
  The checkout is broken.

Expected:
  Asks exactly one question — the single most critical missing piece
  (likely: "What is happening vs what should happen?").
  Does NOT ask about assignee, priority, or project.

Pass if: Exactly one question asked. Not two. Not zero.
```

---

**TC-003** | High-stakes input → asks exactly two questions
```
Input:
  We need to redesign the entire authentication system to support SSO,
  OAuth, and multi-tenant login across all platforms before the next release.

Expected:
  Scores complexity: high. Routes to DISCOVERY path.
  Asks exactly:
    Q1: "What is the core problem this solves or behaviour that needs to change?"
    Q2: "What does success look like — how will you know this is done?"
  Flags session: "discovery-pending — full interview needed in Phase 2"

Pass if: Both Q1 and Q2 appear verbatim. No other questions asked.
```

---

**TC-004** | Empty input → rejection message
```
Input:
  (send a blank message or just a space)

Expected:
  "I need a requirement, bug report, or feature description to work with.
   What would you like to create a ticket for?"

Pass if: Exact rejection message shown. No ticket drafted.
```

---

**TC-005** | Single word input → rejection message
```
Input:
  Bug

Expected:
  "I need a requirement, bug report, or feature description to work with.
   What would you like to create a ticket for?"

Pass if: Rejection message shown. No batch table or question shown.
```

---

**TC-006** | "QA batch" keyword → routes to QA_INTAKE
```
Input:
  QA batch from today's session:
  1. Login button broken on iOS
  2. Cart total shows wrong amount
  3. Profile image not saving

Expected:
  Routes to QA_INTAKE (not LAYER_2_FASTPATH).
  Shows "Found 3 findings — processing." header.
  Drafts all 3 as BUG type, src: qa-testing, env: staging.

Pass if: QA_INTAKE header format shown (not FASTPATH batch table header).
```

---

**TC-007** | Numbered list without "QA batch" keyword → routes to QA_INTAKE
```
Input:
  1. Payment API returns 500 on empty card details
  2. Logout button missing from profile screen

Expected:
  Routes to QA_INTAKE (numbered list trigger).
  Shows "Found 2 findings — processing."

Pass if: QA_INTAKE flow triggered, not FASTPATH.
```

---

**TC-008** | TC-NNN format → routes to QA_INTAKE
```
Input:
  TC-042: Login button unresponsive on iPhone 14 iOS 17 — FAIL
  TC-043: Cart total correct after promo code — PASS
  TC-044: Profile image not saving on Android — FAIL

Expected:
  Routes to QA_INTAKE.
  Processes TC-042 and TC-044 as findings (FAIL).
  Skips TC-043 silently (PASS).
  States: "Found 2 findings — processing." (not 3)

Pass if: TC-043 is silently skipped, only 2 findings drafted.
```

---

## GROUP 2 — Fast Path Single Requirement (S2, S3, S4)

---

**TC-009** | Reporter name auto-extracted from input
```
Input:
  The order summary screen shows the wrong total when a promo code
  is applied. Reported by Tanu.

Expected:
  REQ{} by: field populated as "Tanu" without asking.
  No question asked about reporter.

Pass if: Reporter not asked. Batch table shows ticket with no "reporter" flag.
```

---

**TC-010** | Environment auto-extracted
```
Input:
  On staging, the API returns a 404 for /api/orders when the
  user has no past orders. Expected: empty list response.

Expected:
  REQ{} env: staging — extracted silently.
  No question asked about environment.

Pass if: env not asked. Ticket description includes "staging" in Context section.
```

---

**TC-011** | Production environment → urgent priority auto-applied
```
Input:
  On production, users cannot complete payment — stripe integration
  throwing 500 errors. All users affected. Reported by Shubham.

Expected:
  REQ{} priority: 1 (urgent) — applied from TEAM_CONTEXT Section 4 rule:
  "bug + production environment → urgent (1)"

Pass if: Priority shown as urgent/high in batch table. Not asked.
```

---

**TC-012** | Frontend bug → tags: frontend + bug
```
Input:
  The login button on the home screen has the wrong colour —
  should be #FF5733 but shows as grey. iOS and Android both affected.
  Reported by Vijay.

Expected:
  Tags resolved: frontend (ID 2) + bug (ID 4).
  No domain question asked.

Pass if: Both tags present in created ticket (verify in Odoo).
```

---

**TC-013** | Backend bug → tags: backend + bug
```
Input:
  The /api/profile endpoint returns a 403 for valid authenticated users.
  Backend issue confirmed. Reported by Kunal.

Expected:
  Tags resolved: backend (ID 44) + bug (ID 4).

Pass if: Both tags present in created ticket (verify in Odoo).
```

---

**TC-014** | Unknown domain → bug tag only, no domain tag
```
Input:
  The tiffin delivery tracking is not updating in real time.
  Not sure if frontend or backend. Reported by Shubham.

Expected:
  Tag: bug (ID 4) only.
  Batch table flag: ⚠️ no tag matched — domain added at triage.

Pass if: Only bug tag. Domain tag not guessed.
```

---

**TC-015** | Assumption surfaced for >60% complete requirement
```
Input:
  Add email notifications when a tiffin order is confirmed.
  Users should receive an email immediately after placing an order.

Expected:
  Two implicit assumptions surfaced.
  Likely: email template design, or whether to use existing email service.
  Flagged as ⚠️ Assumed: [assumption].

Pass if: At least one ⚠️ Assumed flag visible in batch table or output.
```

---

## GROUP 3 — Batch Review Table Commands (S5)

Run TC-001 first to get a ticket into the table, then run these commands.

---

**TC-016** | DETAIL [n] → shows full REQ{} fields
```
After TC-001 batch table appears:
Input: DETAIL 1

Expected:
  Shows full ticket detail — all REQ{} fields expanded.
  Includes: what, expect, steps, impact, ac, env, tags, priority.

Pass if: All REQ{} fields visible. Not just title.
```

---

**TC-017** | EDIT [n] title=new title → updates title, re-scores
```
After TC-001 batch table appears:
Input: EDIT 1 title=Login button broken iOS 17

Expected:
  Title updated in REQ{} for ticket 1.
  Quality score re-calculated for row 1 only.
  Row 1 re-displayed with updated title and new score.
  Other rows unchanged. Table not re-sorted or re-numbered.

Pass if: Only row 1 updates. Order preserved.
```

---

**TC-018** | EDIT [n] tags=frontend,bug → resolves tag names to IDs
```
Input: EDIT 1 tags=frontend,bug

Expected:
  frontend resolved to ID 2.
  bug resolved to ID 4.
  REQ{} tag_ids updated.
  No question asked about IDs.

Pass if: Tags updated silently using TEAM_CONTEXT Section 3 IDs.
```

---

**TC-019** | EDIT [n] with invalid field → shows field list error
```
Input: EDIT 1 reporter=Tanu

Expected:
  "Unknown field — valid fields: title | priority | env | tags | deadline | ac | description"

Pass if: Error shown. REQ{} not modified.
```

---

**TC-020** | DROP [n] → removes from batch, saves to draft queue
```
Input: DROP 1

Expected:
  Ticket 1 removed from batch table.
  Saved to draft queue in session context.
  Other tickets keep their position and numbers.

Pass if: Ticket removed. Remaining tickets not renumbered.
```

---

**TC-021** | RESTORE [n] → retrieves dropped ticket back into batch
```
After TC-020 (ticket 1 dropped):
Input: RESTORE 1

Expected:
  Ticket 1 re-appended to batch table (at end, not original position).
  Duplicate check re-run for restored ticket.
  Quality score re-calculated.

Pass if: Ticket reappears in table. Position at end (not position 1).
```

---

**TC-022** | RESUME DRAFTS → retrieves all dropped tickets this session
```
After dropping multiple tickets:
Input: RESUME DRAFTS

Expected:
  All dropped tickets from this session retrieved.
  Batch review table re-displayed with all retrieved tickets.
  Duplicate checks re-run.

Pass if: All dropped tickets visible again.
```

---

**TC-023** | CONFIRM ALL → creates all tickets sequentially
```
After batch table with 2 clean tickets:
Input: CONFIRM ALL

Expected:
  Rule 1 (permission) already satisfied by CONFIRM ALL.
  Both tickets created in Odoo sequentially.
  Each reported: ✅ #[id] created → [url]
  Final: [2] created | [0] skipped | [0] failed

Pass if: Both tickets visible in Odoo project 58. Stage = Backlog (347).
         Assignee field EMPTY on both.
```

---

**TC-024** | CONFIRM ALL with all flagged tickets → shows one warning only
```
Setup: Get a batch where all tickets have ⚠️ flags.
Input: CONFIRM ALL

Expected:
  "All [N] tickets have flags. Create anyway?"
  Shows this warning ONCE only. Not per ticket.

Pass if: Warning shown once. Responds to YES by creating. Responds to NO by returning to table.
```

---

**TC-025** | Unrecognised command → one-line help
```
Input: GO

Expected:
  "Unknown command. Available: CONFIRM ALL / DETAIL [n] / EDIT [n] [field]=[value]
   / DROP [n] / FIX [n] / RESTORE [n] / RESUME DRAFTS"
  Nothing else.

Pass if: One line of help shown. No attempt to interpret "GO".
```

---

## GROUP 4 — Question Handler (S6)

---

**TC-026** | Type 1 — resolvable question, answered from TEAM_CONTEXT
```
Input:
  Who is the QA lead for TiffinConnect?

Expected:
  "Shubham Upadhyay (Odoo ID: 42)"
  Answered in one line from TEAM_CONTEXT Section 1.
  No ticket created. Does not ask a follow-up.

Pass if: Correct answer in one line. No ticket flow triggered.
```

---

**TC-027** | Type 2 — decision between two options → spike/park/TBD
```
Input:
  Should we use Firebase or AWS SNS for push notifications?

Expected:
  Offers: A) spike ticket  B) park in PARKING_LOT.md  C) TBD placeholder
  Does NOT pick an option or give an opinion.

Pass if: All three options offered. No preference stated.
```

---

**TC-028** | Type 2 → choose TBD placeholder → ticket created with TBD field
```
After TC-027:
Input: C

Expected:
  Proceeds to S4 with unresolvable field set to "TBD — notification provider needs decision"
  Quality score capped at 2/5.
  Batch table flag: ⚠️ TBD — notification provider needs decision before dev starts.

Pass if: TBD visible in REQ{}. Quality score ≤ 2/5.
```

---

**TC-029** | Type 3 — "Should we add X?" → implied requirement extracted
```
Input:
  Should we add a confirmation email when a user places an order?

Expected:
  "Implied requirement: add confirmation email when order is placed. Correct?"
  Waits for yes/no. Does NOT assume yes. Does NOT evaluate if it's a good idea.

Pass if: Exact implied requirement stated. Awaits confirmation.
```

---

**TC-030** | Type 3 confirmed yes, ≥70% complete → goes directly to S4
```
After TC-029:
Input: yes

Expected:
  Completeness scored ≥70% (enough info to draft).
  Goes directly to S4 → batch review table.
  Does NOT ask a clarifying question.

Pass if: Batch table shown without any question.
```

---

**TC-031** | Type 3 confirmed yes, <70% complete → one question then S4
```
Input:
  Should we add push notifications?

Then: yes

Expected:
  Completeness scored <70% (missing: when, who, what triggers).
  Asks ONE clarifying question only.
  After answer: goes to S4. No further questions.

Pass if: Exactly one question asked after "yes". Not two.
```

---

**TC-032** | Type 3 confirmed no → asks "What should the requirement be?"
```
After TC-029:
Input: no

Expected:
  "What should the requirement be?"
  Takes the answer as new input and re-runs from S1.
  Maximum one correction round.

Pass if: Single follow-up question. Takes new input and re-classifies.
```

---

**TC-033** | Type 4 — exploratory thought → park or discuss only (no ticket)
```
Input:
  I wonder if we should eventually rethink how we handle user sessions.

Expected:
  Offers only: A) Add to PARKING_LOT.md open questions  B) Discuss further
  Does NOT offer spike ticket, TBD, or start intake flow.

Pass if: Only two options shown. No REQ{} drafted.
```

---

## GROUP 5 — Multi-Requirement Detection (S7)

---

**TC-034** | Two requirements in one input → processed in order, one batch table
```
Input:
  1. The login button on iOS is unresponsive — should trigger login flow.
     Reported by Shubham. Staging.
  2. The order API returns 500 on empty cart. Backend issue. High priority.
     Reported by Kunal. Staging.

Expected:
  "Found 2 requirements — processing in order."
  Both drafted sequentially.
  ONE batch review table shown at the end with both tickets.

Pass if: Single batch table with 2 rows. Not two separate tables.
```

---

**TC-035** | Token estimate shown when input is large (>500 tokens)
```
Input: Paste a requirement description that is clearly large (5+ paragraphs,
       multiple features combined — simulate a PRD excerpt).

Expected:
  Token estimate block shown BEFORE processing:
    Task:    [what is happening]
    Tokens:  ~[number] estimated
    Budget:  [fits / tight / exceeds]
    Effects: [what will change]
    Proceed? YES / NO / BREAKDOWN

  Claude waits for response before starting.

Pass if: Estimate block shown. No processing started without YES.
```

---

## GROUP 6 — Duplicate Check (S8)

---

**TC-036** | Bug matching existing open ticket → ⚠️ possible duplicate flag
```
Input:
  Login button unresponsive on iOS — users can't log in on iPhone 14.
  Reported by Shubham.

  (Note: ticket #2514 with this exact title already exists in Odoo)

Expected:
  1 list_tickets(project_id=<active project>, tag=<primary tag>) call made; in-memory title matching against results.
  Batch table shows: ⚠️ possible duplicate of #2514 — Login button unresponsive on iOS mobile

Pass if: Duplicate flag present. Ticket #2514 referenced.
```

---

**TC-037** | CONFIRM ALL with duplicate flag → pauses for A/B/C choice
```
After TC-036 batch table with duplicate flag:
Input: CONFIRM ALL

Expected:
  Pauses on flagged ticket:
  "Ticket 1 is a possible duplicate of #2514 — Login button unresponsive on iOS mobile.
   A) Create anyway  B) Skip  C) Link to existing instead"
  Other clean tickets in batch continue without pausing.

Pass if: Pauses only on the duplicate ticket. Other tickets created normally.
```

---

**TC-038** | Choose option C (link to existing) → no new ticket created
```
After TC-037 duplicate prompt:
Input: C

Expected:
  "Linked to existing #2514 — [url]. No new ticket created.
   Add context as a comment in Odoo if needed."
  Counted as skipped in final summary.

Pass if: No new ticket in Odoo. #2514 referenced. Counted as skip.
```

---

**TC-039** | New unique bug → no duplicate flag, clean table entry
```
Input:
  The tiffin menu images are not loading on the menu screen.
  Only happens on slow connections. Reported by Vijay. Staging.

Expected:
  list_tickets(tag=<primary tag>) cache used from prior call; no new MCP call made.
  In-memory title match finds no match. Batch table entry shows — (clean flag, no ⚠️).

Pass if: Clean flag shown. No duplicate flag on a genuinely new ticket.
```

---

## GROUP 7 — Field Mapping & Ticket Creation (S9)

---

**TC-040** | Created ticket has empty assignee (never auto-assigned)
```
Input:
  The profile picture upload fails on Android. Reported by Vijay. Staging.
Input: CONFIRM ALL → YES

Expected:
  Ticket created in Odoo.
  Assignee field: EMPTY.
  Reporter (Vijay) NOT used as assignee.

Pass if: Verify in Odoo — ticket #[id] has no assignee.
```

---

**TC-041** | Created ticket stage = Backlog (ID 347)
```
After any successful CONFIRM ALL:

Expected:
  All created tickets land in Backlog stage (ID 347).

Pass if: Verify in Odoo — stage column shows "Backlog".
```

---

**TC-042** | Description formatted as HTML with correct h3 sections
```
After any successful CONFIRM ALL — view ticket in Odoo:

Expected:
  Description contains these sections as <h3> headings:
  - What is happening
  - Expected behaviour
  - Steps to reproduce
  - Acceptance criteria
  - Impact
  - Context (Reported by | Source | Env)

Pass if: All 6 h3 sections present in Odoo ticket description.
```

---

**TC-043** | Staging bug → normal priority (0) in Odoo
```
Input:
  Order total is wrong after applying a promo code. Staging only.
  Reported by Shubham.
Input: CONFIRM ALL → YES

Expected:
  Ticket created with priority: normal (0) per TEAM_CONTEXT Section 4.

Pass if: Odoo ticket shows normal/low priority (no star).
```

---

**TC-044** | Production bug → urgent priority (1) in Odoo
```
Input:
  Payment is failing for all users on production. Stripe 500 error.
  Reported by Shubham.
Input: CONFIRM ALL → YES

Expected:
  Ticket created with priority: urgent (1) per TEAM_CONTEXT Section 4.

Pass if: Odoo ticket shows urgent/high priority (starred).
```

---

## GROUP 8 — LAYER_0_GLOBAL Rules

---

**TC-045** | Rule 1 — permission required before any write action
```
Input any complete bug report.

Expected:
  Claude drafts the ticket and shows batch table.
  Does NOT create the ticket without CONFIRM ALL.
  States intended action and waits.

Pass if: No ticket appears in Odoo before CONFIRM ALL is sent.
```

---

**TC-046** | Rule 4 — clarifying question ≤ 3 lines
```
Input: The checkout is broken. (vague — triggers a question)

Expected:
  The clarifying question is maximum 3 lines long.

Pass if: Question response is 3 lines or fewer.
```

---

**TC-047** | Rule 5 — no narration, no input repetition
```
Input any bug report.

Expected:
  Claude does NOT say:
  - "I will now process this requirement..."
  - "You mentioned that the login button..."
  - "Great! Let me help you with that..."
  Output is new information only: table, question, or confirmation.

Pass if: None of those narration phrases appear in the response.
```

---

**TC-048** | Rule 6 — one clarifying question maximum in fast path
```
Input:
  Something is broken in the app.

Expected:
  Asks exactly ONE question — the single most critical missing piece.
  Not two questions in one response.
  Not "and also, what environment?"

Pass if: Single question only. Even if multiple things are unknown.
```

---

**TC-049** | Rule 7 — never asks same question twice in a session
```
Input bug report → Claude asks about environment → answer "staging"
Input second bug report in same session.

Expected:
  Claude does NOT ask about environment again.
  Applies "staging" from prior session decision.

Pass if: Environment not re-asked in the same session.
```

---

**TC-050** | Rule 8 — end of session → TEAM_CONTEXT learning proposals
```
After a session where a new field mapping decision was made
(e.g., mapping a new domain to a tag), close the session.

Expected:
  Before ending, Claude proposes:
  "SESSION LEARNINGS — add to TEAM_CONTEXT.md Section 5?
   [A] [date] | [decision] — Y/N"

Pass if: Proposal shown. Claude waits for Y/N. Does NOT update TEAM_CONTEXT without approval.
```

---

## GROUP 9 — QA Intake Full Flow (QA_INTAKE)

---

**TC-051** | Passing tests silently skipped
```
Input:
  TC-101: Login on Android — PASS
  TC-102: Cart total incorrect after discount — FAIL
  TC-103: Logout works correctly — PASS

Expected:
  "Found 1 finding — processing." (not 3)
  Only TC-102 drafted. TC-101 and TC-103 skipped silently.

Pass if: Count is 1. No mention of TC-101 or TC-103.
```

---

**TC-052** | All QA fields auto-set correctly
```
Input:
  QA batch:
  1. Profile image not saving on Android. Expected: image saves and displays.

Expected:
  REQ{} fields auto-set:
    t: BUG
    env: staging
    src: qa-testing
    steps: "Steps not provided — QA to confirm"

Pass if: All 4 fields match expected values. Not asked about any of them.
```

---

**TC-053** | TC-NNN reference stored in prd: field
```
Input:
  TC-204: Payment button disabled after first failed attempt — FAIL
  Expected: button re-enables after 3 seconds.

Expected:
  REQ{} prd: TC-204
  ac: "Given the fix, when payment button is tapped after failure,
       then button re-enables after 3 seconds."

Pass if: TC-204 in prd: field. ac: follows QA format.
```

---

**TC-054** | Vague finding → shown separately with ADD/DISCARD/PARK
```
Input:
  QA batch:
  1. Login broken.
  2. Cart total incorrect — shows $0 instead of correct amount. Staging.

Expected:
  Finding 1 ("Login broken") flagged as too vague.
  Before review table:
    "⚠️ 1 finding too vague to draft: 1. Login broken.
     Options: ADD 1 to add information now | DISCARD 1 | PARK 1"
  Revised count shown: "Found 2 total. 1 too vague. Processing 1 finding."

Pass if: Vague finding shown separately. Count adjusted. Only 1 ticket drafted.
```

---

**TC-055** | ADD [n] for vague finding → one question asked, then drafted
```
After TC-054 vague finding shown:
Input: ADD 1

Expected:
  Asks ONE targeted question for minimum viable information.
  After answer: drafts REQ{} and adds to batch table.

Pass if: Exactly one question. Ticket appears in table after answering.
```

---

**TC-056** | PARK [n] for vague finding → output for manual saving
```
After TC-054 vague finding shown:
Input: PARK 1

Expected:
  If PARKING_LOT.md available: appended to Section 4 (Vague Findings).
  If not available: outputs manual save block:
    PARK: [date] | Login broken | action: RESUBMIT | session: qa-session-2026-05-19 | priority: MED

Pass if: Parking output shown (or PARKING_LOT.md updated). Not silently discarded.
```

---

**TC-057** | Duplicate check batches in parallel for ≤5 findings
```
Input:
  QA batch:
  1. Login button broken on iOS
  2. Cart shows wrong total
  3. Profile image not saving

Expected:
  1 list_tickets(tag=<primary tag>) call made; results cached and reused for all 3 findings.
  In-memory title matching runs per finding. All duplicate flags built BEFORE showing the batch table.

Pass if: Batch table shown with duplicate flags pre-populated (not added after). Only 1 MCP call made (not 3).
```

---

**TC-058** | QA session complete summary block format
```
After CONFIRM ALL in a QA session with 2 created, 1 skipped:

Expected:
  QA SESSION COMPLETE
  ━━━━━━━━━━━━━━━━━━
  ✅ Created:  2 tickets
  ⏭️  Skipped:  1
  ❌ Failed:   0
  Nothing else after this block.

Pass if: Exact format shown. No trailing text or narration after the block.
```

---

## GROUP 10 — Edge Cases & Out-of-Scope

---

**TC-059** | Meeting notes → out-of-scope rejection
```
Input:
  Notes from today's standup:
  - Sahil to fix login button by Friday
  - Kunal to review API docs
  - Tanu to update staging environment

Expected:
  "Meeting notes are out of scope for ticket creation —
   consider adding action points as Task tickets instead."
  No ticket drafted.

Pass if: One-line rejection. No REQ{} or batch table shown.
```

---

**TC-060** | Pure vendor task → out-of-scope rejection
```
Input:
  Chase Stripe support to fix the webhook timeout issue on their end.

Expected:
  "This is out of scope for ticket creation — no internal dev work required.
   [Optional alternative suggested]"

Pass if: One-line rejection. No ticket drafted.
```

---

**TC-061** | URL-only input → rejection message
```
Input:
  https://fiftyfive-technologies-pvt-ltd.odoo.com/odoo/project/58/task/2514

Expected:
  "I need a requirement, bug report, or feature description to work with.
   What would you like to create a ticket for?"

Pass if: Rejection message. No ticket flow started.
```

---

**TC-062** | Creation failure → saved to PARKING_LOT / shown for manual save
```
Simulate: provide a malformed input that passes classification but
          triggers an MCP create_ticket() error.
          (e.g. use an invalid tag ID via EDIT before CONFIRM ALL)

Expected:
  Error shown: RETRY / SKIP / STOP offered.
  On SKIP: failure saved to PARKING_LOT.md Section 1 if available.
  If not available: full failure details shown for manual save.
  Not silently discarded.

Pass if: Failure explicitly surfaced. Not hidden.
```

---

**TC-063** | Conflicting signals → explicit input wins over TEAM_CONTEXT
```
Input:
  Frontend bug — but this is actually a backend API issue with the UI endpoint.
  Reported by Tanu. Staging.

Expected:
  Explicit statement ("backend API issue") overrides header keyword "frontend".
  Tags: backend + bug. Not frontend + bug.
  Batch table shows: ⚠️ Assumed: backend used — explicit statement overrides "frontend" keyword.

Pass if: backend tag applied. Assumption flagged.
```

---

## GROUP 11 — PARKING_LOT (Session-Start Checklist, Writing Rules, TTL)

> **Setup note:** Most tests in this group require manually backdating items in
> `context/PARKING_LOT.md` before starting a Claude Enterprise session.
> Use the format exactly as defined in the file header.
> Start a fresh session after each edit so the session-start checklist fires.

---

**TC-064** | Session-start: expired S1/S3/S4 items silently auto-cleaned
```
Setup:
  Add to Section 1 (date 8 days ago):
    → 2026-05-11 | Old failed ticket | reason: MCP error | retries: 0 | raised by: Shubham | priority: MED | action: RETRY
  Add to Section 3 (date 15 days ago):
    → 2026-05-04 | Should we add dark mode? | context: UI idea | raised by: Vijay | decision needed from: PM | priority: LOW
  Start a new session.

Expected:
  Both items deleted silently.
  "Auto-cleaned 2 expired items from PARKING_LOT." shown once.
  If N=0 on a clean run: no message at all.

Pass if: Message shows correct count. Items no longer in file.
         Section 2 expired items NOT silently cleaned (go to step 4c instead).
```

---

**TC-065** | Session-start: resolved items removed (step 2)
```
Setup:
  Add items with resolution states:
    → 2026-05-15 | Login bug | ... ✅ TICKETED #2514
    → 2026-05-15 | Cart bug  | ... ❌ DISCARDED
    → 2026-05-15 | Old req   | ... ⤴️ PROMOTED
  Start a new session.

Expected:
  All three lines removed silently at session start.
  No message shown for step 2 removal.

Pass if: All three lines gone from file after session start.
```

---

**TC-066** | Session-start: Last Reviewed unset → surfaces review reminder
```
Setup:
  Ensure line 2 of PARKING_LOT.md contains: Last Reviewed: (unset)
  Start a new session.

Expected:
  "PARKING_LOT has never been reviewed — schedule a review with QA Lead."

Pass if: Exact message shown. Session proceeds normally after.
```

---

**TC-067** | Session-start: warnings batched and sorted before display
```
Setup:
  Add to Section 1 (date 5 days ago, day 5):
    → 2026-05-14 | Failed ticket A | reason: error | retries: 0 | raised by: Shubham | priority: HIGH | action: RETRY
  Add to Section 2 (escalate-by = today or past, NOT expired):
    → 2026-05-05 | Auth SSO decision | reason: needs PM | decision needed from: PM | escalate by: 2026-05-19 | priority: MED | escalation sent: (absent)
  Start a new session.

Expected:
  Both collected BEFORE display.
  Sorted order: Section 2 escalation due (4a) first, then Section 1 (4d).
  Shows: "[2] PARKING_LOT items need attention: ..."
  Offers: YES / SKIP ALL / [n]

Pass if: Single batched block shown. Not two separate prompts.
         Escalation due item listed first regardless of section order.
```

---

**TC-068** | Session-start step 4a: Section 2 escalate-by passed → notification only, auto-advance
```
Setup:
  Add to Section 2 (date 20 days ago, escalate-by = yesterday):
    → 2026-04-29 | Push notification provider decision | reason: needs PM | decision needed from: PM | escalate by: 2026-05-18 | priority: MED
  Start session → on batch prompt, address this item (YES or [n]).

Expected:
  "ACTION NEEDED: notify PM (or QA Lead if PM is vacant) that
   'Push notification provider decision' requires a decision — escalate-by date has passed."
  No user choice required — Claude auto-advances to next item.

Pass if: Notification shown. No A/B/C choice offered. Auto-advances.
         QA Lead used if PM is vacant (PM role currently vacant).
```

---

**TC-069** | Session-start step 4c: Section 2 expired, x0 escalations → ESCALATE or DISCARD (no EXTEND)
```
Setup:
  Add to Section 2 (date 31+ days ago, no escalation sent field):
    → 2026-04-18 | Dark mode toggle decision | reason: product decision needed | decision needed from: PM | escalate by: 2026-05-02 | priority: LOW
  Start session.

Expected:
  "ACTION NEEDED: 'Dark mode toggle decision' has expired with no decision received.
   Escalate to PM (or QA Lead if vacant), or DISCARD?"
  Options: ESCALATE | DISCARD
  EXTEND not offered at x0.

Pass if: EXTEND NOT in the options. ESCALATE and DISCARD only.
         Cannot be skipped — SKIP ALL replaced with SKIP NON-REQUIRED.
```

---

**TC-070** | Session-start step 4c: x1 escalation → ESCALATE AGAIN / DISCARD / EXTEND offered
```
Setup:
  Add to Section 2 (31+ days old, 1 escalation sent):
    → 2026-04-18 | SSO implementation decision | reason: needs PM | decision needed from: PM | escalate by: 2026-05-02 | priority: HIGH | escalation sent: 2026-05-10 (x1)
  Start session → address expired item.

Expected:
  "FOLLOW-UP: 'SSO implementation decision' was escalated [N] days ago (x1 total).
   Escalate again / DISCARD / EXTEND (if extensions remain)"
  On ESCALATE AGAIN: escalation sent field updated to "2026-05-19 (x2)" in-place.

Pass if: All three options shown. Escalation field updated in-place on ESCALATE AGAIN.
```

---

**TC-071** | Session-start step 4c: x3 escalations → FINAL NOTICE, ESCALATE AGAIN not offered
```
Setup:
  Add to Section 2 (31+ days old, 3 escalations):
    → 2026-04-18 | Feature X decision | reason: needs PM | decision needed from: PM | escalate by: 2026-05-02 | priority: MED | escalation sent: 2026-05-15 (x3)
  Start session → address expired item.

Expected:
  "FINAL NOTICE: 'Feature X decision' has been escalated 3 times with no decision.
   Maximum escalations reached — EXTEND (if extensions remain) or DISCARD only."
  ESCALATE AGAIN NOT offered.

Pass if: "FINAL NOTICE" shown. Only EXTEND and DISCARD offered.
```

---

**TC-072** | Session-start step 4d: Section 1 day 5+ → RETRY/DISCARD/LATER, retry threshold check
```
Setup:
  Add to Section 1 (5 days old, retries: 3):
    → 2026-05-14 | Profile image upload bug | reason: MCP timeout | retries: 3 | raised by: Vijay | priority: MED | action: RETRY
  Start session.

Expected:
  Prepends: "Profile image upload bug has already failed 3 times."
  Then: "Retry item 'Profile image upload bug' expires in 2 days — RETRY / DISCARD / LATER"
  On RETRY: retries incremented to 4, date reset to today, ticket creation queued.

Pass if: "already failed 3 times" shown before the prompt. RETRY still offered (not auto-discarded).
```

---

**TC-073** | Session-start step 4d: RETRY queued → runs after checklist completes
```
Setup: same as TC-072, choose RETRY.

Expected:
  Full checklist completes first.
  Then: "Running 1 queued retry from PARKING_LOT S1:"
  Attempts ticket creation.
  On success: ✅ #[id] 'Profile image upload bug' created → [url]
  Item marked ✅ TICKETED #[id] in S1.

Pass if: Retry runs AFTER checklist, not during. Item updated on success.
```

---

**TC-074** | Session-start step 4e: Section 3 day 12+ → YES extends TTL, max 2 extensions
```
Setup:
  Add to Section 3 (12 days old):
    → 2026-05-07 | Should we support multiple languages? | context: UX idea | raised by: Tanu | decision needed from: PM | priority: LOW
  Start session → address with YES.

Expected:
  "Open question 'Should we support multiple languages?' expires in 2 days — still relevant? YES / DISCARD"
  On YES (first extension): appends "extended: 2026-05-19 (x1)". TTL resets from today.
  On YES again next time: "extended: [date] (x2)".
  On YES a third time: "Extended twice — escalate to PM or DISCARD. No further extension available."

Pass if: Extension field appended in-place. TTL resets. Max 2 enforced.
```

---

**TC-075** | Session-start step 4f: Section 4 day 5+ (no action) → RESUBMIT/DISCARD
```
Setup:
  Add to Section 4 (5 days old, no action set):
    → 2026-05-14 | Something broken in checkout | session: qa-session-2026-05-14 | raised by: Shubham | priority: MED
  Start session.

Expected:
  "Vague finding 'Something broken in checkout' expires in 2 days — RESUBMIT or DISCARD?"

Pass if: RESUBMIT and DISCARD offered. On RESUBMIT: S4 item marked ⤴️ PROMOTED,
         new S1 entry appended using raw finding text as title.
```

---

**TC-076** | Session-start step 4g: Section 4 RESUBMIT-tagged day 7+ → not silently deleted
```
Setup:
  Add to Section 4 (7 days old, action: RESUBMIT):
    → 2026-05-12 | Cart total wrong | session: qa-session-2026-05-12 | raised by: Shubham | priority: MED | action: RESUBMIT
  Start session.

Expected:
  Item NOT silently deleted (RESUBMIT-tagged items exempt).
  "Vague finding 'Cart total wrong' tagged RESUBMIT is expiring — RESUBMIT now or DISCARD?"

Pass if: Item surfaces for decision. Not silently gone.
```

---

**TC-077** | Session-start step 5: max-items warning when section has ≥10 items
```
Setup:
  Add 10+ unresolved items to Section 1.
  Start session.

Expected:
  "WARNING: Section 1 has [X] items — process may be stalled. Review recommended."

Pass if: Warning shown when count ≥ 10.
```

---

**TC-078** | Session-start step 6: unresolved items summary → YES shows compact table
```
Setup:
  Add 2 unresolved items (no resolution state) across different sections.
  Start session → clear any step 4 warnings → reach step 6.

Expected:
  "PARKING_LOT has [N] unresolved items — review first? YES / SKIP"
  On YES: shows compact table grouped by section:
    S1 Retry Queue: [N] | S2 Deferred: [N] | S3 Questions: [N] | S4 Vague: [N]
    Then lists each item: [section] [date] [title] [priority]
  Display only — no action taken.

Pass if: Table shown. No ticket creation or deletion triggered.
         On SKIP: session proceeds. Not surfaced again until next session.
```

---

**TC-079** | Writing rule: S1 dedup — same title increments retries, no new append
```
Setup:
  Section 1 already contains:
    → 2026-05-17 | Login button broken iOS | reason: MCP error | retries: 1 | raised by: Shubham | priority: MED | action: RETRY

Then trigger a new ticket creation failure for the same title.

Expected:
  No new S1 entry appended.
  Existing entry retries field updated: retries: 2 in-place.
  Inline: "Added to PARKING_LOT S1: Login button broken iOS" (or equivalent update notice).

Pass if: Only one S1 entry for this title. retries field incremented.
```

---

**TC-080** | Writing rule: S1 retry threshold ≥3 → DISCARD recommended, not auto-discarded
```
Setup: after TC-079, trigger another failure → retries reaches 3.

Expected:
  "Login button broken iOS has failed 3 times — DISCARD recommended."
  Claude waits for human confirmation.
  Does NOT auto-discard.

Pass if: Recommendation surfaced. Item remains until human confirms DISCARD.
```

---

**TC-081** | Writing rule: S2 new item → ESCALATE-BY AUTO-COMPUTE applied
```
Trigger: Type 2 question → park to PARKING_LOT S2 (e.g. "Should we use Firebase or AWS SNS?" → B).

Expected:
  New S2 entry appended with escalate by: auto-computed:
    MED priority → today (2026-05-19) + 14 days = 2026-06-02
    HIGH priority → today + 7 days = 2026-05-26
    LOW priority → today + 25 days = 2026-06-13

Pass if: escalate by: field present and matches correct offset for the priority.
```

---

**TC-082** | Writing rule: cross-promotion uses today as item date, not original
```
Setup: manually promote a Section 3 item to Section 2.

Expected:
  New S2 entry:
    - item date = today (promotion date), not original S3 date
    - "promoted from: S3 on 2026-05-19" appended
    - escalate by: auto-computed from today
  Original S3 item marked ⤴️ PROMOTED.

Pass if: New S2 entry has today's date. Original marked ⤴️ PROMOTED.
         escalate by: based on today, not original item date.
```

---

**TC-083** | Writing rule: RESUBMIT from S4 → S1 dedup check first
```
Setup:
  Section 1 has: → 2026-05-17 | Cart total wrong | retries: 0 | ...
  Section 4 has: → 2026-05-14 | Cart total wrong | action: RESUBMIT | ...
  Trigger RESUBMIT on S4 item.

Expected:
  S1 dedup finds matching title.
  retries incremented on existing S1 entry (not new append).
  S4 item marked ⤴️ PROMOTED.
  If retries >= 3 after increment: DISCARD recommendation surfaced.

Pass if: No new S1 entry. retries updated. S4 marked promoted.
```

---

**TC-084** | SKIP NON-REQUIRED blocks session when expired S2 items exist
```
Setup:
  Add expired S2 item (31+ days old, x0 escalations) — this requires a decision.
  Add expiring S1 item (day 5) — this can be skipped.
  Start session → batch shows 2 items.

Expected:
  "Note: 1 expired item(s) require a decision and cannot be skipped."
  SKIP ALL replaced with SKIP NON-REQUIRED.
  On SKIP NON-REQUIRED: S1 item skipped, S2 expired item addressed immediately.

Pass if: SKIP ALL not offered. S2 expired item cannot be bypassed.
```

---

## Test Run Tracker

| TC | Description | Result | Notes |
|---|---|---|---|
| TC-001 | Fast path, no questions | | |
| TC-002 | Vague → one question | | |
| TC-003 | High complexity → two questions | | |
| TC-004 | Empty input → rejection | | |
| TC-005 | Single word → rejection | | |
| TC-006 | "QA batch" → QA_INTAKE | | |
| TC-007 | Numbered list → QA_INTAKE | | |
| TC-008 | TC-NNN format → QA_INTAKE, PASS skipped | | |
| TC-009 | Reporter auto-extracted | | |
| TC-010 | Environment auto-extracted | | |
| TC-011 | Production → urgent priority | | |
| TC-012 | Frontend bug → frontend+bug tags | | |
| TC-013 | Backend bug → backend+bug tags | | |
| TC-014 | Unknown domain → bug tag only | | |
| TC-015 | Assumption surfaced | | |
| TC-016 | DETAIL [n] | | |
| TC-017 | EDIT title | | |
| TC-018 | EDIT tags → resolved to IDs | | |
| TC-019 | EDIT invalid field → error | | |
| TC-020 | DROP [n] | | |
| TC-021 | RESTORE [n] | | |
| TC-022 | RESUME DRAFTS | | |
| TC-023 | CONFIRM ALL → creates tickets | | |
| TC-024 | CONFIRM ALL all flagged → one warning | | |
| TC-025 | Unrecognised command → help | | |
| TC-026 | Type 1 question → answered from context | | |
| TC-027 | Type 2 → spike/park/TBD | | |
| TC-028 | Type 2 → TBD → capped score | | |
| TC-029 | Type 3 → implied requirement | | |
| TC-030 | Type 3 yes, ≥70% → direct to S4 | | |
| TC-031 | Type 3 yes, <70% → one question | | |
| TC-032 | Type 3 no → "what should it be?" | | |
| TC-033 | Type 4 → park/discuss only | | |
| TC-034 | Two requirements → one batch table | | |
| TC-035 | Large input → token estimate block | | |
| TC-036 | Duplicate bug → ⚠️ flag | | |
| TC-037 | CONFIRM ALL with duplicate → A/B/C | | |
| TC-038 | Option C → no ticket, link shown | | |
| TC-039 | New unique bug → clean flag | | |
| TC-040 | Created ticket has empty assignee | | |
| TC-041 | Created ticket stage = Backlog | | |
| TC-042 | Description has 6 HTML h3 sections | | |
| TC-043 | Staging bug → normal priority | | |
| TC-044 | Production bug → urgent priority | | |
| TC-045 | No ticket before CONFIRM ALL | | |
| TC-046 | Question ≤ 3 lines | | |
| TC-047 | No narration in responses | | |
| TC-048 | One clarifying question max | | |
| TC-049 | No repeat questions in session | | |
| TC-050 | End-of-session TEAM_CONTEXT proposals | | |
| TC-051 | PASS tests silently skipped in QA | | |
| TC-052 | QA fields auto-set | | |
| TC-053 | TC-NNN in prd: field | | |
| TC-054 | Vague finding → ADD/DISCARD/PARK | | |
| TC-055 | ADD [n] → one question → drafted | | |
| TC-056 | PARK [n] → output shown | | |
| TC-057 | Parallel duplicate check ≤5 findings | | |
| TC-058 | QA SESSION COMPLETE format | | |
| TC-059 | Meeting notes → out of scope | | |
| TC-060 | Vendor task → out of scope | | |
| TC-061 | URL-only → rejection | | |
| TC-062 | Creation failure → surfaced, not hidden | | |
| TC-063 | Conflicting signals → explicit wins | | |
| TC-064 | Expired S1/S3/S4 items silently cleaned | | |
| TC-065 | Resolved items removed at session start | | |
| TC-066 | Last Reviewed unset → review reminder | | |
| TC-067 | Warnings batched and sorted before display | | |
| TC-068 | Step 4a: escalate-by passed → notification, auto-advance | | |
| TC-069 | Step 4c x0: ESCALATE/DISCARD only (no EXTEND) | | |
| TC-070 | Step 4c x1: ESCALATE AGAIN/DISCARD/EXTEND offered | | |
| TC-071 | Step 4c x3: FINAL NOTICE, no ESCALATE AGAIN | | |
| TC-072 | Step 4d day 5+: retry threshold prepended | | |
| TC-073 | Step 4d RETRY queued, runs after checklist | | |
| TC-074 | Step 4e day 12+: YES extends TTL, max 2 | | |
| TC-075 | Step 4f day 5+: RESUBMIT/DISCARD for vague | | |
| TC-076 | Step 4g: RESUBMIT-tagged not silently deleted | | |
| TC-077 | Step 5: max-items warning ≥10 | | |
| TC-078 | Step 6: unresolved summary, YES shows table | | |
| TC-079 | Writing: S1 dedup → increment retries, no new append | | |
| TC-080 | Writing: S1 retry ≥3 → DISCARD recommended, not auto | | |
| TC-081 | Writing: S2 new item → escalate-by auto-computed | | |
| TC-082 | Writing: cross-promotion uses today as item date | | |
| TC-083 | Writing: RESUBMIT from S4 → S1 dedup first | | |
| TC-084 | SKIP NON-REQUIRED blocks when expired S2 present | | |
