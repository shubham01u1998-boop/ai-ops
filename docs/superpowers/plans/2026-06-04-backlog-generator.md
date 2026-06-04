# BACKLOG_GENERATOR Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `skills/project-initiator/BACKLOG_GENERATOR.md` — a skill file that reads `arch.md`, creates an Odoo project, maps team roles to Odoo users, generates a structured ticket hierarchy with rich descriptions, and bulk-creates all tickets via MCP tools.

**Architecture:** BACKLOG_GENERATOR is a markdown skill file (not code) that Claude executes as instructions. It follows the same pattern as ARCH_PROPOSER.md and DOC_GENERATOR.md: pre-flight → conversation sections → approval gate → MCP write actions → session state update. Testing is done via QA simulation against synthetic fixtures.

**Tech Stack:** Markdown (skill file), Odoo MCP tools (`create_project`, `bulk_create_tickets`, `add_subtasks`, `create_tag`, `list_active_projects`, `list_tickets`), Claude Code CLI.

---

## Cold-Start Brief

**Read these files in order before starting any task:**

1. `CLAUDE.md` — project overview, phase status, Odoo connection details (project ID 58, stage IDs, team user IDs), and the Project Initiator chain
2. `docs/superpowers/specs/2026-06-04-backlog-generator-design.md` — the full design spec for BACKLOG_GENERATOR V1.4 (this is the authoritative source for all content)
3. `skills/project-initiator/ARCH_PROPOSER.md` — read as a skill file pattern reference (417 lines, same structure to follow)
4. `skills/project-initiator/DOC_GENERATOR.md` — read as a skill file pattern reference (758 lines, shows Review & Save and Rules patterns)
5. `tests/fixtures/arch-proposer/synthetic-01/arch.md` — the RetailEdge arch.md; BACKLOG_GENERATOR will read this file as input in both test fixtures

**Key facts you need:**
- Skill files are markdown instruction files that Claude executes. There is no Python or JavaScript to write.
- The "test" for a skill file is a QA simulation: you simulate what Claude would do when running the skill against a fixture, check every checkbox in `expected_behaviors.md`, and write a `test_report.md`.
- Odoo MCP tools available: `create_project`, `bulk_create_tickets`, `add_subtasks`, `create_tag`, `list_active_projects`, `list_tickets`, `create_stage`, `create_ticket` (single), `get_ticket`
- Odoo connection: instance `fiftyfive-technologies-pvt-ltd.odoo.com`, project TiffinConnect ID=58
- Team user IDs (for test fixtures): Sahil=50, Vijay=62, Kunal=41, Tanu=57, Shubham=42
- Stage IDs (TiffinConnect): Backlog=347, To Do=348, In Progress=349, Bug=350, Done=351

---

## File Structure

| Action | Path | Purpose |
|---|---|---|
| Create | `skills/project-initiator/BACKLOG_GENERATOR.md` | The skill file itself — all sections |
| Create | `tests/fixtures/backlog-generator/TEST_SCRIPT.md` | Instructions for running both fixtures |
| Create | `tests/fixtures/backlog-generator/synthetic-01/arch.md` | RetailEdge arch.md (copy from arch-proposer) |
| Create | `tests/fixtures/backlog-generator/synthetic-01/expected_behaviors.md` | PASS-path QA checklist |
| Create | `tests/fixtures/backlog-generator/synthetic-01/test_report.md` | Written during Task 9 QA simulation |
| Create | `tests/fixtures/backlog-generator/synthetic-02-duplicate/arch.md` | Same RetailEdge arch.md |
| Create | `tests/fixtures/backlog-generator/synthetic-02-duplicate/expected_behaviors.md` | DUPLICATE-path QA checklist |
| Create | `tests/fixtures/backlog-generator/synthetic-02-duplicate/test_report.md` | Written during Task 10 QA simulation |
| Modify | `CLAUDE.md` | Add BACKLOG_GENERATOR V1.4 as done in Phase Status + chain status |

---

## Task 1: Cold-Start — Read context files

**Files:** Read only. No writes in this task.

- [ ] **Step 1: Read CLAUDE.md**

  Read `CLAUDE.md`. Confirm you understand: (a) the Project Initiator chain, (b) Odoo connection details, (c) team user IDs, (d) stage IDs.

- [ ] **Step 2: Read the design spec**

  Read `docs/superpowers/specs/2026-06-04-backlog-generator-design.md`. This is the authoritative source for all skill content. Understand every section before writing.

