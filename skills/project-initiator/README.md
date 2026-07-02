# Project Initiator — README
# fiftyfive-tech internal consulting tooling
# Current: V1.5 (full chain complete through ESTIMATOR)

## What this is

Project Initiator is a Claude Code toolchain for fiftyfive-tech consultants. It takes rough, partial, or informal input documents from a client engagement and converts them into structured project-starting documentation and Odoo tickets through a guided conversation chain.

**Full chain:** START → DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → DOC_GENERATOR → BACKLOG_GENERATOR → ESTIMATOR

Each skill is a separate file. You run them one at a time from the engagement folder. Each skill writes an output file and updates `session_state.md` when done. If you forget where you are, say `run START` from the engagement folder — it tells you the current stage and next step.

---

## Skill files in this folder

| File | Version | What it does | Generates |
|---|---|---|---|
| `START.md` | V1.3 | Entry point — creates new engagements or resumes existing ones | `project.md`, `session_state.md` |
| `DISCOVERY.md` | V1.0 | Reads raw input docs, extracts requirements, resolves conflicts, fills gaps | `discovery.md` |
| `MVP_SYNTHESIZER.md` | V1.1 | Reads discovery.md, scopes the MVP via framing + feature prioritization | `mvp-scope.md` |
| `ARCH_PROPOSER.md` | V1.3 | Reads mvp-scope.md, selects stack, designs components, sprint map, effort signals | `arch.md` |
| `DOC_GENERATOR.md` | V1.3.5 | Reads all 3 upstream artifacts, generates client and internal docs | `docs/` subfolder |
| `BACKLOG_GENERATOR.md` | V1.4.1 | Reads arch.md, generates ticket hierarchy, creates tickets in Odoo via MCP | `backlog.md` + Odoo tickets |
| `ESTIMATOR.md` | V1.5 | Reads arch.md + backlog.md, converts S/M/L/XL to hours/days/cost | `estimates.md` |

---

## Folder convention

```
~/fiftyfive-engagements/
  active/
    supplysync/          ← one engagement folder per client
      input/             ← raw docs (PDF, text, markdown)
      project.md         ← created by START
      session_state.md   ← updated by each skill
      discovery.md       ← DISCOVERY output
      mvp-scope.md       ← MVP_SYNTHESIZER output
      arch.md            ← ARCH_PROPOSER output
      docs/              ← DOC_GENERATOR output
      backlog.md         ← BACKLOG_GENERATOR output
      estimates.md       ← ESTIMATOR output
  blocked/
  completed/
  archived/
```

---

## Quickstart — new engagement

```
1. Open Claude Code in ~/fiftyfive-engagements/
2. Say: run START
   → answer 3 questions (client name, engagement name, project type)
   → START creates the folder structure and tells you to drop docs in input/

3. Drop raw docs into ~/fiftyfive-engagements/active/<slug>/input/

4. Open Claude Code in ~/fiftyfive-engagements/active/<slug>/
5. Say: run DISCOVERY  → produces discovery.md
6. Say: run MVP_SYNTHESIZER  → produces mvp-scope.md
7. Say: run ARCH_PROPOSER  → produces arch.md
8. Say: run DOC_GENERATOR  → produces docs/ folder (optional, not required for tickets)
9. Say: run BACKLOG_GENERATOR  → produces backlog.md + creates Odoo project + tickets
10. Say: run ESTIMATOR  → produces estimates.md
```

Each skill confirms the output before writing — you review the draft and approve it.

---

## Skill details

### START (V1.3)

**Run from:** `~/fiftyfive-engagements/` (parent directory)  
**Shortcut:** also run from inside an engagement folder to resume that engagement directly

**Two flows:**
- **New project** — asks client name, engagement name, project type. Shows exactly what will be created before doing anything. Creates `active/<slug>/`, `input/`, `project.md`, `session_state.md`.
- **Resume** — scans `active/`, `blocked/`, `completed/`, `archived/` buckets. Shows a numbered registry sorted by last session date. Select a number → see current stage + open items + next skill to run.

**Status transitions:** after showing resume output, offers to move an engagement between buckets (active → blocked → completed → archived → reactivate). Shows confirmation with exact folder move paths before executing.

---

### DISCOVERY (V1.0)

**Run from:** `<client-name>/` folder  
**Requires:** `input/` folder with at least one `.pdf`, `.txt`, or `.md` file

**Flow:**
1. Reads all input docs, runs silent extraction across 10 categories
2. Shows a compact summary with confidence levels (HIGH / MED / LOW)
3. Resolves conflicts first (same topic stated differently across docs)
4. Asks critical gap questions — things MVP_SYNTHESIZER needs that aren't in the docs
5. Lets you defer any answer to Open Questions
6. Shows full draft for review → you approve before it writes

**Output — `discovery.md`:**
- Project Context, Users, Core Problem
- Features Mentioned (with confidence levels)
- Constraints (timeline, budget, tech, other)
- Open Questions (deferred items)
- Confidence Notes (LOW confidence and resolved CONFLICTs)
- Source Docs (one line per input file)

