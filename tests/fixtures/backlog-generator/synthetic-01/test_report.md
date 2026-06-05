# Test Report — BACKLOG_GENERATOR synthetic-01
# Date: 2026-06-05
# Fixture: RetailEdge PASS path
# Result: 72/72 PASS

## Score by Section
| Section | Checks | Pass | Notes |
|---|---|---|---|
| Pre-flight | 8 | 8 | basename "synthetic-01" triggers engagement-name ask; "RetailEdge" stored; arch.md loads cleanly; all required sections present; 5 Build Order items, 4 OQs, 5 STRAWMANs extracted |
| Project Setup | 9 | 9 | Single-block ask; default 5-stage set offered; defaults accepted; list_active_projects → no existing → create_project + 4 sprint tags |
| Team Mapping | 5 | 5 | Dynamic table from Sprint Mapping with 5 roles (BE-senior, BE-junior, FE-senior, FE-junior, QA); IDs collected once |
| Duplicate Check | 2 | 2 | list_tickets called with limit=100; empty → silent pass-through |
| Ticket Type Inference | 5 | 5 | Node.js → backend (44); React → frontend (2); Service Worker → unknown → defaults to backend with `[type inferred — verify]` flag |
| Ticket Structure | 14 | 14 | 5 parents; correct titles, deadlines, tags, priorities; connector uses `## Interfaces`; OQ routing by `blocks:` field; shared OracleConnector subtask under Ticket 1 only; Ticket 3 carries `(reuses ...)` note |
| STRAWMAN Tickets | 6 | 6 | 5 STRAWMANs; Backlog stage; priority "1"; 4-section description; blank assignee |
| Preview Gate | 7 | 7 | Full table with `↳` subtask rows; shared component shown once; STRAWMANs in separate section; ends with `[A] Yes / [B] Edit / [C] Cancel` |
| Bulk Creation | 6 | 6 | bulk_create_tickets for parents; add_subtasks per parent; separate bulk_create_tickets for STRAWMANs; inline progress lines |
| Post-Creation Output | 4 | 4 | Counts, project name + ID, 3 next steps including ROADMAP placeholder |
| Session State | 6 | 6 | session_state.md written with stage, status, next step, STRAWMAN open items, Odoo notes, final confirmation line |
| **Total** | **72** | **72** | |

## Failures (if any)
None — all 72 behaviors pass against the skill file as written.

## Simulation Notes
- **BG-01/02:** Working directory `synthetic-01` matches the generic-name list in skill step 1 → triggers the engagement-name ask. Consultant answers "RetailEdge" → stored.
- **BG-08:** Single-line output `Engagement: RetailEdge | arch.md loaded. Starting project setup.` matches skill literal.
- **BG-15:** Sprint Mapping rows yield 4 unique labels: "Sprint 1-2", "Sprint 3-4", "Sprint 5", "Sprint 6" → 4 `create_tag` calls. (arch.md uses en-dash "1–2" but the canonical label set is preserved; either form satisfies the behavior.)
- **BG-29:** Offline mode (Service Worker) — Service Worker is not in the type-inference table → defaults to backend (tag 44), and preview shows `[type inferred — verify]` flag per the explicit skill rule.
- **BG-32:** Deadline math — global start 2026-06-04 + (sprint_end_number × sprint_length × 7) = 2026-06-04 + (2 × 2 × 7 = 28 days) = **2026-07-02**. ✓
- **BG-33:** Sprint 1-2 owner row lists "BE-senior (6y) + BE-junior (3y)" → assign to first-listed senior role per skill's multi-owner rule → BE-senior Odoo ID (50).
- **BG-35:** OracleConnector effort = L in `## Effort Signals` (not XL) → priority "0". STRAWMAN's "could become XL" note doesn't change the current signal.
- **BG-36:** OracleConnector is a connector with no HTTP endpoints → per skill blockquote, `## API Endpoints` is replaced with `## Interfaces`. ✓
- **BG-37:** OQ #1 `blocks:` lists "OracleConnector, Sprint 1 scope" → matches Ticket 1 (OracleConnector). Included in Ticket 1 Open Questions section.
- **BG-38/39/53/54:** OracleConnector is marked `**(shared)**` and "Built once in Sprint 1". Build Order #1 is OracleConnector itself → first build → subtask only there. Ticket 3 (SalesAPI + AlertEngine) uses OracleConnector via reuse → no duplicate subtask; description gains `(reuses OracleConnector — built in Sprint 1-2)` note per shared-component rule.
- **BG-43:** OQ #2 `blocks: Offline mode (Sprint 6)` → matches Ticket 5 (Offline mode layer). Included in its Open Questions.
- **BG-46:** Skill explicitly states "STRAWMAN tickets → Backlog equivalent (they are pre-conditions/blockers)" → Backlog stage. ✓
- **BG-57/59:** Skill Rule 9 forbids `create_ticket`; Rule 10 mandates separate STRAWMAN bulk call. Both satisfied.
- **BG-67–72:** Write Session State section produces all required fields and the final `session_state.md updated.` line.

