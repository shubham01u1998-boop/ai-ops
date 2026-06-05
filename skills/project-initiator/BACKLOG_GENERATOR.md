# VERSION: 1.4 | Last updated: 2026-06-04 | Reviewed: pending
# BACKLOG_GENERATOR — Project Initiator V1.4
# Part of the fiftyfive-tech Project Initiator toolchain.

---

## Purpose

BACKLOG_GENERATOR reads `arch.md` from the engagement folder, maps team roles to Odoo users,
creates an Odoo project with stages, generates a structured ticket hierarchy
(parent ticket + subtasks per Build Order item), previews the full set before any creation,
gets explicit approval, then bulk-creates all tickets in Odoo via MCP tools.

No new architectural decisions are made — all data comes from arch.md.
BACKLOG_GENERATOR is an execution layer only: it never modifies arch.md or any upstream file.

V1.4 scope: this skill only. No orchestrator. No SESSION_STATE.md dependencies.

---

## Pre-flight

Before asking anything, silently:

1. Run `basename "$PWD"` via Bash tool. Use result as engagement name throughout.
   - If name is generic (e.g. `test`, `temp`, `folder`, `synthetic-01`): ask once —
     "What's the engagement or client name?" Store the answer. Do not ask again.

2. Use Read tool to load `arch.md` from current working directory.
   - If not found: surface this message and stop.

```
No arch.md found in this folder.

BACKLOG_GENERATOR requires a completed arch.md as input.
Run ARCH_PROPOSER first from this engagement folder, then run BACKLOG_GENERATOR.

Expected folder structure:
  ~/fiftyfive-engagements/<client-name>/
    arch.md     ← produced by ARCH_PROPOSER
    mvp-scope.md
    discovery.md
```

3. Validate arch.md contains required sections. If any are missing, surface which ones
   and stop:
```
arch.md is missing required sections: [list]
These sections are produced by ARCH_PROPOSER. Re-run ARCH_PROPOSER to regenerate arch.md.
```
Required sections: `## Sprint Mapping`, `## Build Order`, `## Components`, `## Tech Stack`.
Optional (used if present): `## Effort Signals`, `## Open Questions`, `## STRAWMAN Summary`, `## Integration Points`.

4. Extract from arch.md:
   - `## Sprint Mapping` → team roster + sprint dates + sprint-to-component assignments
   - `## Build Order` → ordered list (becomes parent tickets, one per item)
   - `## Components` → per-feature list (becomes subtasks under each parent)
   - `## Tech Stack` → layer decisions (used to infer backend vs frontend ticket type)
   - `## Effort Signals` → S/M/L/XL per feature (used for priority; XL → high)
   - `## Open Questions` → questions with `blocks:` fields (added to relevant tickets)
   - `## STRAWMAN Summary` → tentative decisions (become dedicated STRAWMAN tickets)
   - `## Integration Points` → table of systems + risk + open questions

5. After loading successfully, output one line:
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
    1. Backlog       — blocked items, pre-conditions
    2. To Do         — ready to pick up (default for new tickets)
    3. In Progress
    4. Bug           — reserved; not used by BACKLOG_GENERATOR at creation
    5. Done

  Use default stages? (yes / provide custom)
```

**If consultant accepts defaults:** use the 5-stage default set above.

**If consultant provides custom stages:** ask one follow-up per semantic role:
```
For each custom stage, what is its purpose?
  — Which stage is for newly created regular tickets? (equivalent to "To Do")
  — Which stage is for bug tickets? (equivalent to "Bug") — reserved for future use, not used by BACKLOG_GENERATOR at creation
  — Which stage is for blocked/pre-condition items? (equivalent to "Backlog")
```
Store answers as semantic role aliases. All ticket-creation logic uses
"To Do equivalent", "Bug equivalent", "Backlog equivalent" — never hardcoded stage names.

**Before creating the project:** Call `list_active_projects` to check if a project with
this name already exists.

If found:
```
⚠ A project named "<name>" already exists in Odoo.
  [A] Use existing project (will check for duplicate tickets before creating)
  [B] Create new project with a different name
  [C] Cancel
```
- [A]: store the existing project ID; run Duplicate Check against it; skip `create_project`.
- [B]: ask for a new name; re-check `list_active_projects`; then create.
- [C]: stop.

If not found: `create_project(name=<project_name>, stages=[<stage_list>])`
→ Output: `Project "<name>" created in Odoo.`

**Sprint tags:** Collect all unique sprint labels from `## Sprint Mapping`
(e.g., "Sprint 1-2", "Sprint 3-4", "Sprint 5", "Sprint 6"). After project creation,
create one Odoo tag per sprint label via `create_tag`. Store sprint label → tag_id map.

