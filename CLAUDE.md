# ai-ops — Claude Code Instructions

## What this repo is
Development home for the **Odoo AI Ticket Pipeline** for TiffinConnect.
Skill files are written here, reviewed, then manually uploaded to **Claude Enterprise** (Project: Ticket Intake — TiffinConnect).
The MCP server (`odoo-mcp/`) connects Claude Enterprise to Odoo 19 via Python.

---

## Repo Structure

```
ai-ops/
  skills/
    core/                  # LAYER_0_GLOBAL.md, BUG_REPORT_TEMPLATE.md
    layers/                # LAYER_2_FASTPATH.md, QA_INTAKE.md
    project-initiator/     # DISCOVERY.md (V1.0), MVP_SYNTHESIZER.md (V1.1)
  context/                 # TEAM_CONTEXT.md, PARKING_LOT.md
  docs/
    superpowers/
      specs/               # Design specs (2026-06-03-project-initiator-design.md, etc.)
  tests/
    fixtures/
      discovery/           # Synthetic engagement fixtures for DISCOVERY testing
      mvp-synthesizer/     # Synthetic engagement fixtures for MVP_SYNTHESIZER testing
  templates/               # future: generic project template
  ci/                      # future: GitLab CI integration
  odoo-mcp/                # Python MCP server (separate repo context)
```

---

## Phase Status

| Phase | Description | Status |
|---|---|---|
| 0 | Repo setup, folder structure | Done |
| 1 | 5 skill files built + deployed to Claude Enterprise | Done |
| 2 | 9 new skill files (classifier, mapper, PRD bridge, etc.) | Not started |
| 3 | TRIAGE_AGENT, Teams bot groundwork | Not started |
| 4 | GitLab CI auto-ticket from test failures | Not started |
| 5 | CLAUDE.md for IDE repos (ai-ops, frontend, backend) | In progress |
| 6 | Generic project template extraction | Not started |
| 7 | Teams bot, Power Automate, Azure Function | Not started |
| PI V1.0 | DISCOVERY skill — built, tested on 2 synthetic fixtures (64/64) | Done |
| PI V1.1 | MVP_SYNTHESIZER skill — readiness gate + mvp-scope.md | Done |
| PI V1.2 | START skill + SESSION_STATE integration | Done |
| PI V1.3 | ARCH_PROPOSER skill — built, tested on 2 synthetic fixtures (49/49 + 17/17) | Done |
| PI V1.3.5 | DOC_GENERATOR skill — built, tested on 2 synthetic fixtures (56/56 + 22/22) | Done |
| PI V1.4 | BACKLOG_GENERATOR skill — built, tested on 2 synthetic fixtures (72/72 + 22/22) | Done |
| PI V1.4.1 | BACKLOG_GENERATOR patch — write backlog.md first, 3-option Next Steps Gate | Done |
| PI V1.5 | ESTIMATOR skill — built, tested on 2 synthetic fixtures | Not started |

---

## Phase 1 Files (production, do not break)

| File | Version | Location |
|---|---|---|
| LAYER_0_GLOBAL.md | 1.6 | skills/core/ |
| LAYER_2_FASTPATH.md | 2.4 | skills/layers/ |
| QA_INTAKE.md | 1.5 | skills/layers/ |
| BUG_REPORT_TEMPLATE.md | — | skills/core/ |
| TEAM_CONTEXT.md | 1.3 | context/ |

Changes to any Phase 1 file require: edit here → review → push → re-upload to Claude Enterprise.

---

## Phase 2 Plan (locked — do not change without discussion)

Steps in order — complete one before starting the next:

1. `LAYER_1_CLASSIFIER.md` — smart routing brain, replaces temporary Routing Rule in LAYER_0_GLOBAL
2. `LAYER_2_PREPROCESSOR.md` — structured interview for vague requirements
3. `REQUIREMENT_BRIDGE.md` — PRD→TRD conversion, PM session then Developer session
4. `LAYER_3_TICKET_MAPPER.md` — replaces S9 inline mapper, full field mapping
5. `PRD_BREAKDOWN.md` — decompose PRD sections into discrete tickets with dependency map
6. `PARKING_LOT.md skill instruction` — context/, 4 sections, Claude reads/writes
7. `SESSION_STATE.md + resume logic` — context/, session tracking for long runs
8. `TEAM_CONTEXT self-update` — Rule 8 in LAYER_0_GLOBAL, end-of-session proposals

