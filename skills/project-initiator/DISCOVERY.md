# VERSION: 1.0 | Last updated: 2026-06-03 | Reviewed: ✅
# DISCOVERY — Project Initiator V1.0
# Part of the fiftyfive-tech Project Initiator toolchain.
# Phase 1 skill files (LAYER_0_GLOBAL, LAYER_2_FASTPATH, QA_INTAKE) are separate — do not modify them here.

---

## Purpose

DISCOVERY reads rough, partial, or informal input documents — specs, meeting notes, emails, requirement fragments — and produces a structured `discovery.md` artifact through a guided conversation with the consultant.

DISCOVERY only extracts and clarifies. It does not choose architecture, define MVP scope, select tech stacks, or decompose into tickets. Those are downstream skills (target vision: MVP_SYNTHESIZER → ARCH_PROPOSER → BACKLOG_GENERATOR). The output is a clean, human-reviewed discovery document ready to anchor the rest of the engagement.

V1.0 scope: this skill only. No orchestrator wraps it. No SESSION_STATE.md. Run it from an engagement folder that has an `input/` subfolder.

---

## Pre-flight

Before asking anything, silently:

0. **Detect engagement name.** Run `basename "$PWD"` via Bash tool. Use the result as the engagement name throughout the session and in the `discovery.md` header.
   - If the name is generic (e.g. `test`, `temp`, `folder`, `project`, `discovery`, `input`, `docs`) or looks like a path fragment: ask once — "What's the engagement or client name for this project?" Store the answer. Do not ask again.

1. Use the Glob tool with pattern `input/*` to list all files in the input folder.
   - If Glob returns results: proceed — `input/` exists and contains files.
   - If Glob returns nothing: surface the message below and stop.
2. Group the Glob results by file extension before reading.
3. For each file:
   - `.pdf`, `.txt`, `.md`, `.markdown` → read using Read tool.
   - `.docx`, `.doc` → skip with: `Skipped <filename> — parse_docx MCP tool not yet built. Convert to PDF or text and re-run.`
   - Any other extension → skip with one line: `Skipped <filename> — unsupported format.`
4. After reading all readable files, output one line:

```
Engagement: <name> | Read N input doc(s) — ~M words total. Starting extraction.
```

If no readable files found (folder missing or empty):
```
No input/ folder found (or folder is empty).

Expected folder structure:
  ~/fiftyfive-engagements/<client-name>/
    input/    ← put your docs here (PDF, text, markdown)
    (discovery.md will be saved here when done)

Open Claude Code from the <client-name>/ folder and run DISCOVERY again.
If you're working on multiple engagements, each client gets its own folder.
```
Stop. Do not proceed without input docs.

---

## Extraction Pass

Read all input content silently. Build an internal extraction. Do not show this verbatim to the consultant — it is working state.

Extract per category:

| Category | What to extract |
|---|---|
| Project type / domain | What kind of system: web app, mobile app, platform, API, internal tool, etc. |
| Primary users | Who uses it, what their roles are, rough scale if mentioned |
| Core problem | The central pain point or business need being addressed |
| Features mentioned | Explicit features, functions, or capabilities described anywhere in the docs |
| Tech mentioned | Frameworks, languages, platforms, databases, third-party integrations |
| Timeline | Dates, deadlines, delivery windows, milestones |
| Constraints | Budget, team size, regulatory, performance, geographic, other non-negotiables |
| Conflicts | The same topic stated differently in different docs (e.g., "React" in spec, "Vue" in notes) |
| Critical gaps | Topics where downstream phases genuinely cannot proceed without an answer |
| Detail gaps | Topics that are useful but can be deferred to Open Questions if consultant doesn't know |

For each category, assign a confidence marker:
- **HIGH** — stated explicitly and consistently across multiple docs
- **MED** — stated in only one doc, or clearly inferred from context
- **LOW** — not mentioned, or in direct conflict across docs

---

## Initial Summary

Present the extraction to the consultant in this compact block. Then immediately begin the question loop — do not wait for a response to the summary.

```
Extracted from N doc(s):

Project:     <description> [HIGH/MED/LOW]
Users:       <description> [HIGH/MED/LOW]
Problem:     <description> [HIGH/MED/LOW]
Features:    <list — comma-separated brief items> [HIGH/MED/LOW]
Tech:        <list — flag CONFLICT: <topic> if same decision appears differently across docs> [HIGH/MED/LOW]
Timeline:    <description, or "not mentioned"> [HIGH/MED/LOW]
Constraints: <list, or "none found"> [HIGH/MED/LOW]

Conflicts: N | Critical gaps: N | Detail gaps: N
```

---

## Question Loop

One question at a time. Strict order:

### Phase 1 — Resolve conflicts first

For each conflict detected:
```
[Doc A] says <X>. [Doc B] says <Y> for <topic>. Which is current?
```

### Phase 2 — Critical gaps

Topics where MVP_SYNTHESIZER or ARCH_PROPOSER genuinely cannot proceed without an answer. Ask why it matters briefly if the consultant might not see the connection:
```
<Topic> is not mentioned in the docs. <One sentence on why it matters downstream.> <Specific question>?
```

### Phase 3 — Detail gaps

Nice-to-have. Explicitly tell the consultant they can defer:
```
<Topic> — <specific question>? (You can defer this — it goes to Open Questions.)
```

### Batching exception

When 2–4 questions share the same tight topic (e.g. scale: users + volume + storage), present them grouped with a clear label. Do not batch across topics.

```
Scale (3 questions):
a) <question>
b) <question>
c) <question>
```

### Handling consultant responses

- Gives answer → update internal extraction, mark resolved, move to next question.
- Says "defer" or "I don't know" → add to Open Questions list, move to next question.
- Says "stop" or "that's enough" → skip remaining questions, go to Draft Artifact immediately.

After all questions are exhausted or stopped, proceed to Draft Artifact.

---

## Draft Artifact

Produce the draft and show it to the consultant. Use this exact format:

```markdown
# Discovery — <engagement name>
# Generated: <YYYY-MM-DD> | Reviewed by: <consultant name if known>

## Project Context
<Business context and problem statement. 2–4 sentences.>

## Users
**Primary:** <who they are, scale if known>
**Secondary:** <if known, or omit section>

## Core Problem
<What is being solved and why it matters. 2–3 sentences.>

## Features Mentioned
- <Feature name> — <brief description> [HIGH/MED/LOW]
- <Feature name> — <brief description> [HIGH/MED/LOW]
(add one line per feature from extraction)

## Constraints
- Timeline: <date or window, or "not specified">
- Budget: <if known, or omit>
- Tech: <confirmed items, or "deferred / TBD per item">
- Other: <any hard constraints found — data residency, compliance, etc.>

## Open Questions
1. <Question text> — <why deferred: "consultant deferred" / "not mentioned in docs">
2. ...

## Confidence Notes
- <Category>: LOW — <why confidence is low>
- <Category>: CONFLICT resolved — <what was decided and by whom>
(omit categories where confidence was HIGH and no conflict occurred)

## Source Docs
- <filename> — <one-line summary of what this doc contained>
```

After showing draft, suggest the most natural next step based on the conversation so far:
```
Looks complete — save discovery.md? [A] Yes / [B] Edit a section / [C] Add more context
```

If the consultant responds without using [A]/[B]/[C] exactly:
- Clear approval ("yes", "save it", "looks good", "correct", "perfect", "go ahead") → interpret as [A]. State: "Saving discovery.md." and proceed to Save.
- Edit or correction offered ("change X to Y", "the timeline is wrong", etc.) → interpret as [B]. Apply the change, re-show the affected section, re-offer the same prompt.
- New information offered ("also, I forgot to mention...") → interpret as [C]. Treat as a new answer in the question loop. Update extraction silently, then re-show the draft with changes applied.
- Ambiguous → surface the intent: "Reading this as [A] Save — correct? (yes / [B] edit a section / [C] add more context)"

---

## Save

Per LAYER_0_GLOBAL Rule 1: the Draft Artifact prompt is the permission gate. Do not write `discovery.md` until the consultant explicitly approves — via [A], unambiguous natural-language approval ("yes", "save it", "looks good"), or confirmation after an ambiguous-intent check. Ambiguous responses must surface intent first before writing.

Write `discovery.md` to the current working directory.

Output:
```
discovery.md saved. DISCOVERY complete.
```

---

## Rules for this skill

1. Never choose architecture, MVP scope, or tech stack. Surface options and conflicts — the consultant decides.
2. Never auto-fill a gap. If the answer is not in the docs and the consultant doesn't provide it, it becomes an Open Question.
3. Confidence levels: HIGH, MED, LOW only. No "VERY HIGH", no "MEDIUM-LOW", no other variants.
4. When docs conflict on the same topic, name both docs and ask explicitly — do not silently pick one.
5. LAYER_0_GLOBAL Rule 4 output limits apply: clarifying questions ≤3 lines, errors ≤2 lines.
6. LAYER_0_GLOBAL Rule 5 applies: no narration. Do not describe what you are about to do.
7. V1.0 boundary: never decompose into tickets, never produce an architecture, never scope the MVP. Those are downstream skills. If asked, respond: "That's handled by a later skill — DISCOVERY focuses on extraction only."
