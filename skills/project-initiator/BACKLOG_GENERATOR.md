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

After project creation, show the team roster extracted from `## Sprint Mapping`:

```
Team roles from arch.md:

  Role           | arch.md name   | Odoo user ID
  ---------------|----------------|-------------
  BE-senior      | (name)         | ?
  BE-junior      | (name)         | ?
  FE-senior      | (name)         | ?
  FE-junior      | (name)         | ?
  QA             | (name)         | ?

Enter Odoo user ID for each role, or type "skip" to leave unassigned.
Note: the person must have an Odoo account to be assigned tickets.
```

Store completed role → assignee_id map. Roles marked "skip" → `assignee_ids: []`.

Do not ask again. If a role appears in multiple sprints, the same mapping applies throughout.

---

## Duplicate Check

Runs before the preview gate — always, regardless of new vs existing project.

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

**Preview table:** Duplicate-matched titles are marked `[EXISTS]` in the table even after
[A] is chosen, so the consultant sees what was skipped.

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
