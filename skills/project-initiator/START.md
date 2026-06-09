# VERSION: 1.0 | Last updated: 2026-06-04 | Reviewed: pending
# START — Project Initiator V1.2
# Part of the fiftyfive-tech Project Initiator toolchain.

---

## Purpose

START is the entry point for the Project Initiator toolchain.

Two flows:
- **New project:** guided setup — asks client name, engagement name, project type,
  confirms before creating folder structure, writes project.md + session_state.md
- **Resume:** scans for active engagements, displays registry with stage + last session date,
  routes consultant to next step

Run from `~/fiftyfive-engagements/` (parent directory) for full registry.
Run from inside a client folder (shortcut) to resume that specific engagement directly.

V1.2 scope: this skill only. No auto-routing between skills — consultant still invokes
each skill manually. START tells them which one is next.

---

## Pre-flight

Before asking anything, silently:

1. Run `basename "$PWD"` via Bash tool.
2. Use Read tool to check if `project.md` exists in the current directory.
   - If yes: this is an engagement folder → go to **Resume Single Engagement**.
   - If no: this is the parent directory → continue.
3. Scan for engagement folders:
   a. Run `ls -d */` via Bash from current directory (ignore errors if no subdirs).
   b. Known status buckets: active, blocked, completed, archived.
   c. For each subdirectory found:
      - If the subdirectory name matches a known bucket → run `ls -d <bucket>/*/`
        via Bash to list engagements inside. Mark each found path with its bucket name.
      - Otherwise → attempt Read on `<subdir>/project.md` directly.
        If found, mark as status: active (legacy flat folder — no migration prompted).
   d. For each engagement path collected (bucketed or flat), attempt Read on
      `<path>/project.md`. On success, add to registry: {path, status, project_data}.
   e. Build registry sorted by Last session date, most recent first.
      Within each status group, sort descending.
4. If registry list is empty → go directly to **New Project Flow** (skip main menu).
5. If registry list has entries → show **Main Menu**.

---

## Main Menu

Display rules:
- Show Active and Blocked groups by default (skip group header if that group is empty).
- Numbers are global across groups (no per-group restart).
- Sort within each group by Last session date, most recent first.
- Completed and Archived shown as collapsed count + option letter at the bottom.

```
Active:
1. RetailEdge (FreshMart) — PWA — ARCH_PROPOSER complete — 2026-06-04
2. SupplySync (LogiCo) — Web — MVP_SYNTHESIZER complete — 2026-05-30

Blocked:
3. VectorSeven (VectorSeven Inc) — Enterprise — DISCOVERY complete — 2026-05-22

(C) completed (4)   (A) archived (1)   (N) new project
```

If consultant types C or A: display that group's entries with numbers continuing
from the last shown number. Resume and transition flow identical to active/blocked.

If Active and Blocked are both empty: skip menu, go directly to New Project Flow.

---

## New Project Flow

Ask one at a time:

```
Client name?
```
```
Engagement / project name?
```
```
Project type? (web / mobile / PWA / enterprise / hybrid)
```

Slug rule: take engagement name, lowercase, spaces → hyphens, remove special characters.
Examples: "RetailEdge" → `retailedge` | "Vector Seven" → `vector-seven` | "StyleMart" → `stylemart`

Show confirmation before any filesystem action:
```
About to create:
  ~/fiftyfive-engagements/active/retailedge/
  ~/fiftyfive-engagements/active/retailedge/input/
  ~/fiftyfive-engagements/active/retailedge/project.md
  ~/fiftyfive-engagements/active/retailedge/session_state.md

Confirm? (yes / no)
```

On "no" or any negative response:
```
Cancelled — no folders created.
```

On "yes":
1. Run via Bash: `mkdir -p ~/fiftyfive-engagements/active/<slug>/input`
2. Write `~/fiftyfive-engagements/<slug>/project.md`:

```markdown
# Project — <engagement name>
Client: <client name>
Type: <type>
Started: <YYYY-MM-DD>
Status: active
Stage: not started
Last session: <YYYY-MM-DD>
```

3. Write `~/fiftyfive-engagements/<slug>/session_state.md`:

```markdown
# Session State — <engagement name>
# Updated: <YYYY-MM-DD>

## Current Stage
Last completed: none
Status: not started

## Next Step
Run: DISCOVERY
From: ~/fiftyfive-engagements/active/<slug>/

## Open Items
(none)

## Notes
```

Output:
```
Created: ~/fiftyfive-engagements/active/<slug>/

Drop raw engagement docs in ~/fiftyfive-engagements/active/<slug>/input/
Then open Claude Code in ~/fiftyfive-engagements/active/<slug>/ and say: run DISCOVERY
```

---

