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
    project-initiator/     # DISCOVERY.md (V1.0), future skills
  context/                 # TEAM_CONTEXT.md, PARKING_LOT.md
  docs/
    superpowers/
      specs/               # Design specs (2026-06-03-project-initiator-design.md, etc.)
  tests/
    fixtures/
      discovery/           # Synthetic engagement fixtures for DISCOVERY testing
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
| PI V1.1+ | MVP_SYNTHESIZER → ARCH_PROPOSER → BACKLOG_GENERATOR → ROADMAP | Not started |

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

## Project Initiator (V1.0 — DISCOVERY only)

Skill location: `skills/project-initiator/DISCOVERY.md`

V1.0 scope: DISCOVERY skill only. No orchestrator. No SESSION_STATE.md. No per-engagement folder system.

**Folder convention:** Each engagement gets its own named folder at `~/fiftyfive-engagements/<client-name>/` with an `input/` subfolder for raw docs. Multiple projects stay separate — one folder per client.

**How to run:** Open Claude Code in the `<client-name>/` folder (not in `input/`). Say `run DISCOVERY`. DISCOVERY reads the folder name as the engagement name automatically.

Design spec: `docs/superpowers/specs/2026-06-03-project-initiator-design.md`

Test fixtures: `tests/fixtures/discovery/synthetic-01/` (SupplySync), `tests/fixtures/discovery/synthetic-02/` (StyleMart) — run DISCOVERY against these to verify the skill before deploying changes. Both fixtures scored 64/64 on 2026-06-03.

Note: PROJECT_INITIATOR orchestrator is target vision (V1.2+), not present in V1.0.

Target vision: DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → BACKLOG_GENERATOR → ROADMAP — full chain documented in spec, none of it built in V1.0.

MCP tools pending for future phases: `odoo-mcp/PENDING_CHANGES.md`

---

## MCP Server

Location: `odoo-mcp/` — fully built, 38/38 tests passing.
Key files: `server.py`, `odoo_client.py`, `read.py`, `write.py`, `utils.py`, `cache.py`
Do not modify MCP server files without running the test suite first.
