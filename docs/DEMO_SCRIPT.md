# TiffinConnect AI Ticket Pipeline — Demo Script
# VERSION: 1.0 | Created: 2026-05-21 | Pre-flight: PASSED (all stray tickets cleared)

---

## Run of Show

```
Pre-demo setup     5 min  — browser tabs, Claude Enterprise, Odoo board
Opening script     2 min  — context-setting before first keystroke
Scenario 1         4 min  — Developer reports a bug verbally
Scenario 2         3 min  — Vague requirement from a meeting
Scenario 3         2 min  — "Should we add this?" — implied requirement
Scenario 4         3 min  — QA batch with a vague finding
Scenario 5         2 min  — Open question → PARKING_LOT
Wrap-up            2 min  — Live Odoo board + session learnings
Buffer             4 min  — Questions / recovery
─────────────────────────
Total             ~22 min
```

---

## Pre-Demo Setup (do this before the room fills)

**Browser tabs open in this order:**
1. Claude Enterprise — Ticket Intake — TiffinConnect project
2. Odoo — TiffinConnect board (Kanban view, Backlog column visible)
3. This script (separate device or printed)

**Verify in Claude Enterprise:**
- Hammer icon (🔨) in the toolbar — MCP connected. If missing: Project Settings → MCP → reconnect.
- Knowledge files visible: LAYER_0_GLOBAL, LAYER_2_FASTPATH, QA_INTAKE, BUG_REPORT_TEMPLATE, TEAM_CONTEXT
- PARKING_LOT.md loaded (check Knowledge list — required for fallback paths to work)