---

## Team Mapping

After project creation, show the team roster extracted from `## Sprint Mapping`.

Build this table dynamically from `## Sprint Mapping`. One row per unique role. If arch.md shows a human name, use it in the `arch.md name` column; if only a role label or seniority is given, use that.

```
Team roles from arch.md:

  Role           | arch.md name   | Odoo user ID
  ---------------|----------------|-------------
  <role from Sprint Mapping> | <name or label from Sprint Mapping> | ?
  (one row per unique role present in arch.md Sprint Mapping)

Enter Odoo user ID for each role, or type "skip" to leave unassigned.
Note: the person must have an Odoo account to be assigned tickets.
```

Store completed role → assignee_id map. Roles marked "skip" → `assignee_ids: []`.

Do not ask again. If a role appears in multiple sprints, the same mapping applies throughout.

---

## Duplicate Check

Runs before the preview gate — always, regardless of new vs existing project.

At this point, all parent ticket titles (one per Build Order item, formatted as `[Sprint X-Y] <item name>`) and STRAWMAN ticket titles (`⚠ STRAWMAN: Verify <decision> before Sprint 1`) have been generated internally. Run this check against that internal title list before showing the preview.

Call `list_tickets(project_id=<project_id>, limit=100)`.
Compare each generated ticket title (case-insensitive, exact match) against existing titles.

If duplicates found:
```
⚠ Duplicate check: <N> tickets already exist with matching titles:
  - "[Sprint 1-2] OracleConnector" — matches existing ticket #<id>
  - "[Sprint 3-4] SalesAPI + AlertEngine" — matches existing ticket #<id>

  [A] Skip duplicates (create only new tickets)
  [B] Cancel — do not create anything
  [C] Create anyway — will produce duplicates in Odoo
```
Consultant must explicitly respond before the preview table is shown.

**Preview table:** When [A] is chosen, duplicate-matched titles are marked `[EXISTS]` so the consultant sees what was skipped.
When [C] is chosen, all tickets (including duplicates) appear as normal rows — no `[EXISTS]` marker.
When [B] is chosen: output `Cancelled. No tickets created.` and stop.

If no duplicates found: proceed silently to preview.

---

## Ticket Type Inference

Infer ticket type from the component's tech annotation in `## Components` and the confirmed
tech in `## Tech Stack`.

| Component tech annotation | Ticket type | Auto-tag |
|---|---|---|
| React, Vue, Next.js, Flutter, React Native | frontend | `frontend` (tag ID 2) |
| Node.js, Express, .NET Core, FastAPI, Python | backend | `backend` (tag ID 44) |
| Azure App Service, AKS, Container Apps | backend | `backend` (tag ID 44) |
| Shared, connector, aggregator, job, worker | backend | `backend` (tag ID 44) |

If type cannot be determined: default to backend. Flag inline in preview:
`[type inferred — verify]`.

---

## Ticket Structure

Each parent ticket corresponds to one Build Order item.

### Title format
```
[Sprint N] <Build Order item name>
```
Where N = sprint number(s) from `## Sprint Mapping` for that Build Order item.
Example: `[Sprint 1-2] OracleConnector`

### Default stage at creation
- Regular tickets → To Do equivalent
- STRAWMAN tickets → Backlog equivalent (they are pre-conditions/blockers)

### Tags
Two tags per ticket:
1. **Type tag** — from Ticket Type Inference (`frontend` ID 2 or `backend` ID 44)
2. **Sprint tag** — from sprint label → tag_id map (created in Project Setup)
`tag_ids = [type_tag_id, sprint_tag_id]`

### Deadline
Compute sprint end date from Sprint Mapping timeline: take the global start date, add (sprint_number × sprint_length_weeks × 7 days). Use the sprint end date as the deadline.
If timeline or sprint dates cannot be determined from arch.md → no deadline set (not an error).

### Assignee
From role → Odoo user ID map (Team Mapping step).
If role not found or skipped → `assignee_ids: []`.
When a Build Order item is owned by multiple roles in Sprint Mapping, assign to the first-listed (typically senior) role. If unclear, leave `assignee_ids: []` for the consultant to assign in Odoo.

### Priority
- Default: `"0"` (low) for all tickets.
- XL effort signal in `## Effort Signals` → `"1"` (high).
- STRAWMAN tickets → `"1"` (high) — they block Sprint 1 start.

---

### Description format — Backend ticket

