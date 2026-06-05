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
    4. Bug           — default stage for bug tickets at creation
    5. Done

  Use default stages? (yes / provide custom)
```

**If consultant accepts defaults:** use the 5-stage default set above.

**If consultant provides custom stages:** ask one follow-up per semantic role:
```
For each custom stage, what is its purpose?
  — Which stage is for newly created regular tickets? (equivalent to "To Do")
  — Which stage is for newly created bug tickets? (equivalent to "Bug")
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
Sprint end date from `## Sprint Mapping` for the sprint this item falls in.
If no sprint end date available → no deadline set (not an error).

### Assignee
From role → Odoo user ID map (Team Mapping step).
If role not found or skipped → `assignee_ids: []`.

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
- <question from arch.md ## Open Questions whose `blocks:` field names this component>
(none if no matching open questions)
```

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
- <question from arch.md ## Open Questions whose `blocks:` field names this component>
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
In all later parent tickets that use the same shared component, add a note in the
description only: `(reuses <ComponentName> — built in Sprint N)`.
Do not create duplicate subtasks for the same shared component.

---