**Verify in Odoo:**
- TiffinConnect board is empty of test tickets (pre-flight cleared #2552, #2551, #2624–2629)
- Backlog column visible and clean

**Font size:** Increase Claude chat font to 140% so audience can read it.

---

## Opening Script (say this verbatim)

> "Before this system, creating a ticket meant opening Odoo, filling in every field manually,
> deciding the priority, picking the project, tagging it, and hoping you remembered all the
> context from the conversation. If it was a QA session with five findings, you were doing
> that five times.
>
> Now you open Claude, describe what you found, and it handles everything — including
> checking whether the ticket already exists, inferring priority and tags, and writing a
> structured description that any developer can pick up without asking you a single question.
>
> I am going to show you five situations your team deals with every day."

---

## SCENARIO 1 — Developer reports a bug verbally
**Time: 4 minutes**

### What this shows
| Capability | Skill rule |
|---|---|
| Token estimate + permission gate | LAYER_0 Rule 2 |
| YES / NO / BREAKDOWN options | LAYER_0 Rule 3 |
| Silent fast-path classification | FASTPATH S1 |
| Extraction from plain language | FASTPATH S2 |
| Assumption surfacing + ⚠️ flags | FASTPATH S3 |
| Duplicate check via cached list_tickets | FASTPATH S8 |
| Quality stars (⭐) | FASTPATH S5 |
| DETAIL command — full ticket preview | FASTPATH S5 |
| CONFIRM ALL — batch creation | LAYER_0 Rule 1b |
| Field mapping (project, stage, priority, tags, no assignee) | FASTPATH S9 |
| Structured HTML description in Odoo | FASTPATH S9 |
| Final summary (N created / N skipped / N failed) | FASTPATH S9 |

### Say this

**Case 1**
> "Kunal found a bug while working on the API. He describes it the way he normally would —
> informally, in plain language. Watch what happens."

**Case 2**
> "Tanu spent the afternoon debugging a login issue and pinged the channel. No ticket yet —
> just a Slack message. Let's create it the way she sent it."

**Case 3**
> "Kunal just DM'd you. Production issue, happened an hour ago. Watch how the system
> handles this differently from a staging bug — specifically the priority."

### Type exactly

**Case 1**
```
The order API is returning a 500 error when the cart is empty.
Happens every time on staging. Started after yesterday's deployment.
Reported by Kunal Sharma.
```

**Case 2**
```
OTP verification is failing for phone numbers with +91 country code prefix.
The API returns a 400 error and login is completely blocked for those users.
Happening on staging. Reported by Tanu Lamba.
```

**Case 3**
```
Subscription renewal is broken on production. The next_billing_date field
returns null after the first payment goes through. Users are not being charged
for renewals. Started this morning. Reported by Kunal Sharma.
```

> **Presenter note for Case 3:** Input says production + billing impact — Claude will
> set priority=1 (Urgent) per TEAM_CONTEXT Section 4 priority rules. At the Odoo step
> point out "Priority: Urgent — production bug overrides the type-based default."

### Walkthrough — step by step

**1. Token estimate block appears first**

> "Before Claude does anything, it tells you exactly what it is about to do, how much it will
> cost, and what will change in Odoo. Nothing happens until you say YES."

Point out the three options:
- `YES` — proceed
- `NO` — cancel
- `BREAKDOWN` — if the budget is tight, split into sessions

> "BREAKDOWN is how you handle large PRD imports or batches that exceed the session budget.
> You won't need it today."

Type: `YES`

**2. Classification happens silently**

> "No routing menu, no 'which skill should I use?' Claude classified this as a single bug
> report, confirmed completeness is high enough for fast path, and got to work. You never
> see that decision."

**3. Duplicate check runs**

> "Before drafting anything, Claude fetched all existing bug tickets from the project and
> checked this title against them in memory. One call to Odoo — results cached for the
> rest of the session so it won't repeat that call."

**4. Batch table appears — pause here**

Point out the quality score:
> "Three stars means: specific title, has steps implied, the acceptance criteria is clear.
> It lost two stars because there is no PRD reference and no impact statement naming a team."

Point out the ⚠️ Assumed flag:
> "Claude flagged what it assumed — that 'empty cart' means zero items, not a deleted cart.
> It did not ask. Completeness was high enough to proceed. The flag is there so the developer
> can verify."

**5. Type DETAIL 1**

> "DETAIL shows every field before anything is created. This is the full structured ticket —
> What is happening, Expected behaviour, Steps to reproduce, Acceptance criteria, Impact,
> Context. A developer can pick this up without asking a single question."

Read out the acceptance criteria line to the audience.

**6. Type CONFIRM ALL**

> "One confirm. That is it."

**7. Switch to Odoo tab immediately**

Point out the ticket appearing:
> "Stage: Backlog. Priority: Normal — staging bug, not production. Tags: Backend and Bug —
> inferred from the API description. Assignee: empty — assignment happens at triage, not
> at creation. Description: fully structured HTML."

Click into the ticket and show the description sections.

---

## SCENARIO 2 — Vague requirement from a meeting
**Time: 3 minutes**

### What this shows
| Capability | Skill rule |
|---|---|
| Completeness scoring (<60% triggers question) | FASTPATH S1 |
| One clarifying question maximum | LAYER_0 Rule 6 |
| Assumption flags after answer | FASTPATH S3 |
| Quality scoring reflects missing fields | FASTPATH S5 |
| FIX command — resolve a flag inline | FASTPATH S3 |
| EDIT command — change a specific field | FASTPATH S5 |
| DROP command — remove from batch | FASTPATH S5 |

### Say this

**Case 1**
> "Vijay came out of a product meeting with this note. This is exactly what lands in Slack."

**Case 2**
> "Sahil mentions during standup that he's been getting complaints from a few users.
> He knows what the problem area is but hasn't had time to dig in yet."

**Case 3**
> "Tanu flags a recurring support request in the team group chat — same issue coming up
> multiple times, but nobody has raised a ticket for it."

### Type exactly

**Case 1**
```
We need to improve the checkout experience.
Users are dropping off before completing payment.
```

**Case 2**
```
The subscription flow is confusing. Users don't seem to understand
what they are signing up for before they pay.
```

**Case 3**
```
Vendors are saying the dashboard is hard to use.
They keep asking us how to find their pending orders.
```

**Answer to clarifying question — Case 1:**
```
The confirm button is hard to find on mobile. Users on iOS especially are missing it.
```

**Answer to clarifying question — Case 2:**
```
The plan details are not shown before payment — users don't see which meals
they are getting or the delivery schedule before they confirm.
```

**Answer to clarifying question — Case 3:**
```
Pending and approved orders look identical on the vendor dashboard.
There is no visual distinction between the two states.
```

### Walkthrough — step by step

**1. Claude scores completeness and asks one question**

> "Claude scored this below 60% complete — not enough to draft a ticket. It asked exactly
> one question. The single most important missing piece. Not three questions. One."

> "This is Rule 6 — one clarifying question maximum. If Claude ever asks you two questions
> in the same message, that is a bug. Screenshot it and send it to me."

**2. Answer the question**

Type:
```
The confirm button is hard to find on mobile. Users on iOS especially are missing it.
```

**3. Batch table appears — pause here**

Point out the ⚠️ Assumed flags:
> "Claude made three assumptions — the reporter, the platform affected, the priority. All
> flagged. You can leave them as-is or fix them now."

Point out the quality score:
> "Three stars. Lost two because there is no PRD reference and no reporter was named in
> the original input."

**4. Show FIX command**

Type: `FIX 1`

> "FIX resolves a specific flag. Claude asks about the most critical assumption only.
> You answer in one word — yes, no, or the correct value. The ticket re-scores.
> You never restart the whole flow."

Answer the FIX question (confirm or correct the assumption).

**5. Show EDIT command**

Type: `EDIT 1 tags=frontend,bug`

> "EDIT changes a specific field without touching anything else. Valid fields: title,
> priority, env, tags, deadline, ac, description. Tag names — not IDs. Claude resolves
> the IDs from TEAM_CONTEXT."

**6. Type DROP 1**

> "Dropping this one — we don't want a demo ticket in Odoo. In real use you would
> CONFIRM ALL. Note: DROP saves the draft in memory this session. You can recover it
> with RESTORE 1. Once the session ends it is gone."

---

## SCENARIO 3 — "Should we add this?" — implied requirement
**Time: 2 minutes**

### What this shows
| Capability | Skill rule |
|---|---|
| Type 3 detection — question naming one feature | FASTPATH S6 |
| Implied requirement extraction | FASTPATH S6 |
| Confirmation before proceeding | FASTPATH S6 |
| Single follow-up question if completeness <70% | FASTPATH S6 |
| RESTORE command mention | FASTPATH S5 |

### Say this

**Case 1**
> "This one is subtle but it matters. Sahil asks a question during standup. Watch what
> Claude does — and what it deliberately does not do."

**Case 2**
> "Vijay mentions something in the Slack channel after testing the app.
> Sounds like a question, but watch what Claude pulls out of it."

**Case 3**
> "Kunal raises this after a call with the PM. He is not sure if it's in scope —
> so he phrases it as a question. Watch."

### Type exactly

**Case 1**
```
Should we add a retry mechanism for failed payment transactions?
```

**Case 2**
```
Should we show the estimated delivery time on the order tracking screen?
```

**Case 3**
```
Can we add a way for vendors to set their daily order capacity limit?
```

### Walkthrough — step by step

**1. Claude extracts the implied requirement**

> "Claude did not ask for more information. It did not create a ticket. It extracted the
> requirement implied by the question and asked you to confirm."

> "This is Type 3 detection. A question that names exactly one thing to build. The team
> does this constantly in standups, Slack, meetings. Without this, those thoughts either
> get lost or turn into noise tickets."

**2. Type: yes**

> "Claude confirms and now checks completeness of the confirmed requirement. If it is
> below 70% complete, it asks one follow-up question — then proceeds directly to the
> review table regardless."

**3. Answer the follow-up question briefly, then review table appears**

> "Ticket ready. Notice no reporter was stated, no environment, no PRD — three stars
> because the requirement itself is specific and testable."

**4. Type DROP 1**

> "Dropping for the demo. Recoverable this session with RESTORE 1 — that brings it
> back to the batch table, re-runs the duplicate check, and picks up from where you were."

---

## SCENARIO 4 — QA batch with a vague finding
**Time: 3 minutes**

### What this shows
| Capability | Skill rule |
|---|---|
| QA_INTAKE routing detection | LAYER_0 Routing Rule |
| Input parser — any format accepted | QA_INTAKE S1 |
| Token estimate for batch | QA_INTAKE S1 |
| Batch draft — all findings simultaneously | QA_INTAKE S3 |
| Vague finding detection | QA_INTAKE S3 |
| ADD / DISCARD / PARK commands for vague findings | QA_INTAKE S3 |
| PARK to PARKING_LOT Section 4 (Vague Findings, 7d TTL) | QA_INTAKE S3 |
| ticket_cache["bug"] shared across all findings (1 MCP call) | QA_INTAKE S2 |
| QA review table (N findings / N clean / N flagged) | QA_INTAKE S4 |
| QA-specific field rules (src=qa-testing, env=staging default) | QA_INTAKE S3 |
| QA SESSION COMPLETE summary block | QA_INTAKE S6 |

### Say this

**Case 1**
> "Shubham just finished a test session with three findings. One of them is not specific
> enough to ticket yet. Normally this is 30 minutes of Odoo work. Watch."

**Case 2**
> "Shubham spent the afternoon testing the vendor app. Same situation — three findings,
> one of them too vague to create a ticket from. Watch how the system handles that one."

**Case 3**
> "Shubham ran through the subscription flow end to end. Three findings from the session.
> One came from a note he jotted down mid-test — not detailed enough yet."

### Type exactly

**Case 1**
```
QA batch from today's test session:
1. Login with Google fails on Android — redirects to blank screen after OAuth2.
2. Something wrong with the promo codes.
3. Push notification not received when order status changes to Out for Delivery.
```

**Case 2**
```
Vendor app findings from today:
1. Vendor dashboard shows incorrect pending order count — shows 5 but only 3 are visible in the list.
2. The skip meal feature doesn't seem to work properly.
3. Subscription renewal date shows current month instead of next month on the diner profile screen.
```

**Case 3**
```
Subscription flow test session findings:
1. Address auto-fill broken on iOS — nothing happens when user selects a suggestion from the dropdown.
2. There's something off with the payment screen during checkout.
3. Order confirmation email not received after successful payment — tested 3 times on staging.
```

> **Presenter note:** In all 3 cases, the vague finding is the middle one (finding 2).
> Type `PARK 2` in each. The other two findings proceed to the review table cleanly.

### Walkthrough — step by step

**1. QA_INTAKE routing fires**

> "Claude detected a numbered list of findings and routed to QA_INTAKE automatically.
> No manual skill selection. The team never chooses which skill to use."

**2. Token estimate for batch**

> "Batch token estimate — three tickets at ~750 tokens each. Budget fits.
> Type YES."

Type: `YES`

**3. Vague finding flagged before the review table**

> "Finding 2 is flagged before anything else. Claude could not form a title or observable
> detail from 'something wrong with the promo codes' — circular description, nothing
> actionable. Three options shown."

Point out: `ADD 2` / `DISCARD 2` / `PARK 2`

Type: `PARK 2`

> "PARK saves it to PARKING_LOT Section 4 — Vague Findings — with a 7-day TTL.
> Shubham comes back next session, adds the actual steps and error, resubmits.
> Nothing is lost. Nothing creates noise in Odoo."

**4. QA review table appears with 2 tickets**

> "Two tickets drafted, duplicate-checked — one list_tickets call for the 'bug' tag,
> cached across both findings simultaneously. Quality scored. Both clean."

Point out the QA-specific fields: `src: qa-testing`, `env: staging`

> "Claude defaulted environment to staging — correct for QA findings. If a finding
> was on production, Shubham would type EDIT 1 env=prod."

**5. Type CONFIRM ALL**

**6. QA SESSION COMPLETE block appears**

> "Post-session summary. N created, N skipped, N failed. If anything failed,
> it saves to PARKING_LOT Section 1 Retry Queue automatically."

**7. Switch to Odoo**

> "Two tickets. One confirm. The vague finding is parked — not discarded, not
> a noise ticket, not forgotten."

---

## SCENARIO 5 — Open question that should not become a ticket
**Time: 2 minutes**

### What this shows
| Capability | Skill rule |
|---|---|
| Type 4 detection — exploratory thinking, no feature named | FASTPATH S6 |
| No ticket creation — not offered, not implied | FASTPATH S6 |
| PARK to PARKING_LOT Section 3 (Open Questions, 14d TTL) | FASTPATH S6 |
| TTL + warn-at-day-12 behaviour | PARKING_LOT S3 |
| Discuss option | FASTPATH S6 |

### Say this

**Case 1**
> "This is one of the most important things the system gets right. Watch what happens
> when someone types a thought instead of a requirement."

**Case 2**
> "Vijay drops this in the team chat after reading a competitor's announcement.
> It's a real thought worth keeping. Watch what the system does with it."

**Case 3**
> "Kunal raises this during a planning session. Everyone nods. Nobody writes a ticket.
> Normally this thought disappears. Watch what happens instead."

### Type exactly

**Case 1**
```
I wonder if we should migrate to a different payment gateway.
```

**Case 2**
```
Maybe we should think about adding a loyalty points system at some point.
```

**Case 3**
```
Has anyone considered building real-time delivery tracking
instead of relying on static order status updates?
```

### Walkthrough — step by step

**1. No ticket, no offer to create one**

> "Claude did not create a ticket. It did not ask clarifying questions. It did not
> offer to create a ticket. It correctly identified this as exploratory thinking —
> Type 4. Two options: park it or discuss it."

**2. Point out what was NOT said**

> "It did not say 'This is out of scope.' It did not reject the input. It recognised
> this as a real thought worth preserving and gave you two options."

**3. Type: A**

> "Parked to PARKING_LOT Section 3 — Open Questions. 14-day TTL. Claude will warn
> at day 12 if nothing has happened. After 14 days it auto-deletes — not silently,
> with an escalation. Max two extensions if the question is still live."

> "When the team is ready to act on this, they come back and describe the actual
> requirement: which gateway, why, what the integration looks like. That becomes
> a ticket. This thought becomes a ticket only when it is ready."

---

## WRAP-UP — Live Odoo + Session Learnings
**Time: 2 minutes**

### Switch to Odoo — show what was created

Point to each ticket and read the title:
- Scenario 1 ticket — "Order API returns 500 error when cart is empty"
- Scenario 4 tickets — the two QA findings

For Scenario 1 ticket:
> "Click into this one. Look at the description. What is happening. Expected behaviour.
> Steps to reproduce. Acceptance criteria. Impact. Context — who reported it, what source,
> what environment. Any developer on the team can pick this up right now without asking
> a single question."

For the QA tickets:
> "Both in Backlog. Both unassigned. Triage owner assigns in the morning — that is
> Phase 2. Until then they sit here, structured, ready."

### Show session learnings (Rule 8)

> "At the end of every session where Claude made routing or mapping decisions,
> it proposes additions to TEAM_CONTEXT — the knowledge file it reads when deciding
> tags, priority, project. You review each proposal. Approve with Y, reject with N.
> Approved entries mean Claude never asks the same routing question again in any
> future session. This is how the system gets smarter over time without anyone
> configuring anything."

### Close with three sentences

> "One: you never need to open Odoo to create a ticket again.
> Two: if Claude asks you more than one question at once, screenshot it and send it to me — that is a bug.
> Three: use this from tomorrow. Anything unexpected, send it to me directly."

---

## Contingency Guide

### MCP not connected — hammer icon missing
Close and reopen Claude Enterprise. Project Settings → MCP → reconnect. Re-run the scenario from the start — the connection is the only thing that failed.

### Duplicate detected at CONFIRM ALL — unexpected pause
A leftover test ticket exists in Odoo. Type `B` (skip) and say: "Duplicate detection is working correctly — it found a matching ticket. In real use you would choose A to create anyway, B to skip, or C to link to the existing one." Move on.

### Token estimate shows "exceeds" instead of "fits"
Type `BREAKDOWN`. Point out the session split proposal — "Session A now: Requirements 1–N. Session B new session: Requirements N+1–end." Say: "This is how the system handles large PRD imports without running out of budget." Type `NO` and reduce the batch to continue.

### Claude asks two questions in one message
Answer the first one. Type `CONFIRM ALL` to continue. Explicitly note to the audience: "That was a bug — it should only ask one question. I will fix that after the demo." This demonstrates the feedback loop.

### Scenario 4 — vague finding PARK fails
PARKING_LOT.md is not loaded in the session. Claude will output the PARK entry as text for manual saving. Continue the demo: "Claude shows you what to save when the file isn't available — you copy it manually." Note the missing file as an action item.

### Ticket creates in wrong stage
Show it in Odoo anyway. "Triage moves it to the right stage. The creation itself worked — the routing rule will be tightened." Move on.

### Anything else breaks
Type `DROP` on whatever is in the table. Start the next scenario. Keep moving — the demo recovers faster by continuing than by fixing live.

---

## Capability Coverage Map (presenter reference)

| Capability | Scenario | Command / moment |
|---|---|---|
| Token estimate + permission gate (YES/NO/BREAKDOWN) | S1 | Before batch table |
| BREAKDOWN session split | Contingency | "Token estimate exceeds" |
| Silent fast-path classification | S1 | Walkthrough point 2 |
| Extraction from plain language | S1 | Walkthrough point 2 |
| Auto tag detection (backend/frontend/bug) | S1 | Odoo ticket view |
| Priority routing (bug+staging→normal) | S1 | Odoo ticket view |
| No assignee at creation | S1 | Odoo ticket view |
| Duplicate check (list_tickets cached, not 3×search) | S1 | Walkthrough point 3 |
| ≥80% duplicate → pause at CONFIRM ALL | Contingency | Unexpected duplicate |
| Quality stars ⭐ | S1, S2 | Batch table |
| ⚠️ Assumed flags | S1, S2 | Batch table |
| DETAIL command | S1 | Type DETAIL 1 |
| CONFIRM ALL (batch, not per-ticket) | S1, S4 | Confirm step |
| Structured HTML description (6 sections) | S1 | Odoo ticket view |
| Final summary (N created / N skipped / N failed) | S1 | After CONFIRM ALL |
| Completeness scoring (<60% triggers question) | S2 | Walkthrough point 1 |
| One clarifying question maximum (Rule 6) | S2 | Walkthrough point 1 |
| FIX command (resolve flag inline) | S2 | Type FIX 1 |
| EDIT command (change specific field) | S2 | Type EDIT 1 tags=... |
| DROP command (remove from batch) | S2, S3 | Type DROP 1 |
| RESTORE command (recover dropped ticket) | S3 | Mentioned after DROP |
| Type 3 detection — implied requirement | S3 | Full scenario |
| Type 4 detection — open question, no ticket | S5 | Full scenario |
| QA_INTAKE routing (auto-detected) | S4 | Walkthrough point 1 |
| Batch draft — all findings simultaneously | S4 | Walkthrough point 3 |
| Vague finding detection + ADD/DISCARD/PARK | S4 | Type PARK 2 |
| PARKING_LOT Section 4 (Vague Findings, 7d TTL) | S4 | After PARK 2 |
| ticket_cache shared across batch (1 MCP call) | S4 | Walkthrough point 4 |
| QA review table (N findings / N clean / N flagged) | S4 | Batch table |
| QA SESSION COMPLETE summary block | S4 | After CONFIRM ALL |
| PARKING_LOT Section 3 (Open Questions, 14d TTL) | S5 | After type A |
| TTL + warn-at-day-12 | S5 | Walkthrough point 3 |
| TEAM_CONTEXT self-update (Rule 8) | Wrap-up | End of session |

---

## Post-Demo Actions

**Tell the team before they leave:**
1. The Project is live — use it from tomorrow for all bugs and requirements
2. `BUG_REPORT_TEMPLATE.md` is pinned in `#bugs` — copy it for structured reports
3. For QA sessions: paste all findings at once, not one at a time
4. One question per message from Claude — more than that is a bug, send a screenshot

**After the demo — clean up Odoo:**
- Drop the two QA batch tickets created in Scenario 4 if you don't want them in Backlog
- The Scenario 1 ticket is a real-looking bug — decide whether to keep or delete

**Check PARKING_LOT.md was updated:**
- Section 3 should have the payment gateway open question from Scenario 5
- Section 4 should have "something wrong with the promo codes" from Scenario 4