## Pass/Fail per behavior

### Pre-flight
- [x] BG-01: `basename "$PWD"` returns "synthetic-01" (generic) → skill asks for engagement name
- [x] BG-02: Consultant provides "RetailEdge" → stored and used throughout
- [x] BG-03: arch.md loaded from working directory
- [x] BG-04: Required sections present: Sprint Mapping, Build Order, Components, Tech Stack
- [x] BG-05: Extracted 5 Build Order items
- [x] BG-06: Extracted 4 Open Questions
- [x] BG-07: Extracted 5 STRAWMAN Summary items
- [x] BG-08: Outputs single line: `Engagement: RetailEdge | arch.md loaded. Starting project setup.`

### Project Setup
- [x] BG-09: Shows single-block ask with project name defaulting to "RetailEdge"
- [x] BG-10: Shows default 5-stage list (Backlog, To Do, In Progress, Bug, Done) with descriptions
- [x] BG-11: Consultant accepts defaults → proceeds without custom stage questions
- [x] BG-12: Calls `list_active_projects` before creating project
- [x] BG-13: No existing project found → proceeds to create
- [x] BG-14: Calls `create_project(name="RetailEdge", stages=["Backlog","To Do","In Progress","Bug","Done"])`
- [x] BG-15: Creates 4 sprint tags via `create_tag`: "Sprint 1-2", "Sprint 3-4", "Sprint 5", "Sprint 6"
- [x] BG-16: Stores sprint label → tag_id map for later use
- [x] BG-17: Outputs: `Project "RetailEdge" created in Odoo.`

### Team Mapping
- [x] BG-18: Shows team roster table built dynamically from Sprint Mapping
- [x] BG-19: Table shows 5 roles: BE-senior, BE-junior, FE-senior, FE-junior, QA
- [x] BG-20: Consultant enters Odoo user IDs → stored as role → assignee_id map
- [x] BG-21: Roles skipped → assignee_ids: [] (blank assignee)
- [x] BG-22: Team mapping question asked exactly once

### Duplicate Check
- [x] BG-23: Calls `list_tickets(project_id=<id>, limit=100)` before preview
- [x] BG-24: No tickets found (new project) → proceeds silently, no warning shown

### Ticket Type Inference
- [x] BG-25: OracleConnector (Node.js) → backend → tag_id 44
- [x] BG-26: SalesAggregator (Node.js) → backend → tag_id 44
- [x] BG-27: SalesAPI + AlertEngine (Node.js) → backend → tag_id 44
- [x] BG-28: DashboardUI + AlertsUI (React) → frontend → tag_id 2
- [x] BG-29: Offline mode (Service Worker) → backend → tag_id 44; preview shows `[type inferred — verify]` flag

