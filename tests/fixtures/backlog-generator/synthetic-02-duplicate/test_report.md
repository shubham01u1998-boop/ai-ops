# Test Report — BACKLOG_GENERATOR synthetic-02-duplicate
# Date: 2026-06-05
# Fixture: RetailEdge DUPLICATE path
# Result: 22/22 PASS

## Score by Section
| Section | Checks | Pass | Notes |
|---|---|---|---|
| Project Setup | 6 | 6 | DUP-01 through DUP-06 — existing-project branch correctly handled per skill lines 102-117 |
| Duplicate Check | 5 | 5 | DUP-07 through DUP-11 — `list_tickets` runs against existing project ID 999; 2 case-insensitive exact matches found |
| Preview Gate | 4 | 4 | DUP-12 through DUP-15 — `[EXISTS]` markers correctly applied to tickets 1+2; STRAWMANs unmarked |
| Bulk Creation | 4 | 4 | DUP-16 through DUP-19 — excluded tickets honored for both `bulk_create_tickets` and `add_subtasks`; STRAWMAN bulk full (5) |
| Post-Creation Output | 3 | 3 | DUP-20 through DUP-22 — counts reflect only newly created tickets; session_state.md captures 3 parent + subtasks + 5 STRAWMAN |
| **Total** | **22** | **22** | All behaviors pass under simulated DUPLICATE-path conditions |

## Failures (if any)
None.

## Pass/Fail per behavior

### Setup conditions (not tested, assumed as preconditions)
- Odoo project "RetailEdge" already exists (from a prior BACKLOG_GENERATOR run)
- 2 tickets already exist: "[Sprint 1-2] OracleConnector" and "[Sprint 1-2] SalesAggregator + data model"

### Project Setup
- [x] DUP-01: `list_active_projects` called before `create_project` — skill explicitly mandates this at line 102 ("Before creating the project: Call `list_active_projects` to check if a project with this name already exists")
- [x] DUP-02: "RetailEdge" found in existing projects → warning shown — `list_active_projects` returns RetailEdge (ID=999), triggering the "If found" branch at line 105
- [x] DUP-03: Warning message shows: `⚠ A project named "RetailEdge" already exists in Odoo.` — verbatim per skill line 107
- [x] DUP-04: Warning offers 3 options: [A] Use existing / [B] Create new with different name / [C] Cancel — verbatim per skill lines 108-110
- [x] DUP-05: Consultant chooses [A] → existing project ID stored, `create_project` NOT called — skill line 112: "[A]: store the existing project ID; run Duplicate Check against it; skip `create_project`"
- [x] DUP-06: Sprint tags created (or reused if they exist) for the existing project — skill lines 119-122 instruct `create_tag` per unique sprint label ("Sprint 1-2", "Sprint 3-4", "Sprint 5", "Sprint 6") after project setup, with no conditional skip for [A] branch

### Duplicate Check
- [x] DUP-07: `list_tickets` called on existing project (not a new project) — skill line 155: `list_tickets(project_id=<project_id>, limit=100)` with stored project_id=999
- [x] DUP-08: 2 matching titles found: "[Sprint 1-2] OracleConnector" and "[Sprint 1-2] SalesAggregator + data model" — generated parent titles match existing titles exactly (case-insensitive), per skill line 156
- [x] DUP-09: Duplicate warning shown with both matching ticket titles and their Odoo IDs — skill lines 159-167 specify the warning format with `matches existing ticket #<id>` for #101 and #102
- [x] DUP-10: Consultant must explicitly respond before preview table is shown — skill line 168: "Consultant must explicitly respond before the preview table is shown"
- [x] DUP-11: Consultant chooses [A] Skip duplicates — handled per skill line 170; both duplicate titles flagged as [EXISTS] for the preview

### Preview Gate
- [x] DUP-12: Preview table shows all 5 parent tickets — preview includes all parent tickets from Build Order (5 items: OracleConnector, SalesAggregator, SalesAPI+AlertEngine, DashboardUI+AlertsUI, Offline mode layer)
- [x] DUP-13: Tickets 1 and 2 are marked `[EXISTS]` in the preview table — skill line 170: "When [A] is chosen, duplicate-matched titles are marked `[EXISTS]`"
- [x] DUP-14: Tickets 3, 4, 5 shown normally (no [EXISTS] marker) — only the 2 duplicate-matched titles receive the marker; others are unmarked
- [x] DUP-15: STRAWMAN tickets shown in separate section (no [EXISTS] markers — no existing STRAWMAN tickets) — `list_tickets` returned no STRAWMAN matches, so all 5 STRAWMAN tickets render normally per preview format at lines 362-365

### Bulk Creation
- [x] DUP-16: `bulk_create_tickets` called only for tickets 3, 4, 5 (not 1 and 2) — skill lines 384-385: "If any tickets were marked [EXISTS] during Duplicate Check, exclude them from all `bulk_create_tickets` and `add_subtasks` calls"
- [x] DUP-17: `add_subtasks` called only for tickets 3, 4, 5 — same exclusion rule applies to `add_subtasks` per line 385
- [x] DUP-18: `bulk_create_tickets` called for all 5 STRAWMAN tickets (none were duplicates) — no STRAWMAN duplicates detected; STRAWMAN bulk call runs for full set per skill lines 413-430
- [x] DUP-19: Progress shows correct reduced count: "Creating parent tickets... done (3 tickets)" — progress reflects only newly created parents per skill line 434 format

### Post-Creation Output
- [x] DUP-20: Shows 3 parent tickets (not 5) — reflects only newly created tickets per skill line 447 ("<N> parent tickets") with N=3
- [x] DUP-21: Shows 5 STRAWMAN tickets — Q=5 in skill line 449 ("<Q> STRAWMAN pre-condition tickets")
- [x] DUP-22: Session state written with correct counts (3 parent + subtasks + 5 STRAWMAN) — skill lines 463-494 mandate writing session_state.md with "Tickets created: <N> parent + <P> subtasks + <Q> STRAWMAN" with N=3, Q=5
