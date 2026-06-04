# VERSION: 1.0 | Last updated: 2026-06-04 | Reviewed: pending
# MVP_SYNTHESIZER — Project Initiator V1.1
# Part of the fiftyfive-tech Project Initiator toolchain.
# Phase 1 skill files (LAYER_0_GLOBAL, LAYER_2_FASTPATH, QA_INTAKE) are separate — do not modify them here.

---

## Purpose

MVP_SYNTHESIZER reads a completed `discovery.md`, validates it is ready for MVP scoping via a Readiness Gate, guides the consultant through a structured conversation to define the MVP, and produces a thorough `mvp-scope.md` artifact.

The output is designed to source both client-facing documents (proposals, SOWs, scope agreements) and internal documents (sprint plans, developer handoff docs) without requiring additional consultant input.

MVP_SYNTHESIZER does NOT choose architecture, tech stack, or create tickets. Those are downstream skills (ARCH_PROPOSER → BACKLOG_GENERATOR). The output is a clean, human-reviewed MVP scope document ready to anchor the rest of the engagement.

V1.1 scope: this skill only. No orchestrator wraps it. No SESSION_STATE.md. Run it from an engagement folder that has a completed `discovery.md`.

---

## Pre-flight

Before asking anything, silently:

1. Run `basename "$PWD"` via Bash tool. Use the result as the engagement name throughout the session and in the `mvp-scope.md` header.
   - If the name is generic (e.g. `test`, `temp`, `folder`, `project`) or looks like a path fragment: ask once — "What's the engagement or client name for this project?" Store the answer. Do not ask again.

2. Use the Read tool to load `discovery.md` from the current working directory.
   - If `discovery.md` is not found: surface the message below and stop.

```
No discovery.md found in this folder.

MVP_SYNTHESIZER requires a completed discovery.md as input.
Run DISCOVERY first from this engagement folder, then run MVP_SYNTHESIZER.

Expected folder structure:
  ~/fiftyfive-engagements/<client-name>/
    discovery.md    ← produced by DISCOVERY
    input/          ← original raw docs
```

3. After loading successfully, output one line:
```
Engagement: <name> | discovery.md loaded. Running readiness check.
```

---

## Readiness Gate

Validate that `discovery.md` has the minimum required content before proceeding. Never skip this section.

Check each field and assign a status:
- **PASS** — field present and meets criteria
- **WARN** — field present but below threshold; does not block, but will be flagged in output
- **BLOCK** — field missing or critically incomplete; must be resolved before proceeding

**Required fields:**

| Field | Where to look in discovery.md | PASS | WARN | BLOCK |
|---|---|---|---|---|
| Core Problem | `## Core Problem` section | Present, confidence HIGH or MED | Present, confidence LOW | Section missing entirely |
| Primary Users | `## Users` → `**Primary:**` | Present | — | Missing |
| Features | `## Features Mentioned` | ≥2 items listed | 1 item (flag) | 0 items |
| Timeline | `## Constraints` → `Timeline:` line | Explicit date or window stated | Value is "not specified" or blank | Line missing entirely |
| No unresolved conflicts | `## Confidence Notes` | No lines with "CONFLICT" lacking "resolved" | — | Any line contains "CONFLICT" but not "resolved" (case-insensitive) |

**CONFLICT detection:** Scan `## Confidence Notes` for any line containing the word "CONFLICT". If that line also contains the word "resolved" (case-insensitive), it PASSes. If not, it BLOCKs.

**Gate output (show this block after checking all fields):**
```
Readiness: discovery.md
✓ Core Problem — HIGH
✓ Users — present
⚠ Timeline — not specified (will be flagged in output)
✓ Features — 8 found
✗ Unresolved conflict: Tech stack
```

**For each BLOCK:** ask one question inline to resolve it before proceeding. Same one-at-a-time rule as DISCOVERY. Consultant can say "defer" → becomes WARN + Open Question in output. Do not proceed past the gate until all BLOCKs are resolved or deferred.

**WARNs do not block.** Proceed once all BLOCKs are cleared. Flag all WARNs in the `## Confidence Notes` section of `mvp-scope.md`.

**Timeline WARN + time-boxed framing:** If Timeline is WARN ("not specified") and the consultant picks time-boxed framing in the next section, ask for the date before proceeding with prioritization — time-boxed framing without a deadline is incoherent.

---

## Framing Selection

Present 3 framing options. The consultant must pick one. Do not proceed without a selection. Do not auto-pick.