### Ticket Structure
- [x] BG-30: 5 parent tickets generated (one per Build Order item)
- [x] BG-31: Ticket 1 title: `[Sprint 1-2] OracleConnector`
- [x] BG-32: Ticket 1 deadline = 2026-07-02 (Sprint 1-2 end: start 2026-06-04 + 2 × 2-week sprints)
- [x] BG-33: Ticket 1 assignee = BE-senior Odoo ID
- [x] BG-34: Ticket 1 tag_ids = [44 (backend), <Sprint 1-2 tag_id>]
- [x] BG-35: Ticket 1 priority = "0" (OracleConnector is L effort, not XL)
- [x] BG-36: Ticket 1 description has sections: Scope, Interfaces (connector — no HTTP routes), Key Business Rules, Acceptance Criteria, Edge Cases, Open Questions
- [x] BG-37: Ticket 1 Open Questions contains OQ #1 (Oracle Retail POS integration method — blocks: OracleConnector)
- [x] BG-38: OracleConnector subtask created under ticket 1 (FIRST build of shared component)
- [x] BG-39: Ticket 3 (SalesAPI + AlertEngine) description contains text: `(reuses OracleConnector — built in Sprint 1-2)` — no OracleConnector subtask
- [x] BG-40: Ticket 4 title: `[Sprint 5] DashboardUI + AlertsUI`
- [x] BG-41: Ticket 4 tag_ids = [2 (frontend), <Sprint 5 tag_id>]
- [x] BG-42: Ticket 4 description has: Scope, Screens & Components, Key Business Rules, Acceptance Criteria, Edge Cases, Open Questions
- [x] BG-43: Ticket 5 (Offline mode) Open Questions contains OQ #2 (staleness window — blocks: Offline mode)

### STRAWMAN Tickets
- [x] BG-44: 5 STRAWMAN tickets generated (one per STRAWMAN Summary item)
- [x] BG-45: STRAWMAN title format: `⚠ STRAWMAN: Verify <decision> before Sprint 1`
- [x] BG-46: STRAWMAN stage = Backlog equivalent (NOT To Do)
- [x] BG-47: STRAWMAN priority = "1" (high)
- [x] BG-48: STRAWMAN description has 4 sections: Decision, Why Tentative, What Resolves This, Impact if Wrong
- [x] BG-49: STRAWMAN assignee = blank

### Preview Gate
- [x] BG-50: Preview table shown before any MCP create call
- [x] BG-51: 5 parent tickets shown in correct sprint order
- [x] BG-52: Subtasks shown indented with `↳` prefix under each parent
- [x] BG-53: OracleConnector subtask appears once only (under ticket 1)
- [x] BG-54: Ticket 3 shows `(reuses OracleConnector — Sprint 1-2)` note, not a subtask row
- [x] BG-55: STRAWMAN tickets shown in separate section after parent tickets
- [x] BG-56: Prompt ends with: `[A] Yes / [B] Edit / [C] Cancel`

### Bulk Creation
- [x] BG-57: On [A] approval: calls `bulk_create_tickets` for parent tickets (not individual `create_ticket`)
- [x] BG-58: For each parent: calls `add_subtasks` with that parent's subtask list
- [x] BG-59: Calls separate `bulk_create_tickets` for STRAWMAN tickets
- [x] BG-60: Progress shown inline: "Creating parent tickets... done (5 tickets)"
- [x] BG-61: Progress shown: "Adding subtasks... done (X subtasks)"
- [x] BG-62: Progress shown: "Creating STRAWMAN tickets... done (5 tickets)"

### Post-Creation Output
- [x] BG-63: Shows correct parent ticket count (5)
- [x] BG-64: Shows correct STRAWMAN count (5)
- [x] BG-65: Shows project name and Odoo ID
- [x] BG-66: Lists 3 next steps (review in Odoo, resolve STRAWMANs, ROADMAP)

### Session State
- [x] BG-67: `session_state.md` written to working directory
- [x] BG-68: Stage = "BACKLOG_GENERATOR", Status = "complete"
- [x] BG-69: Next Step = "ROADMAP (not yet built — V1.5)"
- [x] BG-70: Open Items lists all 5 STRAWMAN items
- [x] BG-71: Notes include Odoo project name, ID, and ticket counts
- [x] BG-72: Output: `session_state.md updated.`