Note: LAYER_2_DISCOVERY.md retired — Project Initiator's DISCOVERY skill covers BA-style extraction.

After all 9: create Requirements and Triage Claude Enterprise Projects.

---

## Odoo Connection

- Instance: `https://fiftyfive-technologies-pvt-ltd.odoo.com`
- Project: TiffinConnect (ID: 58)
- Stages: Backlog(347) → To Do(348) → In Progress(349) → Bug(350) → Done(351)
- Tags: frontend(2), backend(44), bug(4)
- Team: Sahil(50), Vijay(62), Kunal(41), Tanu(57), Shubham(42)

---

## Locked Design Decisions

- Semi-automated throughout — AI drafts, human approves
- `CONFIRM ALL` for bulk tickets — never per-ticket confirmation
- Auto-routing via classifier — human never picks a skill manually
- No auto-assign at ticket creation — triage owner assigns
- Duplicate check: 1 `list_tickets` call by tag (cached per session); in-memory title matching — NOT 3 `search_tickets` calls by title
- Token estimate shown before any action > 500 tokens
- Permission required before any write action
- `TEAM_CONTEXT.md` max 200 lines, Section 5 auto-updated with human approval
- Ticket intake runs in both Claude Code and Claude Enterprise — same skill files, same flow

---

## Working Rules

- One step at a time
- Human writes files in IDE, pastes back here for review
- Review rounds until approved, then push and re-upload to Enterprise
- Test before advancing phases
- Never edit Phase 1 files without explicit instruction

---

## Ticket Intake

The full intake pipeline runs here in Claude Code identically to Claude Enterprise.

### Session-Start Protocol
At the start of every session, silently and in this order:
1. Read `context/PARKING_LOT.md` + `context/PARKING_LOT_SPEC.md` → run the session-start checklist (steps 1–6 per PARKING_LOT_SPEC.md). Surface warnings if any; silent if PARKING_LOT is empty.
2. Read `skills/core/LAYER_0_GLOBAL.md` → apply all rules for the session.
3. Read `context/TEAM_CONTEXT.md` → apply routing, tags, priority rules.

Then ask once: **"Who is this session? Sahil / Vijay / Kunal / Tanu / Shubham"**
Answer populates `by:` in all REQ{} blocks. If intake input arrives before the role is answered, ask the role question first, then process.

### Auto-Detect Mode
Read the input and route automatically — no command needed.

**Intake mode** (bug, feature, improvement, QA batch, requirement):
- Bug description, error message, "failing", "broken", "not working"
- Feature or improvement request
- QA batch / numbered findings / TC-NNN references
- PRD section or requirement description
- When ambiguous, default to intake mode

**Debug mode** (system inspection or investigation):
- Questions about existing tickets ("what tickets are open for…")
- MCP inspection ("list tickets", "get ticket #…", "search for…")
- System questions ("why did S8 flag…", "what's in PARKING_LOT…")

### Intake Mode (Production Flow)
1. Apply LAYER_0 Routing Rule: QA batch/TC-NNN → read `skills/layers/QA_INTAKE.md`; all other → read `skills/layers/LAYER_2_FASTPATH.md`
2. Follow the active skill file exactly — same rules, same output, same commands as Claude Enterprise
3. All MCP tools execute normally (create_ticket, list_tickets, search_tickets, etc.)
4. Production output: REQ{} blocks hidden, MCP calls not shown, errors at 2-line summary, similarity scores hidden

### Debug Mode
Same skill-driven flow, full visibility:
- REQ{} blocks shown after extraction
- MCP calls logged inline before results: `→ list_tickets(project_id=58, tag="bug", limit=50)`
- Raw Odoo errors shown in full
- Similarity scores shown: `"OTP login failing" — 81% match → #2541`

