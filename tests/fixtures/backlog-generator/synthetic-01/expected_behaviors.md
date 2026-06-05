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

- [ ] BG-18: Shows team roster table built dynamically from Sprint Mapping
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
- [ ] BG-29: Offline mode (Service Worker) → backend → tag_id 44; preview shows `[type inferred — verify]` flag

## Ticket Structure

- [ ] BG-30: 5 parent tickets generated (one per Build Order item)
- [ ] BG-31: Ticket 1 title: `[Sprint 1-2] OracleConnector`
- [ ] BG-32: Ticket 1 deadline = 2026-07-02 (Sprint 1-2 end: start 2026-06-04 + 2 × 2-week sprints)
- [ ] BG-33: Ticket 1 assignee = BE-senior Odoo ID
- [ ] BG-34: Ticket 1 tag_ids = [44 (backend), <Sprint 1-2 tag_id>]
- [ ] BG-35: Ticket 1 priority = "0" (OracleConnector is L effort, not XL)
- [ ] BG-36: Ticket 1 description has sections: Scope, Interfaces (connector — no HTTP routes), Key Business Rules, Acceptance Criteria, Edge Cases, Open Questions
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