**Boundary:** extracts and clarifies only. Does not choose architecture, scope MVP, or create tickets.

---

### MVP_SYNTHESIZER (V1.1)

**Run from:** `<client-name>/` folder  
**Requires:** `discovery.md`

**Flow:**
1. **Readiness Gate** — checks discovery.md for core problem, users, ≥2 features, timeline, no unresolved conflicts. BLOCKs must be resolved inline before proceeding.
2. **Framing** — pick one: Time-boxed / Risk-first / Value-first
3. **Feature prioritization** — every feature from DISCOVERY gets IN / OUT / DEFERRED. High-confidence features are batched; non-obvious calls are asked one at a time.
4. **User journeys** — 2–4 key end-to-end flows; shown as a single block for review
5. **Success metrics** — asked once; can be deferred to Open Questions
6. Full draft review → approve before it writes

**Output — `mvp-scope.md`:**
- Problem Restatement, Users, MVP Framing
- Scope: In table (feature, description, confidence, rationale)
- Scope: Out table (feature, why out, deferred to)
- Key User Journeys
- Success Metrics, Constraints, Assumptions
- Open Questions (carried from DISCOVERY + new ones)
- Effort Signals placeholder (deferred to ARCH_PROPOSER)

**Boundary:** scopes the MVP only. No architecture, no tech selection, no tickets.

---

### ARCH_PROPOSER (V1.3)

**Run from:** `<client-name>/` folder  
**Requires:** `mvp-scope.md`

**Flow:**
1. **Readiness Gate** — checks mvp-scope.md for scope, users, tech constraints, timeline, budget, no unresolved conflicts
2. **Stack selection** — locked tech from mvp-scope.md shown first; unconfirmed layers (backend, database, infra, mobile) presented one at a time with 2–3 options + recommendation
3. **Component design** — derives components per feature from the confirmed stack; presented as a single block for review; shared components flagged inline
4. **Integration points** — identified silently, presented with risk ratings (HIGH / MED / LOW) and open questions
5. **Team input** — asks for team roster (roles + seniority)
6. **Build order + sprint mapping** — dependency-driven order, then sprint assignment table
7. **Effort signals** — S/M/L/XL per feature based on component count + integration risk
8. **Client Summary** — plain-language summary produced last (no tech jargon); written as final section
9. Full draft review → approve before it writes

**Output — `arch.md`:**
- Client Summary (plain language, no jargon)
- Tech Stack table (with `[STRAWMAN]` markers for tentative decisions)
- Components (per feature; shared components noted)
- Data Model Hints (key tables and fields)
- Integration Points table (system, approach, risk, open questions)
- Build Order (numbered, dependency-driven)
- Sprint Mapping (team roster + sprint table with owners)
- Effort Signals (S/M/L/XL per feature with rationale)
- Open Questions (with `blocks:` field — used by BACKLOG_GENERATOR)
- STRAWMAN Summary (all tentative decisions — becomes STRAWMAN tickets in Odoo)

**Boundary:** defines architecture only. Does not create Odoo tickets.

---

### DOC_GENERATOR (V1.3.5)

**Run from:** `<client-name>/` folder  
**Requires:** `discovery.md`, `mvp-scope.md`, `arch.md` all present

**Flow:**
1. **Sync Check** — cross-validates all three input files for consistency. DRIFT items (contradictions between files) block the menu until resolved or deferred. WARNs pass through.
2. **Menu** — select which documents to generate (any combination)
3. **Generation** — produces each selected doc with Mermaid diagrams; consultant approves each before saving
4. Saved to `<engagement-folder>/docs/`

**Available documents:**
- **Project Proposal / SOW** — client-facing scope, timeline, commercial terms
- **Technical Architecture Doc** — stack decisions, component diagram, data model, integration notes
- **Sprint Plan** — sprint-by-sprint breakdown for internal use
- **Developer Handoff Doc** — component specs, API interfaces, data model, open questions
- **Scope Agreement** — client sign-off document

**Boundary:** presentation layer only. Asks no new questions — all data comes from upstream artifacts. Does not modify `discovery.md`, `mvp-scope.md`, or `arch.md`.

---

### BACKLOG_GENERATOR (V1.4.1)

**Run from:** `<client-name>/` folder  
**Requires:** `arch.md` with `## Sprint Mapping`, `## Build Order`, `## Components`, `## Tech Stack`

This skill is the bridge between the architecture document and Odoo. It reads arch.md, generates a full ticket hierarchy, previews it for approval, writes `backlog.md`, then creates the tickets in Odoo via MCP tools.

**Flow:**

1. **Project Setup** — confirms project name and stages; calls `list_active_projects` to detect existing projects before creating

2. **Sprint Tags** — creates one Odoo tag per sprint label (`create_tag`); every ticket is tagged with its sprint

3. **Team Mapping** — maps arch.md roles to Odoo user IDs; validates each via `list_metadata`

4. **Duplicate Check** — calls `list_tickets` and compares ticket titles against existing; warns + asks approval before proceeding

