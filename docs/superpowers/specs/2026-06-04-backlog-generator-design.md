# BACKLOG_GENERATOR — Design Spec
**Skill version:** V1.4
**Date:** 2026-06-04
**Author:** Shubham Upadhyay

---

## Position in Chain

```
DISCOVERY (V1.0) → MVP_SYNTHESIZER (V1.1) → ARCH_PROPOSER (V1.3) → DOC_GENERATOR (V1.3.5) → BACKLOG_GENERATOR (V1.4) → ROADMAP (future)
```

---

## Purpose

BACKLOG_GENERATOR reads `arch.md` from the engagement folder, maps team roles to Odoo users,
creates an Odoo project with stages, generates a structured ticket hierarchy
(parent ticket + subtasks per Build Order item), previews the full set before any creation,
gets explicit approval, then bulk-creates all tickets in Odoo via MCP tools.

No new architectural decisions are made — all data comes from arch.md.
BACKLOG_GENERATOR is an execution layer only: it never modifies arch.md or any upstream file.

---

## Inputs

| File | Produced by | Required |
|---|---|---|
| `arch.md` | ARCH_PROPOSER | Yes |

`arch.md` must be present in the current working directory. If missing, skill stops with:
```
No arch.md found in this folder.

BACKLOG_GENERATOR requires a completed arch.md as input.
Run ARCH_PROPOSER first from this engagement folder, then run BACKLOG_GENERATOR.
```

---

## Outputs

All output goes to Odoo via MCP tools. Nothing written to the local filesystem except
`session_state.md` update.

| Output | Tool | Contents |
|---|---|---|
| Odoo project | `create_project` | Project name + stages |
| Parent tickets | `bulk_create_tickets` | One per Build Order item |
| Subtask tickets | `add_subtasks` | One per component under each Build Order item |
| STRAWMAN tickets | `bulk_create_tickets` | One per STRAWMAN item from arch.md |

---

## Pre-flight

Before anything else, silently:

1. Run `basename "$PWD"` via Bash tool. Use result as engagement name throughout.
   - If name is generic (e.g. `test`, `temp`, `folder`): ask once — "What's the engagement
     or client name?" Store the answer. Do not ask again.

2. Use Read tool to load `arch.md` from current working directory.
   - If not found: surface the message above and stop.

3. Validate that arch.md contains all required sections. If any are missing:
```
arch.md is missing required sections: [list]
These sections are produced by ARCH_PROPOSER. Re-run ARCH_PROPOSER to regenerate arch.md.
```
Required sections: `## Sprint Mapping`, `## Build Order`, `## Components`, `## Tech Stack`.
Optional (used if present): `## Effort Signals`, `## Open Questions`, `## STRAWMAN Summary`, `## Integration Points`.

4. Extract from arch.md:
   - `## Sprint Mapping` → team roster, sprint dates, sprint-to-component assignments
   - `## Build Order` → ordered list of items (these become parent tickets)
   - `## Components` → per-feature component list (these become subtasks)
   - `## Effort Signals` → S/M/L/XL per feature (used for priority)
   - `## Open Questions` → questions with `blocks:` fields
   - `## STRAWMAN Summary` → tentative decisions with rationale
   - `## Integration Points` → table of systems + risk + open questions
   - `## Tech Stack` → layer decisions (used to infer backend vs frontend ticket type)

5. After loading, output one line:
```
Engagement: <name> | arch.md loaded. Starting project setup.
```

---

## Project Setup

Ask in a single block:

```
Project setup for <engagement name>:

  Project name: [<engagement name>] — press Enter to accept or type a new name

  Stages (default):
    1. Backlog  — blocked items, pre-conditions
    2. To Do    — ready to pick up (default for new tickets)
    3. In Progress
    4. Bug      — default stage for bug tickets at creation
    5. Done

  Use default stages? (yes / provide custom)
```

**If consultant accepts defaults:** proceed with the 5-stage default set above.

**If consultant provides custom stages:** ask one follow-up per custom stage:

```
For each custom stage, what is its purpose?
  — Which stage is for newly created regular tickets? (equivalent to "To Do")
  — Which stage is for newly created bug tickets? (equivalent to "Bug")
  — Which stage is for blocked/pre-condition items? (equivalent to "Backlog")
```

Store the answers as semantic role aliases. All subsequent ticket-creation logic uses
"To Do equivalent", "Bug equivalent", "Backlog equivalent" — not hardcoded stage names.

**Sprint tags:** After extracting arch.md, collect all unique sprint labels from Sprint
Mapping (e.g., "Sprint 1", "Sprint 1-2", "Sprint 3-4"). Create one Odoo tag per sprint via
`create_tag` immediately after project creation. Store sprint label → tag_id for use in
all ticket creation calls.

**Before creating the project:** call `list_active_projects` and check if a project with
this name already exists.

If found:
```
⚠ A project named "<name>" already exists in Odoo.
  [A] Use existing project (check for duplicate tickets before creating)
  [B] Create new project with a different name
  [C] Cancel
```
- [A]: store the existing project ID; run the Idempotency Check against it; skip `create_project`.
- [B]: ask for a new name; re-check; then create.
- [C]: stop.

If not found: create the project: `create_project(name=<project_name>, stages=[<stage_list>])`

Output: `Project "<name>" created in Odoo.`

---

## Team Mapping

After project creation, show the team roster from `arch.md ## Sprint Mapping`:

```
Team roles from arch.md:

  Role                 | arch.md name     | Odoo user ID
  ---------------------|------------------|-------------
  BE-senior            | (from arch.md)   | ?
  BE-junior            | (from arch.md)   | ?
  FE-senior            | (from arch.md)   | ?
  FE-junior            | (from arch.md)   | ?
  QA                   | (from arch.md)   | ?

Enter Odoo user ID for each role, or type "skip" to leave unassigned.
Note: the person must have an Odoo account to be assigned tickets.
```

Store the completed role → assignee_id mapping. Roles marked "skip" → `assignee_ids: []`.

Do not ask this question again. If a role appears in multiple sprints, the same mapping applies.

---

## Duplicate Check

Runs at two points — always, regardless of new vs existing project.

**Point 1 — Before the preview gate:**
Call `list_tickets(project_id=<project_id>, limit=100)`.
Compare each generated ticket title against existing ticket titles (case-insensitive, exact
match). Collect all matches.

If duplicates found:
```
⚠ Duplicate check: <N> tickets already exist with matching titles:
  - "[Sprint 1-2] OracleConnector" — matches existing ticket #<id>
  - "[Sprint 3-4] SalesAPI" — matches existing ticket #<id>

  [A] Skip duplicates (create only new tickets)
  [B] Cancel — do not create anything
  [C] Create anyway — will produce duplicates in Odoo
```

Consultant must explicitly respond before the preview table is shown.

**Point 2 — Preview table:** Duplicate-matched titles are marked `[EXISTS]` in the table
even after [A] is chosen, so the consultant sees what was skipped.

If no duplicates found: proceed silently to preview.

---

## Ticket Type Inference

Infer ticket type (backend / frontend / infra / shared) from the component's tech layer
using `arch.md ## Tech Stack` and the component's tech annotation in `arch.md ## Components`.

| Component tech annotation | Ticket type | Auto-tag |
|---|---|---|
| React, Vue, Next.js, Flutter | frontend | `frontend (2)` |
| Node.js, .NET, FastAPI, Python | backend | `backend (44)` |
| Azure App Service, AKS, infra | backend | `backend (44)` |
| Shared, connector, aggregator | backend | `backend (44)` |

If type cannot be determined: default to backend. Flag inline: `[type inferred — verify]`.

---

## Ticket Structure

Each parent ticket (one per Build Order item) uses this structure.

### Title format
```
[Sprint N] <Build Order item name>
```
Where N = sprint number(s) from arch.md Sprint Mapping for that item.

### Default stage at creation
- Regular tickets → To Do equivalent
- Bug tickets → Bug equivalent
- STRAWMAN tickets → Backlog equivalent (they are pre-conditions/blockers)