## Resume Flow

On consultant selecting a number from the main menu:

1. Read `session_state.md` from the selected engagement folder.
2. Output:

```
<Engagement Name> — <Stage>
Open Claude Code in ~/fiftyfive-engagements/<slug>/ and run <Next Step>.

Open items from last session:
- <item 1>
- <item 2>
```
If Open Items is empty: show "None."

---

## Resume Single Engagement

If pre-flight found `project.md` in current directory:

1. Read `project.md` — get engagement name and stage.
2. Read `session_state.md` if present.
3. Output same format as Resume Flow above.
4. If `session_state.md` missing: show stage from project.md + next skill suggestion.

Next skill by stage:
- `not started` → DISCOVERY
- `DISCOVERY complete` → MVP_SYNTHESIZER
- `MVP_SYNTHESIZER complete` → ARCH_PROPOSER
- `ARCH_PROPOSER complete` → DOC_GENERATOR (not yet built — V1.3.5)
- `DOC_GENERATOR complete` → BACKLOG_GENERATOR
- `BACKLOG_GENERATOR complete` → ESTIMATOR
- `ESTIMATOR complete` → (chain complete — ROADMAP in future)

---

## Stage Auto-detection

If a subdirectory has no `project.md` but contains skill output files:

Detect stage from files present (check in order — stop at first match):
- `estimates.md` present → `ESTIMATOR complete`
- `backlog.md` present → `BACKLOG_GENERATOR complete`
- `arch.md` present → `ARCH_PROPOSER complete`
- `mvp-scope.md` present → `MVP_SYNTHESIZER complete`
- `discovery.md` present → `DISCOVERY complete`
- None of the above → `not started`

Ask consultant before writing project.md retroactively:
```
Found existing engagement folder: supplysync/
Stage detected: MVP_SYNTHESIZER complete (mvp-scope.md present)

Client name for this engagement?
Project type? (web / mobile / PWA / enterprise / hybrid)
```

Show confirmation with full path, then write `project.md` with confirmed details + detected stage + today's date for Last session. The retroactive project.md must include `Status: active` between Type and Stage fields.

---

## Status Transitions

After showing resume output for any engagement (from Main Menu selection or Resume Single
Engagement), append a status prompt:

**If stage = `ESTIMATOR complete` and current status = `active`:**
```
Chain complete — all skills done.
Status: active  →  (C) mark completed  /  skip
```

**For all other stages:**
```
Status: <current-status>  →  (B) blocked  /  (C) completed  /  (A) archived  /  skip
```

On B / C / A selection:

1. Determine new bucket: B → blocked | C → completed | A → archived.
2. Show confirmation:
   ```
   Move ~/fiftyfive-engagements/<old-status>/<slug>/
     → ~/fiftyfive-engagements/<new-status>/<slug>/
   Update project.md  Status: <old-status> → <new-status>
   Update session_state.md  From: path

   Confirm? (yes / no)
   ```
3. On "no": no changes, end turn.
4. On "yes":
   a. Run via Bash: `mkdir -p ~/fiftyfive-engagements/<new-status>/`
   b. Run via Bash: `mv ~/fiftyfive-engagements/<old-status>/<slug> ~/fiftyfive-engagements/<new-status>/<slug>`
   c. Read `~/fiftyfive-engagements/<new-status>/<slug>/project.md`.
      Replace line `Status: <old-status>` with `Status: <new-status>`. Write file back.
   d. Read `~/fiftyfive-engagements/<new-status>/<slug>/session_state.md`.
      Replace `From: ~/fiftyfive-engagements/<old-status>/<slug>/`
      with   `From: ~/fiftyfive-engagements/<new-status>/<slug>/`. Write file back.
   e. Output:
      ```
      Done. ~/fiftyfive-engagements/<new-status>/<slug>/ — Status updated to <new-status>.
      ```

**Legacy flat folders** (no bucket parent, e.g. `~/fiftyfive-engagements/retailedge/`):
Show `Status: active (legacy path)  →  (B) blocked  /  (C) completed  /  (A) archived  /  skip`
On selection, confirm move to `~/fiftyfive-engagements/<new-status>/<slug>/` and execute as above.
On skip: leave flat, no migration.

---

## Rules for this skill

1. Never create folders or files without explicit human confirmation — show full paths first.
2. Never modify discovery.md, mvp-scope.md, arch.md, or any skill output file.
3. Slug: lowercase, hyphens only, no special characters, derived from engagement name.
4. If no engagement folders found: skip main menu, go directly to new project flow.
5. Stage auto-detection runs if project.md missing — never silently assume stage.
6. LAYER_0_GLOBAL Rule 4 output limits and Rule 5 (no narration) apply.