- [ ] **Step 3: Read ARCH_PROPOSER.md as pattern reference**

  Read `skills/project-initiator/ARCH_PROPOSER.md`. Note: (a) VERSION header format, (b) Pre-flight structure, (c) how sections flow into each other, (d) Rules section format.

- [ ] **Step 4: Read the RetailEdge arch.md that will be used in fixtures**

  Read `tests/fixtures/arch-proposer/synthetic-01/arch.md`. This is the input file BACKLOG_GENERATOR will process. Understand its Build Order (5 items), Components, Sprint Mapping, STRAWMAN Summary (5 items), Open Questions (4 items).

---

## Task 2: Skill file — Part 1 (VERSION header + Purpose + Pre-flight + Project Setup)

**Files:**
- Create: `skills/project-initiator/BACKLOG_GENERATOR.md`

- [ ] **Step 1: Write the VERSION header and Purpose section**

  Write the opening of `skills/project-initiator/BACKLOG_GENERATOR.md`:

  ```markdown
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
  ```

- [ ] **Step 2: Write the Pre-flight section**

  Append to the file (content from spec §Pre-flight):

  ```markdown
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
  Optional (used if present): `## Effort Signals`, `## Open Questions`, `## STRAWMAN Summary`.

  4. Extract from arch.md:
     - `## Sprint Mapping` → team roster + sprint dates + sprint-to-component assignments
     - `## Build Order` → ordered list (becomes parent tickets, one per item)
     - `## Components` → per-feature list (becomes subtasks under each parent)
     - `## Tech Stack` → layer decisions (used to infer backend vs frontend ticket type)
     - `## Effort Signals` → S/M/L/XL per feature (used for priority; XL → high)
     - `## Open Questions` → questions with `blocks:` fields (added to relevant tickets)
     - `## STRAWMAN Summary` → tentative decisions (become dedicated STRAWMAN tickets)

  5. After loading successfully, output one line:
  ```
  Engagement: <name> | arch.md loaded. Starting project setup.
  ```

  ---
  ```

- [ ] **Step 3: Write the Project Setup section**

  Append (content from spec §Project Setup):

  ```markdown
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

  **Sprint tags:** Collect all unique sprint labels from `## Sprint Mapping`
  (e.g., "Sprint 1-2", "Sprint 3-4", "Sprint 5", "Sprint 6"). After project creation,
  create one Odoo tag per sprint label via `create_tag`. Store sprint label → tag_id map.

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

  Output: `Project "<name>" created in Odoo.`

  ---
  ```

- [ ] **Step 4: Review Part 1 against spec**

  Re-read `docs/superpowers/specs/2026-06-04-backlog-generator-design.md` §Pre-flight and §Project Setup.
  Confirm: (a) generic folder name handling present, (b) arch.md missing message exact, (c) required vs optional sections split correct, (d) sprint tag creation placed after project creation, (e) `list_active_projects` called before `create_project`.

- [ ] **Step 5: Commit Part 1**

  ```bash
  git add skills/project-initiator/BACKLOG_GENERATOR.md
  git commit -m "feat(backlog-generator): pre-flight + project setup sections"
  ```

---

## Task 3: Skill file — Part 2 (Team Mapping + Duplicate Check + Ticket Type Inference)

**Files:**
- Modify: `skills/project-initiator/BACKLOG_GENERATOR.md`

- [ ] **Step 1: Append Team Mapping section**

  ```markdown
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
  ```

- [ ] **Step 2: Append Duplicate Check section**

  ```markdown
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
  ```

- [ ] **Step 3: Append Ticket Type Inference section**

  ```markdown
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
  ```

- [ ] **Step 4: Review Part 2 against spec**

  Confirm: (a) Team Mapping shows arch.md names alongside role labels, (b) Duplicate Check runs before preview (not after), (c) type inference table covers all tech annotations that appear in RetailEdge arch.md (Node.js, React, Azure App Service).

- [ ] **Step 5: Commit Part 2**

  ```bash
  git add skills/project-initiator/BACKLOG_GENERATOR.md
  git commit -m "feat(backlog-generator): team mapping + duplicate check + type inference"
  ```

---

## Task 4: Skill file — Part 3 (Ticket Structure — the core section)

**Files:**
- Modify: `skills/project-initiator/BACKLOG_GENERATOR.md`

- [ ] **Step 1: Append Ticket Structure section header + metadata fields**

  ```markdown
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
  ```

- [ ] **Step 2: Append Backend ticket description format**

  ```markdown
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
  ```

- [ ] **Step 3: Append Frontend ticket description format**

  ```markdown
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
  ```

- [ ] **Step 4: Append Subtasks section**

  ```markdown
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
  ```

- [ ] **Step 5: Review Part 3 against spec**

  Verify: (a) title format matches spec `[Sprint N]` prefix, (b) both description formats have all 6 sections (Scope / endpoints or screens / Business Rules / Acceptance Criteria / Edge Cases / Open Questions), (c) shared component subtask rule present, (d) priority rule for XL present.

- [ ] **Step 6: Commit Part 3**

  ```bash
  git add skills/project-initiator/BACKLOG_GENERATOR.md
  git commit -m "feat(backlog-generator): ticket structure + description formats"
  ```

---

## Task 5: Skill file — Part 4 (STRAWMAN Tickets + Preview Gate + Bulk Creation + Post-Creation + Session State + Rules)

**Files:**
- Modify: `skills/project-initiator/BACKLOG_GENERATOR.md`

- [ ] **Step 1: Append STRAWMAN Tickets section**

  ```markdown
  ## STRAWMAN Tickets

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
  ```

- [ ] **Step 2: Append Preview + Approval Gate section**

  ```markdown
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
  - Edit request ("change sprint 5 ticket assignee to Tanu") → [B], apply, re-show table
  - "cancel" or "stop" → [C], do not create anything

  Per LAYER_0_GLOBAL Rule 1: this prompt is the permission gate. Do not write to Odoo
  until the consultant explicitly approves.

  ---
  ```

- [ ] **Step 3: Append Bulk Creation section**

  ```markdown
  ## Bulk Creation

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
  ```

- [ ] **Step 4: Append Post-Creation Output + Write Session State sections**

  ```markdown
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
  ```

- [ ] **Step 5: Append Rules for this skill section**

  ```markdown
  ## Rules for this skill

  1. Never modify `arch.md` or any upstream artifact — BACKLOG_GENERATOR is execution layer only.
  2. Never create tickets without explicit consultant approval at the preview gate.
  3. Custom stages: always ask purpose mapping (To Do / Bug / Backlog equivalents) before
     ticket creation. Never guess stage semantics.
  4. Default stage for new regular tickets = To Do equivalent. Never Backlog.
  5. Default stage for STRAWMAN tickets = Backlog equivalent. They are pre-conditions,
     not ready-to-dev items.
  6. Team mapping: verify Odoo account exists before binding. Unconfirmed roles → blank assignee.
  7. Duplicate check: always run `list_tickets` before showing the preview. Warn + ask approval.
     Never auto-proceed past duplicates.
  8. Sprint tags: create one Odoo tag per sprint label after project creation. Every parent
     ticket and its subtasks get the sprint tag in `tag_ids`. Never omit sprint tags.
  9. Bulk creation only — never call `create_ticket` (single) for the main ticket set.
  10. STRAWMAN tickets use a separate `bulk_create_tickets` call (different stage + priority).
  11. Shared components: one subtask in the ticket where the component is FIRST built.
      Later tickets reference it as text only — no duplicate subtask.
  12. Open Questions in ticket descriptions: only those whose `blocks:` field names this
      specific component. Do not copy all Open Questions into all tickets.
  13. LAYER_0_GLOBAL Rule 4 output limits apply. Rule 5 (no narration) applies.
  14. V1.4 boundary: never modify Odoo tickets after creation (consultant's job in Odoo).
      Never create Odoo tickets that belong to ROADMAP or BACKLOG phase 2.
  ```

- [ ] **Step 6: Review full skill file against spec**

  Read through `skills/project-initiator/BACKLOG_GENERATOR.md`. Check every spec section has a corresponding skill section. Confirm section count matches: Purpose, Pre-flight, Project Setup, Team Mapping, Duplicate Check, Ticket Type Inference, Ticket Structure, STRAWMAN Tickets, Preview + Approval Gate, Bulk Creation, Post-Creation Output, Write Session State, Rules (13 sections).

- [ ] **Step 7: Commit Part 4**

  ```bash
  git add skills/project-initiator/BACKLOG_GENERATOR.md
  git commit -m "feat(backlog-generator): STRAWMAN tickets + preview gate + bulk creation + rules — skill file complete"
  ```

---

## Task 6: Create synthetic-01 fixture (PASS path — new project, no duplicates)

**Files:**
- Create: `tests/fixtures/backlog-generator/synthetic-01/arch.md`
- Create: `tests/fixtures/backlog-generator/synthetic-01/expected_behaviors.md`

The input arch.md is the same RetailEdge file from arch-proposer. Copy its content exactly.

- [ ] **Step 1: Copy arch.md into the fixture folder**

  Read `tests/fixtures/arch-proposer/synthetic-01/arch.md`.
  Write its content to `tests/fixtures/backlog-generator/synthetic-01/arch.md`.

- [ ] **Step 2: Write expected_behaviors.md for synthetic-01**

  Write `tests/fixtures/backlog-generator/synthetic-01/expected_behaviors.md`:

  ```markdown
  # Expected Behaviors — BACKLOG_GENERATOR synthetic-01
  # Fixture: RetailEdge PASS path — new project, default stages, no duplicates

  ## Pre-flight

  - [ ] BG-01: `basename "$PWD"` returns "synthetic-01" (generic) → skill asks for engagement name
  - [ ] BG-02: Consultant provides "RetailEdge" → stored and used throughout
  - [ ] BG-03: arch.md loaded from working directory
  - [ ] BG-04: Required sections present: Sprint Mapping, Build Order, Components, Tech Stack
  - [ ] BG-05: Extracted 5 Build Order items
  - [ ] BG-06: Extracted 4 Open Questions
  - [ ] BG-07: Extracted 5 STRAWMAN Summary items
  - [ ] BG-08: Outputs single line: `Engagement: RetailEdge | arch.md loaded. Starting project setup.`

  ## Project Setup

  - [ ] BG-09: Shows single-block ask with project name defaulting to "RetailEdge"
  - [ ] BG-10: Shows default 5-stage list (Backlog, To Do, In Progress, Bug, Done) with descriptions
  - [ ] BG-11: Consultant accepts defaults → proceeds without custom stage questions
  - [ ] BG-12: Calls `list_active_projects` before creating project
  - [ ] BG-13: No existing project found → proceeds to create
  - [ ] BG-14: Calls `create_project(name="RetailEdge", stages=["Backlog","To Do","In Progress","Bug","Done"])`
  - [ ] BG-15: Creates 4 sprint tags via `create_tag`: "Sprint 1-2", "Sprint 3-4", "Sprint 5", "Sprint 6"
  - [ ] BG-16: Stores sprint label → tag_id map for later use
  - [ ] BG-17: Outputs: `Project "RetailEdge" created in Odoo.`

  ## Team Mapping

  - [ ] BG-18: Shows team roster table with roles and arch.md names from Sprint Mapping
  - [ ] BG-19: Table shows 5 roles: BE-senior, BE-junior, FE-senior, FE-junior, QA
  - [ ] BG-20: Consultant enters Odoo user IDs → stored as role → assignee_id map
  - [ ] BG-21: Roles skipped → assignee_ids: [] (blank assignee)
  - [ ] BG-22: Team mapping question asked exactly once

  ## Duplicate Check

  - [ ] BG-23: Calls `list_tickets(project_id=<id>, limit=100)` before preview
  - [ ] BG-24: No tickets found (new project) → proceeds silently, no warning shown

  ## Ticket Type Inference

  - [ ] BG-25: OracleConnector (Node.js) → backend → tag_id 44
  - [ ] BG-26: SalesAggregator (Node.js) → backend → tag_id 44
  - [ ] BG-27: SalesAPI + AlertEngine (Node.js) → backend → tag_id 44
  - [ ] BG-28: DashboardUI + AlertsUI (React) → frontend → tag_id 2
  - [ ] BG-29: Offline mode (Service Worker) → backend → tag_id 44

  ## Ticket Structure

  - [ ] BG-30: 5 parent tickets generated (one per Build Order item)
  - [ ] BG-31: Ticket 1 title: `[Sprint 1-2] OracleConnector`
  - [ ] BG-32: Ticket 1 deadline = Sprint 1-2 end date from Sprint Mapping
  - [ ] BG-33: Ticket 1 assignee = BE-senior Odoo ID
  - [ ] BG-34: Ticket 1 tag_ids = [44 (backend), <Sprint 1-2 tag_id>]
  - [ ] BG-35: Ticket 1 priority = "0" (OracleConnector is L effort, not XL)
  - [ ] BG-36: Ticket 1 description has sections: Scope, API Endpoints, Key Business Rules, Acceptance Criteria, Edge Cases, Open Questions
  - [ ] BG-37: Ticket 1 Open Questions contains OQ #1 (Oracle Retail POS integration method — blocks: OracleConnector)
  - [ ] BG-38: OracleConnector subtask created under ticket 1 (FIRST build of shared component)
  - [ ] BG-39: Ticket 3 (SalesAPI + AlertEngine) description contains text: `(reuses OracleConnector — built in Sprint 1-2)` — no OracleConnector subtask
  - [ ] BG-40: Ticket 4 title: `[Sprint 5] DashboardUI + AlertsUI`
  - [ ] BG-41: Ticket 4 tag_ids = [2 (frontend), <Sprint 5 tag_id>]
  - [ ] BG-42: Ticket 4 description has: Scope, Screens & Components, Key Business Rules, Acceptance Criteria, Edge Cases, Open Questions
  - [ ] BG-43: Ticket 5 (Offline mode) Open Questions contains OQ #2 (staleness window — blocks: Offline mode)

  ## STRAWMAN Tickets

  - [ ] BG-44: 5 STRAWMAN tickets generated (one per STRAWMAN Summary item)
  - [ ] BG-45: STRAWMAN title format: `⚠ STRAWMAN: Verify <decision> before Sprint 1`
  - [ ] BG-46: STRAWMAN stage = Backlog equivalent (NOT To Do)
  - [ ] BG-47: STRAWMAN priority = "1" (high)
  - [ ] BG-48: STRAWMAN description has 4 sections: Decision, Why Tentative, What Resolves This, Impact if Wrong
  - [ ] BG-49: STRAWMAN assignee = blank

  ## Preview Gate

  - [ ] BG-50: Preview table shown before any MCP create call
  - [ ] BG-51: 5 parent tickets shown in correct sprint order
  - [ ] BG-52: Subtasks shown indented with `↳` prefix under each parent
  - [ ] BG-53: OracleConnector subtask appears once only (under ticket 1)
  - [ ] BG-54: Ticket 3 shows `(reuses OracleConnector — Sprint 1-2)` note, not a subtask row
  - [ ] BG-55: STRAWMAN tickets shown in separate section after parent tickets
  - [ ] BG-56: Prompt ends with: `[A] Yes / [B] Edit / [C] Cancel`

  ## Bulk Creation

  - [ ] BG-57: On [A] approval: calls `bulk_create_tickets` for parent tickets (not individual `create_ticket`)
  - [ ] BG-58: For each parent: calls `add_subtasks` with that parent's subtask list
  - [ ] BG-59: Calls separate `bulk_create_tickets` for STRAWMAN tickets
  - [ ] BG-60: Progress shown inline: "Creating parent tickets... done (5 tickets)"
  - [ ] BG-61: Progress shown: "Adding subtasks... done (X subtasks)"
  - [ ] BG-62: Progress shown: "Creating STRAWMAN tickets... done (5 tickets)"

  ## Post-Creation Output

  - [ ] BG-63: Shows correct parent ticket count (5)
  - [ ] BG-64: Shows correct STRAWMAN count (5)
  - [ ] BG-65: Shows project name and Odoo ID
  - [ ] BG-66: Lists 3 next steps (review in Odoo, resolve STRAWMANs, ROADMAP)

  ## Session State

  - [ ] BG-67: `session_state.md` written to working directory
  - [ ] BG-68: Stage = "BACKLOG_GENERATOR", Status = "complete"
  - [ ] BG-69: Next Step = "ROADMAP (not yet built — V1.5)"
  - [ ] BG-70: Open Items lists all 5 STRAWMAN items
  - [ ] BG-71: Notes include Odoo project name, ID, and ticket counts
  - [ ] BG-72: Output: `session_state.md updated.`
  ```

- [ ] **Step 3: Commit fixture**

  ```bash
  git add tests/fixtures/backlog-generator/
  git commit -m "test(backlog-generator): synthetic-01 PASS-path fixture"
  ```

---

## Task 7: Create synthetic-02-duplicate fixture (DUPLICATE path)

**Files:**
- Create: `tests/fixtures/backlog-generator/synthetic-02-duplicate/arch.md`
- Create: `tests/fixtures/backlog-generator/synthetic-02-duplicate/expected_behaviors.md`

Scenario: Consultant runs BACKLOG_GENERATOR on RetailEdge again. "RetailEdge" project already
exists in Odoo. Two tickets already exist in it. Consultant chooses to use existing project,
sees duplicate warning, skips duplicates, creates only the remaining tickets.

- [ ] **Step 1: Copy arch.md into the fixture folder**

  Read `tests/fixtures/arch-proposer/synthetic-01/arch.md`.
  Write its content to `tests/fixtures/backlog-generator/synthetic-02-duplicate/arch.md`.

- [ ] **Step 2: Write expected_behaviors.md for synthetic-02-duplicate**

  Write `tests/fixtures/backlog-generator/synthetic-02-duplicate/expected_behaviors.md`:

  ```markdown
  # Expected Behaviors — BACKLOG_GENERATOR synthetic-02-duplicate
  # Fixture: RetailEdge DUPLICATE path — existing project, 2 tickets already exist

  ## Setup conditions (not tested, assumed as preconditions)
  - Odoo project "RetailEdge" already exists (from a prior BACKLOG_GENERATOR run)
  - 2 tickets already exist: "[Sprint 1-2] OracleConnector" and "[Sprint 1-2] SalesAggregator + data model"

  ## Project Setup

  - [ ] DUP-01: `list_active_projects` called before `create_project`
  - [ ] DUP-02: "RetailEdge" found in existing projects → warning shown
  - [ ] DUP-03: Warning message shows: `⚠ A project named "RetailEdge" already exists in Odoo.`
  - [ ] DUP-04: Warning offers 3 options: [A] Use existing / [B] Create new with different name / [C] Cancel
  - [ ] DUP-05: Consultant chooses [A] → existing project ID stored, `create_project` NOT called
  - [ ] DUP-06: Sprint tags created (or reused if they exist) for the existing project

  ## Duplicate Check

  - [ ] DUP-07: `list_tickets` called on existing project (not a new project)
  - [ ] DUP-08: 2 matching titles found: "[Sprint 1-2] OracleConnector" and "[Sprint 1-2] SalesAggregator + data model"
  - [ ] DUP-09: Duplicate warning shown with both matching ticket titles and their Odoo IDs
  - [ ] DUP-10: Consultant must explicitly respond before preview table is shown
  - [ ] DUP-11: Consultant chooses [A] Skip duplicates

  ## Preview Gate

  - [ ] DUP-12: Preview table shows all 5 parent tickets
  - [ ] DUP-13: Tickets 1 and 2 are marked `[EXISTS]` in the preview table
  - [ ] DUP-14: Tickets 3, 4, 5 shown normally (no [EXISTS] marker)
  - [ ] DUP-15: STRAWMAN tickets shown in separate section (no [EXISTS] markers)

  ## Bulk Creation

  - [ ] DUP-16: `bulk_create_tickets` called only for tickets 3, 4, 5 (not 1 and 2)
  - [ ] DUP-17: `add_subtasks` called only for tickets 3, 4, 5
  - [ ] DUP-18: `bulk_create_tickets` called for all 5 STRAWMAN tickets (not duplicates)
  - [ ] DUP-19: Progress shows correct reduced count: "Creating parent tickets... done (3 tickets)"

  ## Post-Creation Output

  - [ ] DUP-20: Shows 3 parent tickets (not 5) — reflects only newly created tickets
  - [ ] DUP-21: Shows 5 STRAWMAN tickets
  - [ ] DUP-22: Session state written with correct counts
  ```

- [ ] **Step 3: Commit fixture**

  ```bash
  git add tests/fixtures/backlog-generator/synthetic-02-duplicate/
  git commit -m "test(backlog-generator): synthetic-02-duplicate DUPLICATE-path fixture"
  ```

---

## Task 8: Create TEST_SCRIPT.md

**Files:**
- Create: `tests/fixtures/backlog-generator/TEST_SCRIPT.md`

- [ ] **Step 1: Write TEST_SCRIPT.md**

  ```markdown
  # BACKLOG_GENERATOR Test Script
  # Skill: skills/project-initiator/BACKLOG_GENERATOR.md
  # Version: V1.4

  ---

  ## Fixture Overview

  | Fixture | Path | Purpose |
  |---|---|---|
  | synthetic-01 | tests/fixtures/backlog-generator/synthetic-01/ | PASS path — new project, default stages, no duplicates |
  | synthetic-02-duplicate | tests/fixtures/backlog-generator/synthetic-02-duplicate/ | DUPLICATE path — existing project + 2 duplicate tickets |

  ---

  ## Running synthetic-01 (PASS path)

  1. Open Claude Code
  2. Set working directory to: `tests/fixtures/backlog-generator/synthetic-01/`
  3. Say: `run BACKLOG_GENERATOR`
  4. When asked for engagement name: type **RetailEdge**
  5. Project setup: accept defaults (type `yes`)
  6. Team mapping: enter Odoo user IDs for each role (or `skip`)
  7. Duplicate check: should show no duplicates (new project) — proceeds silently
  8. Preview: review the ticket hierarchy table, type `yes` to approve
  9. Bulk creation: confirm progress messages shown
  10. Score against: `tests/fixtures/backlog-generator/synthetic-01/expected_behaviors.md`
  11. Clean up: no local files to delete (Odoo tickets created in Odoo, not locally)

  ---

  ## Running synthetic-02-duplicate (DUPLICATE path)

  1. Open Claude Code
  2. Set working directory to: `tests/fixtures/backlog-generator/synthetic-02-duplicate/`
  3. Say: `run BACKLOG_GENERATOR`
  4. When asked for engagement name: type **RetailEdge**
  5. Project setup: accept defaults
  6. Existing project check: **RetailEdge** already exists → type `A` to use existing
  7. Team mapping: enter user IDs
  8. Duplicate check: should show 2 duplicate tickets → type `A` to skip duplicates
  9. Preview: should show tickets 1 and 2 marked [EXISTS]
  10. Approve and confirm only 3 parent tickets are created
  11. Score against: `tests/fixtures/backlog-generator/synthetic-02-duplicate/expected_behaviors.md`

  ---

  ## Scoring

  Record results in a `test_report.md` file in each fixture folder.
  Use the format from `tests/fixtures/arch-proposer/synthetic-01/test_report.md`.

  Pass threshold: all checkboxes in expected_behaviors.md marked [x].
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add tests/fixtures/backlog-generator/TEST_SCRIPT.md
  git commit -m "test(backlog-generator): TEST_SCRIPT.md"
  ```

---

## Task 9: QA simulation — synthetic-01 (PASS path)

**Files:**
- Create: `tests/fixtures/backlog-generator/synthetic-01/test_report.md`

Simulate what Claude would do when running BACKLOG_GENERATOR against the RetailEdge arch.md.
For each behavior in expected_behaviors.md, determine: does the skill file produce this behavior?
Mark [x] for pass, write explanation for any fail.

- [ ] **Step 1: Read the skill file and fixture**

  Read `skills/project-initiator/BACKLOG_GENERATOR.md`.
  Read `tests/fixtures/backlog-generator/synthetic-01/expected_behaviors.md`.
  Read `tests/fixtures/backlog-generator/synthetic-01/arch.md`.

- [ ] **Step 2: Simulate skill execution mentally, step by step**

  Walk through every section of BACKLOG_GENERATOR.md as if executing it with synthetic-01/arch.md as input. For each behavior BG-01 through BG-72, determine pass or fail based on what the skill file instructs.

  Key checks:
  - BG-01/02: Does Pre-flight check for generic folder name? Does it ask for engagement name?
  - BG-12/13: Does Project Setup call `list_active_projects` before `create_project`?
  - BG-15: Does Project Setup create 4 sprint tags matching the 4 unique sprint labels in arch.md?
  - BG-38/39: Shared component rule — OracleConnector subtask only in ticket 1, text note in ticket 3?
  - BG-46: STRAWMAN tickets go to Backlog (not To Do)?
  - BG-57: Bulk creation uses `bulk_create_tickets`, not individual `create_ticket`?
  - BG-59: STRAWMAN tickets use separate `bulk_create_tickets` call?

- [ ] **Step 3: Write test_report.md**

  Write `tests/fixtures/backlog-generator/synthetic-01/test_report.md` with format:

  ```markdown
  # Test Report — BACKLOG_GENERATOR synthetic-01
  # Date: 2026-06-04
  # Fixture: RetailEdge PASS path
  # Result: <N>/<N> PASS

  ## Score by Section
  | Section | Checks | Pass | Notes |
  |---|---|---|---|
  | Pre-flight | 8 | ? | |
  | Project Setup | 9 | ? | |
  | Team Mapping | 5 | ? | |
  | Duplicate Check | 2 | ? | |
  | Ticket Type Inference | 5 | ? | |
  | Ticket Structure | 14 | ? | |
  | STRAWMAN Tickets | 6 | ? | |
  | Preview Gate | 7 | ? | |
  | Bulk Creation | 6 | ? | |
  | Post-Creation Output | 4 | ? | |
  | Session State | 6 | ? | |
  | **Total** | **72** | **?** | |

  ## Failures (if any)
  (list any failed behaviors with explanation)

  ## Pass/Fail per behavior
  (copy expected_behaviors.md with [x] or [ ] filled in)
  ```

  Fill in actual scores based on the simulation in Step 2.

- [ ] **Step 4: Commit**

  ```bash
  git add tests/fixtures/backlog-generator/synthetic-01/test_report.md
  git commit -m "test(backlog-generator): synthetic-01 PASS-path QA simulation"
  ```

---

## Task 10: QA simulation — synthetic-02-duplicate (DUPLICATE path)

**Files:**
- Create: `tests/fixtures/backlog-generator/synthetic-02-duplicate/test_report.md`

- [ ] **Step 1: Read fixture and skill file**

  Read `tests/fixtures/backlog-generator/synthetic-02-duplicate/expected_behaviors.md`.
  Read `skills/project-initiator/BACKLOG_GENERATOR.md` (already read in Task 9, but re-read if needed).

- [ ] **Step 2: Simulate DUPLICATE path execution**

  Walk through BACKLOG_GENERATOR.md with the precondition that "RetailEdge" project exists
  and 2 tickets already exist. Check behaviors DUP-01 through DUP-22.

  Key checks:
  - DUP-02/03: Does the skill surface the warning when `list_active_projects` finds "RetailEdge"?
  - DUP-05: Does the skill skip `create_project` when consultant chooses [A]?
  - DUP-08/09: Does the Duplicate Check show both matching ticket titles with IDs?
  - DUP-13: Does the preview table mark tickets 1 and 2 as `[EXISTS]`?
  - DUP-16/17: Does bulk creation exclude the 2 duplicates?
  - DUP-20: Does post-creation output show 3 (not 5) parent tickets?

- [ ] **Step 3: Write test_report.md**

  Write `tests/fixtures/backlog-generator/synthetic-02-duplicate/test_report.md` with same format as Task 9. Score DUP-01 through DUP-22.

- [ ] **Step 4: Commit**

  ```bash
  git add tests/fixtures/backlog-generator/synthetic-02-duplicate/test_report.md
  git commit -m "test(backlog-generator): synthetic-02-duplicate DUPLICATE-path QA simulation"
  ```

---

## Task 11: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Read current CLAUDE.md**

  Read `CLAUDE.md` to see exact current content before editing.

- [ ] **Step 2: Update Phase Status table**

  Find the row: `| PI V1.4 | BACKLOG_GENERATOR | Not started |`
  Change to: `| PI V1.4 | BACKLOG_GENERATOR skill — built, tested on 2 synthetic fixtures (<N>/<N> + <M>/<M>) | Done |`

  (Fill in actual scores from test_reports.)

- [ ] **Step 3: Update chain status line**

  Find: `DOC_GENERATOR (done) → BACKLOG_GENERATOR (V1.4, not started)`
  Change to: `DOC_GENERATOR (done) → BACKLOG_GENERATOR (done)`

- [ ] **Step 4: Add BACKLOG_GENERATOR section after DOC_GENERATOR section**

  After the DOC_GENERATOR section and before the START section, add:

  ```markdown
  ### BACKLOG_GENERATOR (V1.4 — done)

  Skill location: `skills/project-initiator/BACKLOG_GENERATOR.md`

  **How to run:** Open Claude Code in the `<client-name>/` folder (with `arch.md` present).
  Say `run BACKLOG_GENERATOR`. Creates Odoo project + stages, maps team roles, generates
  structured ticket hierarchy, bulk-creates via MCP.

  **Project Setup:** Asks for project name (defaults to folder name) and stages (defaults to
  Backlog/To Do/In Progress/Bug/Done). Custom stages trigger purpose-mapping questions.

  **Ticket format:** Parent ticket per Build Order item, subtasks per component. Backend tickets
  get API endpoint tables; frontend tickets get screen/component lists. Both include business
  rules, acceptance criteria, edge cases, and relevant Open Questions.

  **Duplicate check:** Always runs `list_tickets` before creation. Warns on matches, asks approval.

  **Sprint tags:** Creates one Odoo tag per sprint label; every ticket tagged with its sprint.

  Design spec: `docs/superpowers/specs/2026-06-04-backlog-generator-design.md`

  Test fixtures: `tests/fixtures/backlog-generator/synthetic-01/` (<N>/<N>),
  `tests/fixtures/backlog-generator/synthetic-02-duplicate/` (<M>/<M>)
  ```

- [ ] **Step 5: Commit CLAUDE.md**

  ```bash
  git add CLAUDE.md
  git commit -m "docs(claude-md): BACKLOG_GENERATOR V1.4 complete"
  ```

---

## Self-Review Checklist

Before declaring implementation complete, verify:

1. **Spec coverage:** Every section of `docs/superpowers/specs/2026-06-04-backlog-generator-design.md` has a corresponding section in `skills/project-initiator/BACKLOG_GENERATOR.md`.
2. **No placeholders:** Skill file contains no "TBD", "TODO", or vague instructions.
3. **Fixture completeness:** Both `expected_behaviors.md` files cover the full spec behavior for their path.
4. **QA scores:** Both `test_report.md` files show passing scores (all checkboxes [x]).
5. **CLAUDE.md updated:** Phase Status, chain status, and new BACKLOG_GENERATOR section all present.