5. **Ticket generation (internal):**
   - One **parent ticket** per Build Order item: `[Sprint N] <item name>`
   - One **subtask** per component (shared components appear once only — in the ticket where first built)
   - One **STRAWMAN ticket** per item in `## STRAWMAN Summary` — these are pre-conditions placed in Backlog stage
   - Ticket type inferred from component tech (React/Vue → frontend tag; Node.js/.NET → backend tag)
   - Open Questions from arch.md added to relevant tickets only (those whose `blocks:` field matches)

6. **Preview + Approval Gate** — full table shown before any write. You can request edits before approving. `backlog.md` is written on approval.

7. **Next Steps Gate** — pick:
   - `[A]` Create tickets in Odoo now
   - `[B]` Run ESTIMATOR first (ESTIMATOR has its own Odoo gate)
   - `[C]` Keep backlog.md as DRAFT, push to Odoo later

8. **Bulk Creation** (if `[A]` chosen):
   - `bulk_create_tickets` for all parent tickets
   - `add_subtasks` for each parent's component subtasks
   - `bulk_create_tickets` (separate call) for STRAWMAN tickets in Backlog stage
   - `backlog.md` status updated to `CREATED`

**Output files:**
- `backlog.md` — full backlog draft (project config, team mapping, sprint tags, all ticket descriptions with metadata)

**Odoo output:**
- New project (or uses existing) with stages
- Sprint tags in Odoo
- Parent tickets (one per Build Order item) with full descriptions and metadata
- Subtasks under each parent
- STRAWMAN pre-condition tickets in Backlog stage

**Odoo MCP tools used:** `list_active_projects`, `create_project`, `create_tag`, `list_metadata`, `list_tickets`, `bulk_create_tickets`, `add_subtasks`

**Push Mode:** If `backlog.md` exists with `Status: DRAFT`, re-running BACKLOG_GENERATOR offers to push the existing draft without regenerating it.

---

### ESTIMATOR (V1.5)

**Run from:** `<client-name>/` folder  
**Requires:** `arch.md` (with `## Effort Signals`, `## Sprint Mapping`, `## Build Order`) + `backlog.md`

**Flow:**
1. Detects project start date from arch.md → asks to confirm or override (date shift preserves sprint durations)
2. **Estimation config** (single block): estimate style (fixed or range), working hours/day, include cost?
3. If cost: blended day rate or per-role rates
4. Computes hours, days, cost, sprint dates for every Build Order item
5. Shows review table — you can adjust any row (`3: 32h` or `3: 24–40h`). Rows with unknown size must be filled before writing.
6. Writes `estimates.md`
7. **Odoo Gate** — same 3 options as BACKLOG_GENERATOR Next Steps Gate (skipped if backlog.md was already CREATED)

**Output — `estimates.md`:**
- **Summary block** — total effort, timeline, team size, sprint count, cost
- **Section 1 (Client Summary)** — sprint-by-sprint table; one row per Build Order item
- **Section 2 (Detailed Breakdown)** — one row per parent ticket with sprint, assignee, start/end, hours/days/cost
- **Section 3 (Assumptions)** — size-to-hours mapping, working hours/day, cost rates, exclusions

Format: flat Markdown tables (no merged cells, no nested rows). Suitable for copy-paste to Excel, PDF, or Word. S/M/L/XL labels never appear — always converted to hours/days before writing.

**Boundary:** read-only on all upstream files. Does not modify arch.md, backlog.md, or any other file.

---

## How to create Odoo tickets — end to end

### Prerequisites
- `arch.md` complete in the engagement folder
- Odoo MCP server connected (`odoo-mcp/`)
- Odoo user IDs for your team (use `list_metadata` if unsure)

### Shortest path
```
1. Open Claude Code in ~/fiftyfive-engagements/<client-name>/
2. Say: run BACKLOG_GENERATOR
3. Accept project name → Enter
4. Accept default stages → yes
5. Enter Odoo user IDs for each role (or type "skip")
6. Review the ticket preview table
7. Type: yes  (or [A] to approve and write backlog.md)
8. Next Steps Gate: type [A] to create in Odoo now
9. Odoo tickets are created — backlog.md updated to Status: CREATED
```

### What gets created in Odoo

| Item | Stage | Priority | Notes |
|---|---|---|---|
| Odoo project | — | — | One per engagement |
| Sprint tags | — | — | One tag per sprint label |
| Parent tickets | To Do | Low (XL → High) | One per Build Order item, tagged type + sprint |
| Subtasks | Inherited | — | One per component; shared not duplicated |
| STRAWMAN tickets | Backlog | High | One per tentative arch decision; unassigned |

### Re-running after a skip
If you chose `[C] Skip` earlier and want to push later:
1. Re-run `run BACKLOG_GENERATOR`
2. It detects `backlog.md` with `Status: DRAFT`
3. Select `[A] Push backlog.md to Odoo now`
4. Existing data is used — no regeneration needed

---

## Design spec

`docs/superpowers/specs/` — spec files for each skill (arch proposer, backlog generator, doc generator, estimator, etc.)

## Test fixtures

`tests/fixtures/` — one subfolder per skill with synthetic engagement inputs and expected behavior definitions.