```markdown
## Scope
<1-2 sentences: what this component does and why it exists in this sprint>

## API Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| GET    | /example/route | <what it returns> |
| POST   | /example/route | <what it creates or updates> |

## Key Business Rules
- <rule derived from arch.md or mvp-scope.md context>

## Acceptance Criteria
- [ ] <testable pass/fail criterion>
- [ ] <testable pass/fail criterion>

## Edge Cases
- <edge case the developer must handle>

## Open Questions
- <question from arch.md ## Open Questions whose `blocks:` field names this Build Order item or any component under it>
(none if no matching open questions)
```

> If the component is a connector, job, worker, or aggregator (no HTTP endpoints): replace `## API Endpoints` with `## Interfaces` listing input sources, output targets, and schedule/trigger.

---

### Description format — Frontend ticket

```markdown
## Scope
<1-2 sentences: what screens/flows this component covers>

## Screens & Components
- <ScreenName> — <purpose, key UI elements, filters or controls>
- <ComponentName> — <reusable element, where used>

## Key Business Rules
- <rule>

## Acceptance Criteria
- [ ] <testable pass/fail criterion>
- [ ] <testable pass/fail criterion>

## Edge Cases
- <edge case>

## Open Questions
- <question from arch.md ## Open Questions whose `blocks:` field names this Build Order item or any component under it>
(none if no matching open questions)
```

---

### Subtasks
One subtask per component listed under this Build Order item in `## Components`.
Subtask title = component name + tech in parentheses.
Example: `OracleConnector (Node.js)`
Subtasks inherit the parent ticket's stage, assignee, and sprint tag.

**Shared components:** A component flagged as shared in arch.md (marked `**(shared)**` or
`**Shared:**`) gets a subtask only in the Build Order ticket where it is FIRST built.
"First built" = the Build Order item with the lowest sequence number (position in the Build Order list) that lists this component under `## Components`.
In all later parent tickets that use the same shared component, add a note in the
description only: `(reuses <ComponentName> — built in Sprint N)`.
Do not create duplicate subtasks for the same shared component.

---

## STRAWMAN Tickets

If `## STRAWMAN Summary` is absent from arch.md or contains no items, skip this section entirely — no STRAWMAN tickets are created and the STRAWMAN `bulk_create_tickets` call is omitted.

After all Build Order tickets are generated, create one ticket per item in `## STRAWMAN Summary`.

| Field | Value |
|---|---|
| Title | `⚠ STRAWMAN: Verify <decision> before Sprint 1` |
| Stage | Backlog equivalent (pre-condition, not ready-to-dev) |
| Tags | None (no type tag, no sprint tag) |
| Assignee | blank — triage owner assigns |
| Deadline | Sprint 1 start date from `## Sprint Mapping` |
| Priority | `"1"` (high) — these block Sprint 1 start |

STRAWMAN ticket description format:

```markdown
## Decision
<what the STRAWMAN decision is>

## Why Tentative
<rationale from arch.md ## STRAWMAN Summary>

## What Resolves This
<condition that would confirm or change this decision>

## Impact if Wrong
<which components or sprints are affected>
```

---

## Preview + Approval Gate

Before any MCP create call, run the Duplicate Check, then show the full ticket set:

```
Ticket preview — <engagement name> (<N> tickets + <M> STRAWMAN tickets):

Parent tickets:
| # | Title                                  | Type     | Sprint | Assignee      | Stage  |
|---|----------------------------------------|----------|--------|---------------|--------|
| 1 | [Sprint 1-2] OracleConnector           | Backend  | 1-2    | Sahil         | To Do  |
|   | ↳ OracleConnector (Node.js)            |          |        |               |        |
| 2 | [Sprint 1-2] SalesAggregator + data model | Backend | 1-2   | Vijay         | To Do  |
|   | ↳ SalesAggregator (Node.js)            |          |        |               |        |
| 3 | [Sprint 3-4] SalesAPI + AlertEngine    | Backend  | 3-4    | Sahil         | To Do  |
|   | ↳ SalesAPI (Node.js)                   |          |        |               |        |
|   | ↳ AlertEngine (Node.js)                |          |        |               |        |
|   |   (reuses OracleConnector — Sprint 1-2)|          |        |               |        |
| 4 | [Sprint 5] DashboardUI + AlertsUI      | Frontend | 5      | FE-senior     | To Do  |
|   | ↳ DashboardUI (React)                  |          |        |               |        |
|   | ↳ AlertsUI (React)                     |          |        |               |        |
| 5 | [Sprint 6] Offline mode layer          | Backend  | 6      | FE-senior     | To Do  |
|   | ↳ Offline mode (Service Worker)        |          |        |               |        |

STRAWMAN tickets (5):
|   | ⚠ STRAWMAN: Verify PostgreSQL...       | —        | —      | (unassigned)  | Backlog|
|   | ⚠ STRAWMAN: Verify Oracle POS method...| —        | —      | (unassigned)  | Backlog|
...

Create these <N> tickets in Odoo? [A] Yes / [B] Edit / [C] Cancel
```