### Tags
Two tags per ticket (both set at creation, no manual input required):
1. **Type tag** — auto-assigned from inferred ticket type (`frontend (2)` or `backend (44)`)
2. **Sprint tag** — assigned from the sprint label → tag_id map created in Project Setup

`tag_ids = [type_tag_id, sprint_tag_id]`

### Deadline
Sprint end date from arch.md Sprint Mapping for the sprint this item falls in.
If no sprint end date available → no deadline set (not an error).

### Assignee
Mapped from role → Odoo user ID from Team Mapping step.
If role not found or skipped → `assignee_ids: []`.

### Priority
- Default: `"0"` (low) for all tickets.
- Exception: Build Order items with XL effort signal in `arch.md ## Effort Signals` → `"1"` (high).
- STRAWMAN tickets: `"1"` (high) — they block Sprint 1 start.

### Description format — Backend ticket

```markdown
## Scope
<1-2 sentences: what this component does and why it exists in this sprint>

## API Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| GET    | /example/path | <what it returns> |
| POST   | /example/path | <what it creates/updates> |

## Key Business Rules
- <rule derived from mvp-scope.md or arch.md context>

## Acceptance Criteria
- [ ] <testable criterion>
- [ ] <testable criterion>

## Edge Cases
- <edge case to handle>

## Open Questions
- <question from arch.md ## Open Questions that has `blocks: <this component>`>
  (none if no matching open questions)
```

### Description format — Frontend ticket

```markdown
## Scope
<1-2 sentences: what screens/flows this component covers>

## Screens & Components
- <ScreenName> — <purpose and key UI elements>
- <ComponentName> — <reusable piece, where used>

## Key Business Rules
- <rule>

## Acceptance Criteria
- [ ] <testable criterion>

## Edge Cases
- <edge case>

## Open Questions
- <question from arch.md ## Open Questions that blocks this component>
  (none if no matching open questions)
```

### Sprint Tag
Every ticket is tagged with its sprint (e.g., `Sprint 1`, `Sprint 1-2`). Sprint tags are
created via `create_tag` during Project Setup, one per unique sprint in arch.md Sprint
Mapping, before any ticket creation. Stored as sprint label → tag_id map. Added to
`tag_ids` alongside the type tag (frontend/backend).

### Subtasks
One subtask per component listed under this Build Order item in `arch.md ## Components`.
Subtask title = component name.
Subtasks inherit parent ticket's stage, assignee, and sprint tag.

**Shared components:** A component flagged as shared in arch.md (e.g. marked `**Shared:**`)
gets a subtask only in the Build Order ticket where it is FIRST built. In all later parent
tickets that reference the same shared component, add a text note in the description only:
`(reuses <ComponentName> — built in Sprint N)`. Do not create duplicate subtasks.

---

## STRAWMAN Tickets

After all Build Order tickets are generated, generate one ticket per item in
`arch.md ## STRAWMAN Summary`.

| Field | Value |
|---|---|
| Title | `⚠ STRAWMAN: Verify <decision> before Sprint 1` |
| Stage | Backlog equivalent — these are pre-conditions, not ready-to-dev |
| Tags | None |
| Assignee | Leave blank — triage owner assigns |
| Deadline | Sprint 1 start date from arch.md Sprint Mapping |
| Description | STRAWMAN rationale from arch.md + what would change this decision |

STRAWMAN ticket description format:
```markdown
## Decision
<what the STRAWMAN decision is>

## Why Tentative
<rationale from arch.md STRAWMAN Summary>

## What Resolves This
<condition that would confirm or change this decision>

## Impact if Wrong
<which components or sprints are affected>
```

---

## Preview + Approval Gate

Before any MCP create call, show the full ticket set as a summary table:

```
Ticket preview — <engagement name> (<N> tickets + <M> STRAWMAN tickets):

Parent tickets:
| # | Title                              | Type     | Sprint | Assignee         | Stage  |
|---|------------------------------------|----------|--------|------------------|--------|
| 1 | [Sprint 1-2] OracleConnector       | Backend  | 1-2    | Sahil (BE-sr)    | To Do  |
|   | ↳ OracleConnector setup (subtask)  |          |        |                  |        |
| 2 | [Sprint 1-2] SalesAggregator       | Backend  | 1-2    | Vijay (BE-jr)    | To Do  |
|   | ↳ SalesAggregator job (subtask)    |          |        |                  |        |
...

STRAWMAN tickets:
| | ⚠ STRAWMAN: Verify Oracle REST API before Sprint 1 | — | — | (unassigned) | Backlog |
...

Create these <N> tickets in Odoo? [A] Yes / [B] Edit / [C] Cancel
```

**Natural-language handling:**
- Clear approval ("yes", "create them", "looks good") → interpret as [A]
- Edit request ("change sprint 3 ticket to Vijay") → interpret as [B], apply, re-show table
- "cancel" or "stop" → interpret as [C], do not create anything

---

## Bulk Creation

On approval:

1. Call `bulk_create_tickets` with all parent tickets as a list:
   ```
   bulk_create_tickets(
     project_id=<id>,
     tickets=[
       {title, description, stage_id, assignee_ids, tag_ids, priority, deadline},
       ...
     ]
   )
   ```

2. For each parent ticket returned: call `add_subtasks` with the subtask list.

3. Call `bulk_create_tickets` separately for STRAWMAN tickets (different stage).

Output progress inline:
```
Creating parent tickets... done (8 tickets)
Adding subtasks... done (14 subtasks)
Creating STRAWMAN tickets... done (3 tickets)
```

---

## Post-Creation Output

```
BACKLOG_GENERATOR complete.

Created:
  • <N> parent tickets across <M> sprints
  • <P> subtasks
  • <Q> STRAWMAN pre-condition tickets

Project: <name> (Odoo ID: <id>)
Stages: <stage list>

Next steps:
  1. Review tickets in Odoo and assign any unassigned roles.
  2. Resolve STRAWMAN tickets before Sprint 1 begins.
  3. Run ROADMAP skill when ready (not yet built — V1.5).
```

---

## Write Session State

After creation completes, silently:

1. Write/update `session_state.md` in the current directory:

```markdown
# Session State — <engagement name>
# Updated: <YYYY-MM-DD>

## Current Stage
Last completed: BACKLOG_GENERATOR
Status: complete

## Next Step
Run: ROADMAP (not yet built — V1.5)
From: <current directory path>

## Open Items
<list each STRAWMAN item as:>
- ⚠ STRAWMAN: <decision> — source: BACKLOG_GENERATOR — resolve before Sprint 1
If no STRAWMANs: (none)

## Notes
Odoo project: <name> (ID: <id>)
Tickets created: <N> parent + <P> subtasks + <Q> STRAWMAN
```

2. If `project.md` exists: update `Stage:` and `Last session:` fields only.

3. Output one line: `session_state.md updated.`

---

## Rules

1. Never modify `arch.md` or any upstream artifact — BACKLOG_GENERATOR is execution layer only.
2. Never create tickets without explicit consultant approval at the preview gate.
3. Custom stages: always ask purpose mapping before ticket creation — never guess.
4. Default stage for new tickets = To Do equivalent (not Backlog).
5. Default stage for bug tickets = Bug equivalent.
6. Default stage for STRAWMAN tickets = Backlog equivalent (they are blockers, not ready-to-dev).
7. Team mapping: confirm Odoo account exists before binding. Unconfirmed roles → blank assignee.
8. Duplicate check: always run `list_tickets` before showing the preview. Warn + ask approval before any creation. Never skip, never auto-proceed past duplicates.
8a. Sprint tags: create one Odoo tag per sprint label after project creation. Every ticket gets its sprint tag in `tag_ids`. Never leave a ticket without a sprint tag.
9. Bulk creation only — never call `create_ticket` one at a time for the main ticket set.
10. STRAWMAN tickets get their own `bulk_create_tickets` call (different stage from parent tickets).
11. Open Questions in ticket descriptions: only include questions whose `blocks:` field names this component.
12. LAYER_0_GLOBAL Rule 4 output limits apply. Rule 5 (no narration) applies.
13. V1.4 boundary: never modify Odoo tickets after creation (that is the consultant's job in Odoo).