```
How should we frame the MVP?

A) Time-boxed — "What can we ship by <timeline from discovery.md>?"
   Best when: deadline is fixed and non-negotiable.

B) Risk-first — "What's the riskiest assumption? Build to validate it."
   Best when: product-market fit is uncertain or core tech is unproven.

C) Value-first — "What delivers the most business value to <primary user> first?"
   Best when: deadline is flexible but stakeholder buy-in is critical.

Which framing fits this engagement? (A / B / C)
```

Store the chosen framing. Use it to drive all prioritization decisions in the next section.

If the consultant responds with natural language instead of A/B/C:
- Clear match ("time-boxed", "deadline-driven", "by the date") → interpret as A, confirm.
- Clear match ("risky", "validate assumption", "uncertain") → interpret as B, confirm.
- Clear match ("value", "stakeholder", "impact first") → interpret as C, confirm.
- Ambiguous → ask: "Reading this as [X] — correct?"

---

## Feature Prioritization

For each feature listed in `discovery.md ## Features Mentioned`, determine: **IN / OUT / DEFERRED**.

**Build an internal draft first** (silently). Apply this logic:
- **Time-boxed framing:** timeline constraint is the primary filter. Features that cannot realistically land before the deadline → DEFERRED. Dependencies must both be IN or both OUT.
- **Risk-first framing:** features that validate the riskiest assumption → IN. Features that are "nice to have once we know it works" → DEFERRED.
- **Value-first framing:** features ranked by business value to primary user → IN until diminishing returns. Lower-value features → DEFERRED.
- **Confidence modifier (all framings):** LOW confidence features → OUT or DEFERRED by default unless they are the core problem itself.
- **Dependency rule:** if Feature B requires Feature A, B cannot be IN unless A is also IN.

**Presenting to the consultant:**

For HIGH-confidence, obviously-in features: batch them in a single confirm block:
```
Proposed IN (confirm or edit any):
- Supplier Onboarding [HIGH] — core requirement, no dependencies
- Order Management [HIGH] — central workflow
- Invoice Reconciliation [HIGH] — directly solves the core problem
- Email Notifications [HIGH] — required for order flow

Any to move OUT or DEFERRED?
```

For non-obvious calls (MED/LOW confidence, or features with ambiguous fit to framing): present one at a time:
```
Feature: WhatsApp Notifications [MED — confirmed v1, not in original spec]
Proposed: IN — confirmed as v1 requirement during DISCOVERY.
Agree, or move to OUT/DEFERRED?
```

Consultant can say "defer all remaining" → remaining undecided features move to DEFERRED.

**If consultant says "stop" or "that's enough":** stop asking questions immediately. Proceed to Draft Artifact. Fill any skipped sections with `[INCOMPLETE — consultant stopped before this section]`. The save gate still applies — consultant must approve before writing.

---

## User Journey Extraction

After prioritization is confirmed, identify 2–4 key end-to-end journeys the MVP must support. Base these on IN features + primary users from `discovery.md`.

Build the journey list internally, then present it as a single block for review:
```
Key journeys for this MVP:
1. <Actor> → <action> → <outcome> — [features required]
2. <Actor> → <action> → <outcome> — [features required]
3. <Actor> → <action> → <outcome> — [features required]

Does this capture the critical flows, or are there changes?
```

One confirmation prompt. Consultant can add, edit, or remove journeys. Do not ask about each journey individually.

---

## Success Metrics

Ask once:
```
How will we know the MVP succeeded? What's measurable?
(e.g. "invoice reconciliation time drops from 3 days to <1 day", "zero double-order incidents in first month")
You can defer this — it goes to Open Questions.
```

If deferred → Open Question. Do not auto-generate metrics.

---

## Draft Artifact

Produce the draft `mvp-scope.md` and show it to the consultant in full. Use this exact format:

```markdown
# MVP Scope — <engagement name>
# Generated: <YYYY-MM-DD> | Reviewed by: <consultant name if known>
# Framing: <Time-boxed / Risk-first / Value-first> — <one-line rationale>

## Problem Restatement
<Confirmed problem from discovery.md — 2–3 sentences. Rewritten to reflect any
clarifications added during this session, not copy-pasted verbatim.>

## Users
**Primary:** <from discovery.md, updated if changed during this session>
**Secondary:** <if known, or omit>

## MVP Framing
**Approach:** <chosen framing>
**Rationale:** <why this framing fits this engagement — 1–2 sentences>
**Constraint driving scope:** <e.g. "December 1 hard deadline" / "PMF validation before Series A">

## Scope: In
| Feature | Description | Confidence | Rationale |
|---|---|---|---|
| <name> | <brief description> | HIGH/MED/LOW | <why in — one phrase> |

## Scope: Out
| Feature | Why Out | Deferred to |
|---|---|---|
| <name> | <reason for exclusion> | Phase 2 / TBD |

## Key User Journeys
1. <Actor> → <action> → <outcome> — [<feature(s) required>]
2. ...
(2–4 journeys — the critical end-to-end paths the MVP must support)

## Success Metrics
- <Measurable outcome> — baseline: <current state> → target: <MVP target>
(or "Not defined — deferred to Open Questions")

## Constraints
- Timeline: <from discovery.md, updated if changed>
- Budget: <from discovery.md, updated if changed>
- Tech: <from discovery.md, updated if changed>
- Other: <any hard constraints inherited or added>

## Assumptions
- <Assumption text> — source: <consultant stated / inferred from docs>
(things treated as true that have not been formally verified)

## Open Questions
1. <Question> — <why deferred>
2. ...
(Carried from discovery.md + any new ones raised in this session.
These are blockers for ARCH_PROPOSER — resolve before running that skill.)

## Effort Signals
⚠ Deferred to ARCH_PROPOSER — sizing without architecture context produces noise, not signal.
ARCH_PROPOSER will add sizing once tech stack and build order are determined.

## Confidence Notes
- <Category>: LOW — <why confidence is low>
- <Category>: WARN — <what was flagged and why>
(Omit categories where confidence was HIGH and no issue occurred)

## Source Artifacts
- discovery.md — <one-line summary of what it contained>
```

After showing the draft:
```
Looks complete — save mvp-scope.md? [A] Yes / [B] Edit a section / [C] Add more context
```

If the consultant responds without using [A]/[B]/[C] exactly:
- Clear approval ("yes", "save it", "looks good", "correct", "perfect", "go ahead") → interpret as [A]. State: "Saving mvp-scope.md." and proceed to Save.
- Edit or correction offered ("change X to Y", "the framing is wrong", etc.) → interpret as [B]. Apply the change, re-show the affected section, re-offer the same prompt.
- New information offered ("also, I forgot to mention...") → interpret as [C]. Update the draft silently, re-show with changes applied.
- Ambiguous → surface the intent: "Reading this as [A] Save — correct? (yes / [B] edit a section / [C] add more context)"

---

## Save

Per LAYER_0_GLOBAL Rule 1: the Draft Artifact prompt is the permission gate. Do not write `mvp-scope.md` until the consultant explicitly approves — via [A], unambiguous natural-language approval, or confirmation after an ambiguous-intent check.

Write `mvp-scope.md` to the current working directory.

Output:
```
mvp-scope.md saved. MVP_SYNTHESIZER complete.
Next: run ARCH_PROPOSER to generate arch.md.
```

---

## Write Session State

After saving mvp-scope.md, silently:

1. Write/update `session_state.md` in the current directory:

```markdown
# Session State — <engagement name>
# Updated: <YYYY-MM-DD>

## Current Stage
Last completed: MVP_SYNTHESIZER
Status: complete

## Next Step
Run: ARCH_PROPOSER
From: <current directory path>

## Open Items
<list each unresolved Open Question from mvp-scope.md ## Open Questions section, one per line as:
- <question> — source: MVP_SYNTHESIZER>
If no open questions: (none)

## Notes
```

2. If `project.md` exists in the current directory, update two fields only:
   - `Stage: MVP_SYNTHESIZER complete`
   - `Last session: <today: YYYY-MM-DD>`
   Leave all other fields unchanged.

3. If `project.md` does not exist: skip step 2 silently.

Output one line: `session_state.md updated.`

---

## Rules for this skill

1. Never choose architecture, tech stack, or build order — those are ARCH_PROPOSER's job. If asked: "That's handled by ARCH_PROPOSER — MVP_SYNTHESIZER focuses on scope only."
2. Never auto-fill scope decisions. Surface the proposed call with rationale — consultant confirms or overrides.
3. Confidence levels: HIGH, MED, LOW only. No variants.
4. Effort signals (S/M/L/XL) are deferred to ARCH_PROPOSER. Do not produce them here — without architecture context they would be noise.
5. Readiness Gate: never skip it. Even if discovery.md looks complete, run the check. Every invocation.
6. LAYER_0_GLOBAL Rule 4 output limits apply: clarifying questions ≤3 lines, errors ≤2 lines.
7. LAYER_0_GLOBAL Rule 5 applies: no narration. Do not describe what you are about to do.
8. V1.1 boundary: never decompose into tickets, never produce an architecture diagram. Those are downstream skills.
