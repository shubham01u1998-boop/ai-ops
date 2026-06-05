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
- [ ] DUP-15: STRAWMAN tickets shown in separate section (no [EXISTS] markers — no existing STRAWMAN tickets)

## Bulk Creation

- [ ] DUP-16: `bulk_create_tickets` called only for tickets 3, 4, 5 (not 1 and 2)
- [ ] DUP-17: `add_subtasks` called only for tickets 3, 4, 5
- [ ] DUP-18: `bulk_create_tickets` called for all 5 STRAWMAN tickets (none were duplicates)
- [ ] DUP-19: Progress shows correct reduced count: "Creating parent tickets... done (3 tickets)"

## Post-Creation Output

- [ ] DUP-20: Shows 3 parent tickets (not 5) — reflects only newly created tickets
- [ ] DUP-21: Shows 5 STRAWMAN tickets
- [ ] DUP-22: Session state written with correct counts (3 parent + subtasks + 5 STRAWMAN)
