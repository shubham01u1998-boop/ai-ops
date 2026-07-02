# TiffinConnect AI Ticket Pipeline — System Flow
# VERSION: 1.2 | Created: 2026-05-21 | Updated: 2026-06-09
# Phase 1 (ticket intake): complete | Phase 2: not started
# Project Initiator V1.5 (ESTIMATOR): done | Chain: START → DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → DOC_GENERATOR → BACKLOG_GENERATOR → ESTIMATOR

---

## Current Flow (Phase 1 — Production)

```
╔══════════════════════════════════════════════════════════════════════╗
║  HUMAN INPUT                                                         ║
║  Bug · Feature · QA batch · Open question · PRD section             ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ MCP Protocol
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  ALWAYS LOADED  (Claude Enterprise — every session)                  ║
║  ┌──────────────────────────┐  ┌──────────────────────────────────┐  ║
║  │  LAYER_0_GLOBAL.md v1.6  │  │  TEAM_CONTEXT.md v1.3            │  ║
║  │  Rule 1  Permission      │  │  S1  Team roster + system roles  │  ║
║  │  Rule 2  Token estimate  │  │  S2  Project routing (ID: 58)    │  ║
║  │  Rule 3  Session budget  │  │  S3  Tag taxonomy (IDs 2,44,4)   │  ║
║  │  Rule 3b PARKING_LOT     │  │  S4  Priority rules              │  ║
║  │  Rule 4  Output limits   │  │  S5  Learned decisions           │  ║
║  │  Rule 5  No narration    │  │  S6  Known exceptions            │  ║
║  │  Rule 6  One question    │  │  S7  Out-of-scope list           │  ║
║  │  Rule 7  No repeat ask   │  └──────────────────────────────────┘  ║
║  │  Rule 8  TEAM_CONTEXT    │                                        ║
║  │  Routing Rule (Phase 1)  │                                        ║
║  └──────────────────────────┘                                        ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║
                          Routing Rule (LAYER_0)
                          ┌────────────────────┐
                          │ QA batch / TC-NNN? │
                          └──────┬──────┬───── ┘
                                NO    YES
                                 │      │
                                 ▼      ▼
╔══════════════════════════╗  ╔════════════════════════╗  ╔═════════════════════╗
║  LAYER_2_FASTPATH v2.4   ║  ║  QA_INTAKE v1.5        ║  ║ BUG_REPORT_         ║
║  S1  Classifier          ║  ║  S1  Input parser      ║  ║ TEMPLATE.md         ║
║  S2  Extraction pass     ║  ║  S2  Dedup check       ║  ║                     ║
║  S3  Assumption surface  ║  ║  S3  Batch draft       ║  ║ Loaded only when    ║
║  S4  REQ{} internal doc  ║  ║  S4  QA review table   ║  ║ bug report is       ║
║  S5  Batch review table  ║  ║  S5  Incomplete check  ║  ║ explicitly asked    ║
║  S6  Question handler    ║  ║  S6  Post-create sum   ║  ║ Fetches from Odoo   ║
║  S7  Multi-req detect    ║  ║      → calls S9        ║  ║ tag: bug, proj: 58  ║
║  S8  Duplicate check     ║  ╚════════════════════════╝  ╚═════════════════════╝
║  S9  Inline mapper       ║
╚══════════════════════════╝
          │
          │  Human types: CONFIRM ALL
          ▼
╔══════════════════════════════════════════════════════════════════════╗
║  FALLBACK PATHS  (PARKING_LOT.md — loaded when available)            ║
║  ┌───────────────────────────────────────────────────────────────┐   ║
║  │  S1  Retry Queue     — failed/skipped creates (7d TTL)       │   ║
║  │  S2  Deferred Req    — needs product decision  (30d TTL)     │   ║
║  │  S3  Open Questions  — exploratory thoughts   (14d TTL)      │   ║
║  │  S4  Vague Findings  — QA too vague to draft  (7d TTL)      │   ║
║  └───────────────────────────────────────────────────────────────┘   ║
║  Session-start checklist rules live in PARKING_LOT_SPEC.md           ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ create_ticket() / update_ticket()
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  MCP SERVER  (local Python — odoo-mcp/)  38/38 tests passing         ║
║  ┌──────────────────┐ ┌──────────────────┐ ┌────────────┐ ┌───────┐ ║
║  │  read.py         │ │  write.py        │ │  utils.py  │ │cache  │ ║
║  │  list_tickets    │ │  create_ticket   │ │  list_proj │ │TTL    │ ║
║  │  search_tickets  │ │  update_ticket   │ │  list_stg  │ │LRU    │ ║
║  │  get_ticket      │ │  transition_stg  │ │  list_tags │ │session│ ║
║  └────────┬─────────┘ └────────┬─────────┘ └─────┬──────┘ └───┬───┘ ║
║           └──────────────────── ┤ ─────────────────┘           │    ║
║                                 ▼                               │    ║
║                        odoo_client.py                           │    ║
║                  XML-RPC · API key auth                         │    ║
║                  HTML strip · field projection                  │    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  ODOO 19 ENTERPRISE                                                  ║
║  fiftyfive-technologies-pvt-ltd.odoo.com                            ║
║  ┌──────────────────────┐ ┌──────────────────────┐ ┌─────────────┐ ║
║  │  project.task        │ │  Stages (proj: 58)   │ │  Tags       │ ║
║  │  title · tags        │ │  Backlog     (347)   │ │  frontend 2 │ ║
║  │  stage · priority    │ │  To Do       (348)   │ │  backend 44 │ ║
║  │  description (HTML)  │ │  In Progress (349)   │ │  bug      4 │ ║
║  │  assignee_ids (empty)│ │  Bug         (350)   │ └─────────────┘ ║
║  └──────────────────────┘ │  Done        (351)   │                 ║
║                           └──────────────────────┘                 ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## Key: What changed vs. old diagram

| Component | Old | Now |
|---|---|---|
| PARKING_LOT | Not in flow | Full fallback layer — 4 sections, TTL-based |
| PARKING_LOT_SPEC | — | Separate rules file, read once at session start |
| Claude Code entry point | Redirect to Enterprise | Full pipeline runs in Claude Code — same skill files, same MCP, same output |
| Debug mode | Not available | Auto-detected in Claude Code — shows REQ{}, MCP calls, raw errors, similarity scores |
| QA_INTAKE routing | Implicit | Explicit gate in LAYER_0 Routing Rule |
| QA_INTAKE execution order | — | Section numbers ≠ execution order: S1→S3→S2→S4→S5→S6 (S3 draft runs before S2 dedup — REQ{} must exist first) |
| FASTPATH DROP command | — | Phase 1 limitation: dropped tickets are NOT saved to PARKING_LOT.md — lost at session end. PARKING_LOT.md saving for drops is Phase 2. |
| BUG_REPORT_TEMPLATE | In knowledge | In knowledge — explicit load condition added |
| S3b Draft Queue Failure | Not shown | Wired into failure path in S9 |
| TEAM_CONTEXT System Roles | Not shown | QA Lead / PM / Lead / Triage Owner now defined |
| Phase 2 classifier | Shown as S1 box | Not yet built — Routing Rule is the temp replacement |
| LAYER_2_DISCOVERY | Planned in Phase 2 | Retired — covered by Project Initiator DISCOVERY skill |
| FASTPATH S1 "DISCOVERY path" | Named DISCOVERY | Renamed to INTAKE_INTERVIEW path — avoids name collision with PI skill |
| Project Initiator | Not in flow | Separate pipeline — START → DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → DOC_GENERATOR → BACKLOG_GENERATOR → ESTIMATOR → ROADMAP (future) |
| Readiness Gate | Not in flow | Embedded in each downstream PI skill — validates upstream artifact before proceeding |

---

## Phase 2 Additions (not built — planned)

When Phase 2 is complete, these will slot in between LAYER_0 and the skill files:

```
LAYER_1_CLASSIFIER      — smart routing brain, replaces Routing Rule
LAYER_2_PREPROCESSOR    — structured interview for vague requirements
REQUIREMENT_BRIDGE      — PRD → TRD conversion
LAYER_3_TICKET_MAPPER   — replaces S9 inline mapper
PRD_BREAKDOWN           — decompose PRD into discrete tickets
SESSION_STATE           — session tracking for long runs
```

Note: `LAYER_2_DISCOVERY` has been retired from Phase 2 scope. The Project Initiator's
DISCOVERY skill (skills/project-initiator/) covers BA-style extraction for new engagements.
The FASTPATH S1 reference to "DISCOVERY path" has been renamed "INTAKE_INTERVIEW path"
to eliminate the name collision.

---

## Demo Readiness — Will this work in a demo session?

**YES — the flow is identical in demo and production sessions.** Evidence:

| Check | Status |
|---|---|
| MCP server built and tested | 38/38 tests passing |
| Odoo instance live | fiftyfive-technologies-pvt-ltd.odoo.com |
| Phase 1 skill files production-ready | All versioned and reviewed |
| PARKING_LOT.md exists and is empty | Ready to receive items |
| PARKING_LOT_SPEC.md in session context | Required alongside PARKING_LOT.md — contains session-start checklist rules |
| Tag IDs verified in Odoo | 2, 44, 4 confirmed (fallbacks 43, 1, 45 documented in TEAM_CONTEXT S3) |
| Stage IDs verified | Backlog: 347 through Done: 351 |
| Team Odoo IDs verified | 50, 62, 41, 57, 42 |
| Stray test tickets cleared | #2552, #2551, #2624, #2626, #2627, #2628, #2629 deleted 2026-05-21 |
| Duplicate check pre-flight | 0 collisions on all 5 demo scenario inputs |

**Two files required for demo:** Both `PARKING_LOT.md` and `PARKING_LOT_SPEC.md` must be loaded into the Claude Enterprise session context alongside the skill files. PARKING_LOT.md is the data file; PARKING_LOT_SPEC.md contains the session-start checklist rules that Rule 3b reads at startup. Loading only PARKING_LOT.md means the checklist does not run — fallback paths degrade silently.

**Claude Code (ai-ops) now runs the same intake flow as Claude Enterprise.** The same demo script works in both environments. Claude Code also supports debug mode — auto-detected when input is an inspection or system question — which shows REQ{} blocks, MCP calls, raw errors, and similarity scores.

**What Phase 2 absence means for demo:** The Routing Rule in LAYER_0 handles routing manually (QA batch → QA_INTAKE, everything else → FASTPATH). This is the designed Phase 1 behaviour — not a demo limitation.

---

## S8 Duplicate Check — Actual MCP Call Pattern

This is frequently described incorrectly in presenter notes. The real behaviour:

**What it does NOT do:** 3 × `search_tickets` calls per ticket.

**What it actually does:**
1. Resolves primary dedup tag from the REQ{} block (e.g. `bug`)
2. Checks session cache — if already fetched this tag, uses cached data (0 MCP calls)
3. Cache miss only: calls `list_tickets(project_id=58, tag="bug", limit=50)` — one call, paginates if `has_more=true`
4. Stores `[id, title]` pairs as `ticket_cache["bug"]` for the rest of the session
5. Scores all cached titles against the new ticket title **in memory** — no further MCP calls
6. ≥ 80% similarity → high-confidence duplicate → pauses at CONFIRM ALL
7. 50–79% similarity → possible duplicate → flagged in batch table, no pause

**Presenter walkthrough line (correct version):**
> "Claude fetched all existing bug tickets once and matched your ticket title against them in memory. That single `list_tickets` call is the entire duplicate check — results are cached so it won't re-fetch for the rest of the session."

**Note on two Vijays:** Odoo has `vijay.jangid (ID 23)` and `vijay.mehrotra (ID 62)`. TEAM_CONTEXT Section 1 maps "Vijay" → Mehrotra (62). Claude will always resolve to ID 62. Heads-up the team so there is no confusion during live use.

---

---

## Project Initiator Flow (separate pipeline — runs from engagement folders)

This pipeline is independent of the ticket intake flow above. It runs from
`~/fiftyfive-engagements/<client-name>/` via Claude Code, not Claude Enterprise.

Chain: START → DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → DOC_GENERATOR → BACKLOG_GENERATOR → ESTIMATOR → ROADMAP (future)

```
╔══════════════════════════════════════════════════════════════════════╗
║  ENGAGEMENT FOLDER  ~/fiftyfive-engagements/<client-name>/           ║
║  input/             ← raw docs (PDF, text, markdown)                 ║
║  project.md         ← created by START, updated by each skill        ║
║  session_state.md   ← written by each skill on completion            ║
║  discovery.md       ← produced by DISCOVERY                          ║
║  mvp-scope.md       ← produced by MVP_SYNTHESIZER                    ║
║  arch.md            ← produced by ARCH_PROPOSER                      ║
║  backlog.md         ← produced by BACKLOG_GENERATOR                  ║
║  estimates.md       ← produced by ESTIMATOR                          ║
║  docs/              ← produced by DOC_GENERATOR (SOW, arch doc, etc) ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ say: run START (from parent dir)
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  START  (V1.2 — done)  skills/project-initiator/START.md             ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Pre-flight: detect location (parent dir or engagement dir) │    ║
║  │                                                             │    ║
║  │  New Project Flow                                           │    ║
║  │    Ask: client name · engagement name · project type        │    ║
║  │    Confirm before creating: folder + input/ + project.md   │    ║
║  │    Write project.md + session_state.md (Stage: not started) │    ║
║  │    Output: "Drop docs in input/, then run DISCOVERY"        │    ║
║  │                                                             │    ║
║  │  Resume Flow (registry from all project.md files)           │    ║
║  │    Display: name · client · type · stage · last session     │    ║
║  │    On select: read session_state.md → show next step        │    ║
║  │                        + open items from last session       │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ say: run DISCOVERY
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  DISCOVERY  (V1.0 — done)  skills/project-initiator/DISCOVERY.md    ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Pre-flight                                                  │    ║
║  │    basename "$PWD" → engagement name                        │    ║
║  │    Glob input/*    → list + read all docs                   │    ║
║  │                                                             │    ║
║  │  Extraction Pass (silent)                                   │    ║
║  │    Project · Users · Core Problem · Features · Tech         │    ║
║  │    Timeline · Constraints · Conflicts · Gaps                │    ║
║  │    Confidence: HIGH / MED / LOW per category                │    ║
║  │                                                             │    ║
║  │  Question Loop  (one at a time)                             │    ║
║  │    Phase 1: resolve conflicts first                         │    ║
║  │    Phase 2: critical gaps                                   │    ║
║  │    Phase 3: detail gaps (deferrable)                        │    ║
║  │                                                             │    ║
║  │  Draft Artifact → consultant approves → Save discovery.md  │    ║
║  │  Writes session_state.md  ·  Updates project.md            │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ discovery.md  ·  say: run MVP_SYNTHESIZER
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  MVP_SYNTHESIZER  (V1.1 — done)                                      ║
║  skills/project-initiator/MVP_SYNTHESIZER.md                         ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Readiness Gate on discovery.md                             │    ║
║  │    Required: Core Problem · Users · ≥2 Features ·           │    ║
║  │              Timeline · no unresolved CONFLICTs             │    ║
║  │                                                             │    ║
║  │  Framing Selection  (consultant picks one)                  │    ║
║  │    A) Time-boxed  · B) Risk-first  · C) Value-first        │    ║
║  │                                                             │    ║
║  │  Feature Prioritization  (IN / OUT / DEFERRED)              │    ║
║  │  User Journey Extraction  ·  Success Metrics                │    ║
║  │                                                             │    ║
║  │  Draft Artifact → consultant approves → Save mvp-scope.md  │    ║
║  │  Writes session_state.md  ·  Updates project.md            │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ mvp-scope.md  ·  say: run ARCH_PROPOSER
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  ARCH_PROPOSER  (V1.3 — done)  skills/project-initiator/ARCH_PROPOSER.md ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Readiness Gate on discovery.md + mvp-scope.md              │    ║
║  │                                                             │    ║
║  │  Tech Stack Menu  (constrained options, STRAWMAN marker)    │    ║
║  │  Component Map  (frontend · backend · infra · integrations) │    ║
║  │  Build Order  (sprint-mapped, dependency-ordered)           │    ║
║  │  Effort Signals  (S/M/L/XL per component)                   │    ║
║  │                                                             │    ║
║  │  Draft Artifact → consultant approves → Save arch.md       │    ║
║  │  Writes session_state.md  ·  Updates project.md            │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ arch.md  ·  say: run DOC_GENERATOR
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  DOC_GENERATOR  (V1.3.5 — done)  skills/project-initiator/DOC_GENERATOR.md ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Sync Check: cross-validates discovery.md + mvp-scope.md   │    ║
║  │              + arch.md for drift / inconsistency            │    ║
║  │    DRIFT → blocks menu until resolved or deferred           │    ║
║  │    WARN  → passes through, flagged in affected docs         │    ║
║  │                                                             │    ║
║  │  Document Menu  (consultant picks one or more):             │    ║
║  │    1. Project Proposal / SOW                                │    ║
║  │    2. Technical Architecture Doc                            │    ║
║  │    3. Sprint Plan                                           │    ║
║  │    4. Developer Handoff Doc                                 │    ║
║  │    5. Scope Agreement                                       │    ║
║  │                                                             │    ║
║  │  Each doc saved to docs/ with Mermaid diagrams              │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ docs/  ·  say: run BACKLOG_GENERATOR
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  BACKLOG_GENERATOR  (V1.4.1 — done)                                  ║
║  skills/project-initiator/BACKLOG_GENERATOR.md                       ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Readiness Gate on arch.md                                  │    ║
║  │                                                             │    ║
║  │  Project Setup: name + stages (Odoo project created)        │    ║
║  │  Team Role Mapping  (Odoo user IDs)                         │    ║
║  │  Ticket Hierarchy: parent per Build Order item + subtasks   │    ║
║  │  Duplicate Check: list_tickets before creation              │    ║
║  │  Sprint Tags: one Odoo tag per sprint, every ticket tagged  │    ║
║  │                                                             │    ║
║  │  Writes backlog.md → CONFIRM ALL → bulk_create_tickets      │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ backlog.md + Odoo project  ·  say: run ESTIMATOR
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  ESTIMATOR  (V1.5 — done)  skills/project-initiator/ESTIMATOR.md    ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  Readiness Gate on arch.md + backlog.md                     │    ║
║  │    Required: Effort Signals + Sprint Mapping + Build Order   │    ║
║  │                                                             │    ║
║  │  Config: estimate style (fixed / range) · cost rates        │    ║
║  │          project start date                                 │    ║
║  │                                                             │    ║
║  │  Compute: hours per sprint · contingency · total cost       │    ║
║  │  Review Table: sprint-by-sprint breakdown                   │    ║
║  │                                                             │    ║
║  │  Writes estimates.md  →  Odoo gate (optional)               │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ estimates.md
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║  ROADMAP  (future)                                                   ║
║    Planned: delivery roadmap from Odoo board + arch.md               ║
║    Output: roadmap.md — quarterly view, milestone format             ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Readiness Gate Pattern (embedded in each downstream skill)

Every downstream skill opens with a Readiness Gate that validates the upstream artifact
before any skill logic runs. Gate never auto-fills. Gate never redirects to a previous
skill — resolves inline.

| Status | Meaning | Behaviour |
|---|---|---|
| ✓ PASS | Field present, confidence HIGH or MED | Proceed silently |
| ⚠ WARN | Present but LOW confidence, or ambiguous | Flag in output Confidence Notes; do not block |
| ✗ BLOCK | Field missing or unresolved CONFLICT | Ask one inline question; proceed only after resolved or deferred |

Deferred BLOCKs become WARNs + Open Questions in the output artifact.

### Project Initiator — Test Fixtures

| Fixture | Path | Score |
|---|---|---|
| DISCOVERY synthetic-01 | tests/fixtures/discovery/synthetic-01/ | SupplySync PASS-path — 64/64 |
| DISCOVERY synthetic-02 | tests/fixtures/discovery/synthetic-02/ | StyleMart PASS-path — 64/64 |
| MVP_SYNTHESIZER synthetic-01 | tests/fixtures/mvp-synthesizer/synthetic-01/ | PASS-path — clean discovery.md |
| MVP_SYNTHESIZER synthetic-02-incomplete | tests/fixtures/mvp-synthesizer/synthetic-02-incomplete/ | BLOCK-path — 3 deliberate BLOCKs |
| START synthetic-new | tests/fixtures/start/synthetic-new/ | New project creation — 18/18 |
| START synthetic-resume | tests/fixtures/start/synthetic-resume/ | Registry + resume path — 19/19 |
| ARCH_PROPOSER synthetic-01 | tests/fixtures/arch-proposer/synthetic-01/ | SupplySync PASS-path — 49/49 |
| ARCH_PROPOSER synthetic-02-incomplete | tests/fixtures/arch-proposer/synthetic-02-incomplete/ | RetailEdge BLOCK-path — 17/17 |
| DOC_GENERATOR synthetic-01 | tests/fixtures/doc-generator/synthetic-01/ | RetailEdge PASS-path — 56/56 |
| DOC_GENERATOR synthetic-02-drift | tests/fixtures/doc-generator/synthetic-02-drift/ | RetailEdge DRIFT-path — 22/22 |
| BACKLOG_GENERATOR synthetic-01 | tests/fixtures/backlog-generator/synthetic-01/ | PASS-path — 72/72 |
| BACKLOG_GENERATOR synthetic-02-duplicate | tests/fixtures/backlog-generator/synthetic-02-duplicate/ | Duplicate detection — 22/22 |
| ESTIMATOR synthetic-01 | tests/fixtures/estimator/synthetic-01/ | Fixed hours PASS — 48/48 |
| ESTIMATOR synthetic-02-range | tests/fixtures/estimator/synthetic-02-range/ | Range + cost + date override — 45/45 |

---

*teal = Claude skill layer · amber = MCP execution layer · green = Odoo data*