Adapt the example table to the actual engagement's Build Order and team mapping.

**Natural-language handling:**
- Clear approval ("yes", "create them", "looks good") → [A]
- Edit request ("change sprint 5 ticket assignee to Tanu") → [B], apply changes, re-show the full preview table, then re-ask: `Create these <N> tickets in Odoo? [A] Yes / [B] Edit / [C] Cancel`. Repeat until [A] or [C].
- "cancel" or "stop" → [C], output `Cancelled. No tickets created.` and stop.

Per LAYER_0_GLOBAL Rule 1: this prompt is the permission gate. Do not write to Odoo
until the consultant explicitly approves.

---

## Bulk Creation

If any tickets were marked [EXISTS] during Duplicate Check, exclude them from all
`bulk_create_tickets` and `add_subtasks` calls — including STRAWMAN tickets.

On approval:

1. Call `bulk_create_tickets` for all parent tickets:
```
bulk_create_tickets(
  project_id=<id>,
  tickets=[
    {
      "title": "[Sprint 1-2] OracleConnector",
      "description": "<formatted markdown per Ticket Structure>",
      "stage_id": <To Do equivalent id>,
      "assignee_ids": [<id from team mapping>],
      "tag_ids": [44, <sprint-1-2-tag-id>],
      "priority": "0",
      "deadline": "<sprint end date>"
    },
    ...
  ]
)
```

2. For each parent ticket returned: call `add_subtasks` with that ticket's subtask list:
```
add_subtasks(ticket_id=<parent_id>, subtasks=["OracleConnector (Node.js)", ...])
```

3. Call `bulk_create_tickets` separately for STRAWMAN tickets (different stage):
```
bulk_create_tickets(
  project_id=<id>,
  tickets=[
    {
      "title": "⚠ STRAWMAN: Verify PostgreSQL before Sprint 1",
      "description": "<STRAWMAN format>",
      "stage_id": <Backlog equivalent id>,
      "assignee_ids": [],
      "tag_ids": [],
      "priority": "1",
      "deadline": "<sprint 1 start date>"
    },
    ...
  ]
)
```

Output progress inline:
```
Creating parent tickets... done (<N> tickets)
Adding subtasks... done (<P> subtasks)
Creating STRAWMAN tickets... done (<Q> tickets)
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
- ⚠ STRAWMAN: <decision> — source: BACKLOG_GENERATOR — resolve before Sprint 1
(one line per STRAWMAN item; "(none)" if no STRAWMANs)

## Notes
Odoo project: <name> (ID: <id>)
Tickets created: <N> parent + <P> subtasks + <Q> STRAWMAN
```

2. If `project.md` exists: update `Stage:` and `Last session:` fields only.
   Leave all other fields unchanged.

3. If `project.md` does not exist: skip step 2 silently.

Output one line: `session_state.md updated.`

---

## Rules for this skill

1. Never modify `arch.md` or any upstream artifact — BACKLOG_GENERATOR is execution layer only.
2. Never create tickets without explicit consultant approval at the preview gate.
3. Custom stages: always ask purpose mapping (To Do / Bug / Backlog equivalents) before
   ticket creation. Never guess stage semantics.
4. Default stage for new regular tickets = To Do equivalent. Never Backlog.
5. Default stage for STRAWMAN tickets = Backlog equivalent. They are pre-conditions,
   not ready-to-dev items.
6. Team mapping: consultant-provided Odoo user IDs are accepted as-is. Roles typed as "skip" → `assignee_ids: []`. No account verification is performed — consultant is responsible for entering valid IDs.
7. Duplicate check: always run `list_tickets` before showing the preview. Warn + ask approval.
   Never auto-proceed past duplicates.
8. Sprint tags: create one Odoo tag per sprint label after project creation. Every parent
   ticket and its subtasks get the sprint tag in `tag_ids`. Never omit sprint tags.
9. Bulk creation only — never call `create_ticket` (single) for the main ticket set.
10. STRAWMAN tickets use a separate `bulk_create_tickets` call (different stage + priority).
11. Shared components: one subtask in the ticket where the component is FIRST built.
    Later tickets reference it as text only — no duplicate subtask.
12. Open Questions in ticket descriptions: only those whose `blocks:` field names this
    Build Order item or any component under it. Do not copy all Open Questions into all tickets.
13. LAYER_0_GLOBAL Rule 4 output limits apply. Rule 5 (no narration) applies.
14. V1.4 boundary: never modify Odoo tickets after creation (consultant's job in Odoo).
    Never create Odoo tickets that belong to ROADMAP or BACKLOG phase 2.