All LAYER_0 rules still apply — no steps skipped, no confirmations bypassed.

---

## Bug Reporting

When asked for a bug report (any format — project-wide or single ticket):
1. Read `skills/core/BUG_REPORT_TEMPLATE.md` first
2. Fetch tickets from Odoo using MCP tools (project_id: 58, tag: bug)
3. Format output using the template structure

---

## Project Initiator (V1.0 + V1.1 + V1.3)

### DISCOVERY (V1.0 — done)

Skill location: `skills/project-initiator/DISCOVERY.md`

**Folder convention:** Each engagement gets its own named folder at `~/fiftyfive-engagements/<client-name>/` with an `input/` subfolder for raw docs. Multiple projects stay separate — one folder per client.

**How to run:** Open Claude Code in the `<client-name>/` folder (not in `input/`). Say `run DISCOVERY`. DISCOVERY reads the folder name as the engagement name automatically. Produces `discovery.md`.

Design spec: `docs/superpowers/specs/2026-06-03-project-initiator-design.md`

Test fixtures: `tests/fixtures/discovery/synthetic-01/` (SupplySync), `tests/fixtures/discovery/synthetic-02/` (StyleMart) — both scored 64/64 on 2026-06-03.

### MVP_SYNTHESIZER (V1.1 — done)

Skill location: `skills/project-initiator/MVP_SYNTHESIZER.md`

**How to run:** Open Claude Code in the `<client-name>/` folder (with `discovery.md` present). Say `run MVP_SYNTHESIZER`. Reads `discovery.md`, runs Readiness Gate, guides MVP scoping conversation. Produces `mvp-scope.md`.

**Readiness Gate:** runs at the top of MVP_SYNTHESIZER before any skill logic. Checks discovery.md for required fields (Core Problem, Users, ≥2 Features, Timeline, no unresolved CONFLICTs). BLOCKs ask inline — consultant never redirected to re-run DISCOVERY.

Design spec: `docs/superpowers/specs/2026-06-04-mvp-synthesizer-design.md`

Test fixtures: `tests/fixtures/mvp-synthesizer/synthetic-01/` (SupplySync PASS-path), `tests/fixtures/mvp-synthesizer/synthetic-02-incomplete/` (RetailEdge BLOCK-path)

### ARCH_PROPOSER (V1.3 — done)

Skill location: `skills/project-initiator/ARCH_PROPOSER.md`

**How to run:** Open Claude Code in the `<client-name>/` folder (with `discovery.md` and `mvp-scope.md` present). Say `run ARCH_PROPOSER`. Reads both files, runs Readiness Gate, guides architecture scoping conversation. Produces `architecture.md`.

**Readiness Gate:** runs at the top of ARCH_PROPOSER before any skill logic. Checks discovery.md and mvp-scope.md for required fields (core problem defined, MVP scope complete, user roles identified). BLOCKs ask inline — consultant never redirected to re-run prior skills.

Design spec: `docs/superpowers/specs/2026-06-04-mvp-synthesizer-design.md`

Test fixtures: `tests/fixtures/arch-proposer/synthetic-01/` (SupplySync PASS-path, 49/49), `tests/fixtures/arch-proposer/synthetic-02/` (RetailEdge BLOCK-path, 17/17)

### DOC_GENERATOR (V1.3.5 — done)

Skill location: `skills/project-initiator/DOC_GENERATOR.md`

**How to run:** Open Claude Code in the `<client-name>/` folder (with `discovery.md`, `mvp-scope.md`, and `arch.md` present). Say `run DOC_GENERATOR`. Runs sync check, presents menu, generates selected docs with Mermaid diagrams. Saves to `docs/` subfolder.

**Sync Check:** runs before the menu. Cross-validates all 3 input files for consistency. DRIFT items block the menu until resolved or deferred. WARNs pass through and are flagged in affected docs.

**Documents generated:** Project Proposal/SOW, Technical Architecture Doc, Sprint Plan, Developer Handoff Doc, Scope Agreement — each saved to `<engagement-folder>/docs/`.

