# TiffinConnect AI Ticket Pipeline — System Flow
# VERSION: 1.0 | Created: 2026-05-21 | Phase: 1 complete, Phase 2 not started

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

---

## Phase 2 Additions (not built — planned)

When Phase 2 is complete, these will slot in between LAYER_0 and the skill files:

```
LAYER_1_CLASSIFIER      — smart routing brain, replaces Routing Rule
LAYER_2_PREPROCESSOR    — structured interview for vague requirements
LAYER_2_DISCOVERY       — full BA-style interview
REQUIREMENT_BRIDGE      — PRD → TRD conversion
LAYER_3_TICKET_MAPPER   — replaces S9 inline mapper
PRD_BREAKDOWN           — decompose PRD into discrete tickets
SESSION_STATE           — session tracking for long runs
```

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

*teal = Claude skill layer · amber = MCP execution layer · green = Odoo data*