Design spec: `docs/superpowers/specs/2026-06-04-doc-generator-design.md`

Test fixtures: `tests/fixtures/doc-generator/synthetic-01/` (RetailEdge PASS-path, 56/56), `tests/fixtures/doc-generator/synthetic-02-drift/` (RetailEdge DRIFT-path, 22/22)

### BACKLOG_GENERATOR (V1.4 — done)

Skill location: `skills/project-initiator/BACKLOG_GENERATOR.md`

**How to run:** Open Claude Code in the `<client-name>/` folder (with `arch.md` present).
Say `run BACKLOG_GENERATOR`. Creates Odoo project + stages, maps team roles, generates
structured ticket hierarchy, bulk-creates via MCP.

**Project Setup:** Asks for project name (defaults to folder name) and stages (defaults to
Backlog/To Do/In Progress/Bug/Done). Custom stages trigger purpose-mapping questions.

**Ticket format:** Parent ticket per Build Order item, subtasks per component. Backend tickets
get API endpoint tables (or Interfaces section for connectors/jobs); frontend tickets get
screen/component lists. Both include business rules, acceptance criteria, edge cases, and
relevant Open Questions.

**Duplicate check:** Always runs `list_tickets` before creation. Warns on matches, asks approval.

**Sprint tags:** Creates one Odoo tag per sprint label; every ticket tagged with its sprint.

Design spec: `docs/superpowers/specs/2026-06-04-backlog-generator-design.md`

Test fixtures: `tests/fixtures/backlog-generator/synthetic-01/` (72/72),
`tests/fixtures/backlog-generator/synthetic-02-duplicate/` (22/22)

### START (V1.2 — done)

Skill location: `skills/project-initiator/START.md`

**How to run:** From `~/fiftyfive-engagements/` parent directory. Say `run START`.
New project: creates folder + project.md + session_state.md with human confirmation.
Resume: scans engagement folders, displays registry, routes to next skill.
Also runs from inside a client folder as a shortcut to resume that engagement.

Design spec: `docs/superpowers/specs/2026-06-04-v12-orchestrator-design.md`

Test fixtures: `tests/fixtures/start/synthetic-new/` (new project path, 18/18), `tests/fixtures/start/synthetic-resume/` (registry + resume path, 19/19)

### Chain status

START (V1.2, done) → DISCOVERY (done) → MVP_SYNTHESIZER (done) → ARCH_PROPOSER (done) → DOC_GENERATOR (done) → BACKLOG_GENERATOR (V1.4.1, done) → ESTIMATOR (V1.5, in progress) → ROADMAP (future)

Note: PROJECT_INITIATOR orchestrator is target vision (V1.2+), now in progress with START skill.

### ESTIMATOR (V1.5 — in progress)

Skill location: `skills/project-initiator/ESTIMATOR.md`

**How to run:** Open Claude Code in the `<client-name>/` folder (with `arch.md` and
`backlog.md` present). Say `run ESTIMATOR`. Reads both files, runs pre-flight,
asks for estimate style (fixed/range), optional cost rates, and project start date.
Generates `estimates.md` with Summary + client summary + detailed breakdown. Ends with Odoo gate.

**Readiness Gate:** Checks for both `arch.md` (requires Effort Signals + Sprint Mapping +
Build Order) and `backlog.md`. Blocks with clear message pointing to missing upstream skill.

Design spec: `docs/superpowers/specs/2026-06-08-estimator-design.md`

Test fixtures: `tests/fixtures/estimator/synthetic-01/` (fixed hours PASS),
`tests/fixtures/estimator/synthetic-02-range/` (range + cost + date override)

MCP tools pending for future phases: `odoo-mcp/PENDING_CHANGES.md`

---

## MCP Server

Location: `odoo-mcp/` — fully built, 38/38 tests passing.
Key files: `server.py`, `odoo_client.py`, `read.py`, `write.py`, `utils.py`, `cache.py`
Do not modify MCP server files without running the test suite first.
