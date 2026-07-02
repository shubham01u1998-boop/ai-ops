# START V1.3 — Status Folders Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend the START skill to organize engagements into `active/`, `blocked/`, `completed/`, `archived/` status bucket subfolders, with grouped menu display and managed folder-move transitions.

**Architecture:** Add `Status:` field to `project.md`; pre-flight scans known status buckets 2 levels deep and falls back to flat folders for legacy compat; new projects created inside `active/`; after resume, offer status change — START executes the `mv`, updates `project.md` Status field and `session_state.md` From: path. Flat legacy folders (no bucket parent) are treated as `active` silently.

**Tech Stack:** Markdown skill file (`START.md`), Bash for `mkdir`/`mv`, Read/Write for project.md + session_state.md field updates.

---

## File Map

| File | Action | What changes |
|---|---|---|
| `skills/project-initiator/START.md` | Modify | Pre-flight scan, Main Menu, New Project Flow, add Status Transition section |
| `tests/fixtures/start/synthetic-new/expected_behaviors.md` | Modify | Update paths to `active/`, add Status field to template checks |
| `tests/fixtures/start/synthetic-resume/active/retailedge/project.md` | Create (moved) | Add `Status: active`, update From: path |
| `tests/fixtures/start/synthetic-resume/active/retailedge/session_state.md` | Create (moved) | Update From: path to include `active/` |
| `tests/fixtures/start/synthetic-resume/active/supplysync/project.md` | Create (moved) | Add `Status: active` |
| `tests/fixtures/start/synthetic-resume/active/supplysync/session_state.md` | Create (moved) | Update From: path |
| `tests/fixtures/start/synthetic-resume/active/vectorseven/project.md` | Create (moved) | Add `Status: active` |
| `tests/fixtures/start/synthetic-resume/active/vectorseven/session_state.md` | Create (moved) | Update From: path |
| `tests/fixtures/start/synthetic-resume/expected_behaviors.md` | Modify | Grouped menu format, 2-level scan behavior |
| `tests/fixtures/start/synthetic-resume/retailedge/` | Delete | Replaced by active/ bucket |
| `tests/fixtures/start/synthetic-resume/supplysync/` | Delete | Replaced by active/ bucket |
| `tests/fixtures/start/synthetic-resume/vectorseven/` | Delete | Replaced by active/ bucket |
| `tests/fixtures/start/synthetic-transition/` | Create new fixture | New fixture for status-change flow |
| `tests/fixtures/start/synthetic-transition/active/decorconnect/project.md` | Create | ESTIMATOR complete engagement for transition test |
| `tests/fixtures/start/synthetic-transition/active/decorconnect/session_state.md` | Create | ESTIMATOR complete session state |
| `tests/fixtures/start/synthetic-transition/expected_behaviors.md` | Create | Transition flow expected behaviors |

> **Note:** Original flat fixture files (`synthetic-resume/retailedge/`, `synthetic-resume/supplysync/`, `synthetic-resume/vectorseven/`) are replaced by the `active/` bucket structure. Delete the old flat files after creating the new bucketed ones.

---

## Task 1: Update `project.md` template in START.md

**Files:**
- Modify: `skills/project-initiator/START.md` (New Project Flow section + Stage Auto-detection section)

- [ ] **Step 1: Replace the project.md template block in New Project Flow**

Find this block (under "2. Write `~/fiftyfive-engagements/<slug>/project.md`"):

```
# Project — <engagement name>
Client: <client name>
Type: <type>
Started: <YYYY-MM-DD>
Stage: not started
Last session: <YYYY-MM-DD>
```

Replace with:

```
# Project — <engagement name>
Client: <client name>
Type: <type>
Started: <YYYY-MM-DD>
Status: active
Stage: not started
Last session: <YYYY-MM-DD>
```

- [ ] **Step 2: Add Status field to Stage Auto-detection retroactive project.md**

In the Stage Auto-detection section, the retroactive project.md written after consultant confirms client name + type should also include `Status: active`. Find the implied write and ensure it produces:

```
# Project — <engagement name>
Client: <confirmed client name>
Type: <confirmed type>
Started: <today: YYYY-MM-DD>
Status: active
Stage: <detected stage>
Last session: <today: YYYY-MM-DD>
```

- [ ] **Step 3: Commit**

```bash
git add skills/project-initiator/START.md
git commit -m "feat(start): add Status field to project.md template"
```

---

## Task 2: Update Pre-flight scan in START.md

**Files:**
- Modify: `skills/project-initiator/START.md` (Pre-flight section — step 3 only)

- [ ] **Step 1: Replace step 3 of Pre-flight**

Find:

```
3. Scan for engagement folders: run `ls -d */` via Bash (ignore errors if no subdirs).
   - For each subdirectory found, attempt Read on `<subdir>/project.md`.
   - Build registry list in memory from all successfully read project.md files.
```

Replace with:

```
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
```

- [ ] **Step 2: Commit**

```bash
git add skills/project-initiator/START.md
git commit -m "feat(start): update pre-flight scan to detect status bucket folders"
```

---

## Task 3: Update Main Menu in START.md

**Files:**
- Modify: `skills/project-initiator/START.md` (Main Menu section)

- [ ] **Step 1: Replace the Main Menu section entirely**

Find (the whole block between `## Main Menu` and the next `---`):

```
```
Active engagements:
1. RetailEdge (FreshMart) — PWA — ARCH_PROPOSER complete — 2026-06-04
2. SupplySync (LogiCo) — Web — MVP_SYNTHESIZER complete — 2026-05-30
3. VectorSeven (VectorSeven Inc) — Enterprise — DISCOVERY complete — 2026-05-22

Which? (1 / 2 / 3) or N for new project
```

Sort by Last session date, most recent first.
```

Replace with:

```
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
```

- [ ] **Step 2: Commit**

```bash
git add skills/project-initiator/START.md
git commit -m "feat(start): group main menu by status with collapsed completed/archived"
```

---

## Task 4: Update New Project Flow folder paths in START.md

**Files:**
- Modify: `skills/project-initiator/START.md` (New Project Flow section — 4 places)

- [ ] **Step 1: Update the confirmation block paths**

Find:

```
About to create:
  ~/fiftyfive-engagements/retailedge/
  ~/fiftyfive-engagements/retailedge/input/
  ~/fiftyfive-engagements/retailedge/project.md
  ~/fiftyfive-engagements/retailedge/session_state.md
```

Replace with:

```
About to create:
  ~/fiftyfive-engagements/active/retailedge/
  ~/fiftyfive-engagements/active/retailedge/input/
  ~/fiftyfive-engagements/active/retailedge/project.md
  ~/fiftyfive-engagements/active/retailedge/session_state.md
```

- [ ] **Step 2: Update the mkdir command**

Find:

```
1. Run via Bash: `mkdir -p ~/fiftyfive-engagements/<slug>/input`
```

Replace with:

```
1. Run via Bash: `mkdir -p ~/fiftyfive-engagements/active/<slug>/input`
```

- [ ] **Step 3: Update session_state.md From: path in the template**

In the session_state.md template block (step 3 of New Project Flow), find:

```
From: ~/fiftyfive-engagements/<slug>/
```

Replace with:

```
From: ~/fiftyfive-engagements/active/<slug>/
```

- [ ] **Step 4: Update the handoff output**

Find:

```
Created: ~/fiftyfive-engagements/<slug>/

Drop raw engagement docs in ~/fiftyfive-engagements/<slug>/input/
Then open Claude Code in ~/fiftyfive-engagements/<slug>/ and say: run DISCOVERY
```

Replace with:

```
Created: ~/fiftyfive-engagements/active/<slug>/

Drop raw engagement docs in ~/fiftyfive-engagements/active/<slug>/input/
Then open Claude Code in ~/fiftyfive-engagements/active/<slug>/ and say: run DISCOVERY
```

- [ ] **Step 5: Commit**

```bash
git add skills/project-initiator/START.md
git commit -m "feat(start): create new projects inside active/ bucket folder"
```

---

## Task 5: Add Status Transition section to START.md

**Files:**
- Modify: `skills/project-initiator/START.md` (add new section before `## Rules for this skill`)

- [ ] **Step 1: Insert new section**

Find the line `## Rules for this skill` and insert the following section immediately before it (after the `---` separator above Rules):

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add skills/project-initiator/START.md
git commit -m "feat(start): add status transition section with mv + file update logic"
```

---

## Task 6: Update `synthetic-new` fixture

**Files:**
- Modify: `tests/fixtures/start/synthetic-new/expected_behaviors.md`

- [ ] **Step 1: Update Confirmation Gate paths**

Find:

```
- [ ] Shows full folder paths before creating anything:
      ~/fiftyfive-engagements/retailedge/
      ~/fiftyfive-engagements/retailedge/input/
      ~/fiftyfive-engagements/retailedge/project.md
      ~/fiftyfive-engagements/retailedge/session_state.md
```

Replace with:

```
- [ ] Shows full folder paths before creating anything:
      ~/fiftyfive-engagements/active/retailedge/
      ~/fiftyfive-engagements/active/retailedge/input/
      ~/fiftyfive-engagements/active/retailedge/project.md
      ~/fiftyfive-engagements/active/retailedge/session_state.md
```

- [ ] **Step 2: Update project.md content check**

Find:

```
- [ ] project.md written with correct content:
      # Project — RetailEdge
      Client: FreshMart
      Type: PWA
      Started: <today: YYYY-MM-DD>
      Stage: not started
      Last session: <today: YYYY-MM-DD>
```

Replace with:

```
- [ ] project.md written with correct content:
      # Project — RetailEdge
      Client: FreshMart
      Type: PWA
      Started: <today: YYYY-MM-DD>
      Status: active
      Stage: not started
      Last session: <today: YYYY-MM-DD>
```

- [ ] **Step 3: Update session_state.md From: check**

Find:

```
      From: ~/fiftyfive-engagements/retailedge/
```

Replace with:

```
      From: ~/fiftyfive-engagements/active/retailedge/
```

- [ ] **Step 4: Update Handoff Output checks**

Find:

```
- [ ] Outputs path of created folder
- [ ] Tells consultant to drop docs in input/ subfolder
- [ ] Tells consultant to open Claude Code in retailedge/ and say "run DISCOVERY"
```

Replace with:

```
- [ ] Outputs path ~/fiftyfive-engagements/active/retailedge/
- [ ] Tells consultant to drop docs in ~/fiftyfive-engagements/active/retailedge/input/
- [ ] Tells consultant to open Claude Code in ~/fiftyfive-engagements/active/retailedge/ and say "run DISCOVERY"
```

- [ ] **Step 5: Commit**

```bash
git add tests/fixtures/start/synthetic-new/expected_behaviors.md
git commit -m "test(start): update synthetic-new fixture for active/ bucket paths"
```

---

## Task 7: Migrate `synthetic-resume` fixture to bucket structure

**Files:**
- Create: `tests/fixtures/start/synthetic-resume/active/retailedge/project.md`
- Create: `tests/fixtures/start/synthetic-resume/active/retailedge/session_state.md`
- Create: `tests/fixtures/start/synthetic-resume/active/supplysync/project.md`
- Create: `tests/fixtures/start/synthetic-resume/active/supplysync/session_state.md`
- Create: `tests/fixtures/start/synthetic-resume/active/vectorseven/project.md`
- Create: `tests/fixtures/start/synthetic-resume/active/vectorseven/session_state.md`
- Modify: `tests/fixtures/start/synthetic-resume/expected_behaviors.md`
- Delete: `tests/fixtures/start/synthetic-resume/retailedge/` (flat)
- Delete: `tests/fixtures/start/synthetic-resume/supplysync/` (flat)
- Delete: `tests/fixtures/start/synthetic-resume/vectorseven/` (flat)

- [ ] **Step 1: Create active/retailedge/project.md**

```markdown
# Project — RetailEdge
Client: FreshMart
Type: PWA
Started: 2026-05-10
Status: active
Stage: ARCH_PROPOSER complete
Last session: 2026-06-04
```

- [ ] **Step 2: Create active/retailedge/session_state.md**

```markdown
# Session State — RetailEdge
# Updated: 2026-06-04

## Current Stage
Last completed: ARCH_PROPOSER
Status: complete

## Next Step
Run: DOC_GENERATOR
From: ~/fiftyfive-engagements/active/retailedge/

## Open Items
- Oracle Retail POS integration method unconfirmed — source: ARCH_PROPOSER — blocks: OracleConnector

## Notes
```

- [ ] **Step 3: Create active/supplysync/project.md**

```markdown
# Project — SupplySync
Client: LogiCo
Type: Web
Started: 2026-05-01
Status: active
Stage: MVP_SYNTHESIZER complete
Last session: 2026-05-30
```

- [ ] **Step 4: Create active/supplysync/session_state.md**

```markdown
# Session State — SupplySync
# Updated: 2026-05-30

## Current Stage
Last completed: MVP_SYNTHESIZER
Status: complete

## Next Step
Run: ARCH_PROPOSER
From: ~/fiftyfive-engagements/active/supplysync/

## Open Items
(none)

## Notes
```

- [ ] **Step 5: Create active/vectorseven/project.md**

```markdown
# Project — VectorSeven
Client: VectorSeven Inc
Type: Enterprise
Started: 2026-05-15
Status: active
Stage: DISCOVERY complete
Last session: 2026-05-22
```

- [ ] **Step 6: Create active/vectorseven/session_state.md**

```markdown
# Session State — VectorSeven
# Updated: 2026-05-22

## Current Stage
Last completed: DISCOVERY
Status: complete

## Next Step
Run: MVP_SYNTHESIZER
From: ~/fiftyfive-engagements/active/vectorseven/

## Open Items
(none)

## Notes
```

- [ ] **Step 7: Delete old flat fixture folders**

```bash
rm -rf tests/fixtures/start/synthetic-resume/retailedge
rm -rf tests/fixtures/start/synthetic-resume/supplysync
rm -rf tests/fixtures/start/synthetic-resume/vectorseven
```

- [ ] **Step 8: Replace expected_behaviors.md**

Write the full file:

```markdown
# Expected Behaviors — synthetic-resume (Resume / Registry path)
# Tests START skill reading 3 mock engagement folders in active/ bucket and routing correctly.
# Mock engagements: RetailEdge (ARCH_PROPOSER complete), SupplySync (MVP_SYNTHESIZER complete),
# VectorSeven (DISCOVERY complete) — all in active/ bucket.

## Pre-flight
- [ ] Scans subdirectories — finds `active/` as a known status bucket
- [ ] Scans inside `active/` for engagement folders
- [ ] Reads all 3 project.md files from active/retailedge/, active/supplysync/, active/vectorseven/
- [ ] Builds registry in memory from all 3 with status: active

## Registry Display
- [ ] Shows Active group header with all 3 engagements
- [ ] No Blocked group shown (no blocked engagements in fixture)
- [ ] Each entry shows: name, client, type, stage, last session date
- [ ] RetailEdge shown as: ARCH_PROPOSER complete — 2026-06-04
- [ ] SupplySync shown as: MVP_SYNTHESIZER complete — 2026-05-30
- [ ] VectorSeven shown as: DISCOVERY complete — 2026-05-22
- [ ] Most recent first within Active group: RetailEdge at top, VectorSeven at bottom
- [ ] Completed and Archived shown as collapsed counts: (C) completed (0)  (A) archived (0)
- [ ] "N for new project" option shown

## Resume Selection
- [ ] Consultant picks "1" (RetailEdge)
- [ ] Reads active/retailedge/session_state.md
- [ ] Shows: "RetailEdge — ARCH_PROPOSER complete"
- [ ] Shows next step: "run DOC_GENERATOR"
- [ ] Shows folder path: ~/fiftyfive-engagements/active/retailedge/
- [ ] Shows open items: "Oracle Retail POS integration method unconfirmed — source: ARCH_PROPOSER — blocks: OracleConnector"
- [ ] Shows status transition prompt: "Status: active → (B) blocked / (C) completed / (A) archived / skip"

## Stage Auto-detection (supplementary check)
- [ ] If a subdirectory inside active/ has project.md missing but arch.md present → detects ARCH_PROPOSER complete
- [ ] If mvp-scope.md present (no arch.md) inside active/ → detects MVP_SYNTHESIZER complete
- [ ] Asks consultant to confirm client name + type before writing retroactive project.md
- [ ] Shows confirmation before writing retroactive project.md
- [ ] Retroactive project.md includes Status: active field

## Backward Compat (supplementary check)
- [ ] If a flat folder (no status bucket parent) is found alongside status buckets,
      it is read and shown in the Active group with "(legacy path)" note on status line
- [ ] No automatic migration of legacy folder is triggered

## Test Notes
- Run this simulation by presenting the 3 mock project.md and session_state.md files from
  the active/ bucket to the START skill as if scanned from the parent directory.
- The subagent should verify: 2-level scan behavior, grouped menu format (Active/Blocked),
  collapsed completed/archived, sort order, routing output, and status transition prompt.
- Stage auto-detection and backward compat items are supplementary — simulate an extra
  folder to test each case.
```

- [ ] **Step 9: Commit**

```bash
git add tests/fixtures/start/synthetic-resume/
git commit -m "test(start): migrate synthetic-resume fixture to active/ bucket structure"
```

---

## Task 8: Create `synthetic-transition` fixture

**Files:**
- Create: `tests/fixtures/start/synthetic-transition/active/decorconnect/project.md`
- Create: `tests/fixtures/start/synthetic-transition/active/decorconnect/session_state.md`
- Create: `tests/fixtures/start/synthetic-transition/expected_behaviors.md`

- [ ] **Step 1: Create active/decorconnect/project.md**

```markdown
# Project — DecorConnect
Client: HomeCo
Type: Web
Started: 2026-05-20
Status: active
Stage: ESTIMATOR complete
Last session: 2026-06-08
```

- [ ] **Step 2: Create active/decorconnect/session_state.md**

```markdown
# Session State — DecorConnect
# Updated: 2026-06-08

## Current Stage
Last completed: ESTIMATOR
Status: complete

## Next Step
Run: (chain complete — ROADMAP in future)
From: ~/fiftyfive-engagements/active/decorconnect/

## Open Items
(none)

## Notes
```

- [ ] **Step 3: Create expected_behaviors.md**

```markdown
# Expected Behaviors — synthetic-transition (Status Transition path)
# Tests START skill status transition: chain-complete auto-suggest + move execution.
# Mock engagement: DecorConnect (ESTIMATOR complete) in active/ bucket.
# Simulated consultant answers: select DecorConnect (1), then C (completed), then yes.

## Pre-flight
- [ ] Finds active/ bucket, reads active/decorconnect/project.md
- [ ] Builds registry with 1 engagement: DecorConnect, status: active

## Registry Display
- [ ] Shows DecorConnect in Active group: ESTIMATOR complete — 2026-06-08
- [ ] Numbers: 1. DecorConnect

## Resume Output
- [ ] Consultant picks "1" (DecorConnect)
- [ ] Shows: "DecorConnect — ESTIMATOR complete"
- [ ] Shows: "Chain complete — all skills done."
- [ ] Shows chain-complete status prompt (only C and skip offered):
      "Status: active  →  (C) mark completed  /  skip"
- [ ] Does NOT offer (B) blocked or (A) archived in the chain-complete prompt

## Transition — Consultant selects C
- [ ] Shows confirmation block:
      Move ~/fiftyfive-engagements/active/decorconnect/
        → ~/fiftyfive-engagements/completed/decorconnect/
      Update project.md  Status: active → completed
      Update session_state.md  From: path
      Confirm? (yes / no)
- [ ] Does NOT move any files before confirmation

## Transition — Consultant confirms yes
- [ ] Runs: mkdir -p ~/fiftyfive-engagements/completed/
- [ ] Runs: mv ~/fiftyfive-engagements/active/decorconnect ~/fiftyfive-engagements/completed/decorconnect
- [ ] Reads project.md after move and updates Status: active → completed
- [ ] Reads session_state.md after move and updates From: path (active → completed)
- [ ] Outputs: "Done. ~/fiftyfive-engagements/completed/decorconnect/ — Status updated to completed."

## Transition — Consultant selects skip
- [ ] No move, no file changes
- [ ] No output beyond the resume display

## Manual status change (supplementary — non-chain-complete engagement)
- [ ] For an engagement NOT at ESTIMATOR complete, full 4-option prompt shown:
      "Status: active  →  (B) blocked  /  (C) completed  /  (A) archived  /  skip"
- [ ] Selecting B shows correct confirmation with blocked/ destination
- [ ] Selecting A shows correct confirmation with archived/ destination

## Legacy flat folder transition (supplementary)
- [ ] A flat folder (no bucket parent) shows: "Status: active (legacy path)"
- [ ] Selecting a status change for a legacy folder moves it into the correct bucket
- [ ] After move, project.md Status field written (adds field if not present)

## Test Notes
- Simulate by presenting DecorConnect files to START as if found in the active/ bucket.
- For move steps: verify INTENDED bash commands shown in confirmation — not actual filesystem ops.
- Supplementary checks: simulate additional folder states inline to test each path.
```

- [ ] **Step 4: Commit**

```bash
git add tests/fixtures/start/synthetic-transition/
git commit -m "test(start): add synthetic-transition fixture for status change flow"
```

---

## Self-Review

### Spec coverage

| Requirement | Task |
|---|---|
| `Status: active` field in project.md | Task 1 |
| Status in retroactive project.md (auto-detect) | Task 1 |
| 2-level bucket scan | Task 2 |
| New projects created in `active/` | Task 4 |
| Main menu grouped by status | Task 3 |
| Completed/archived collapsed with count | Task 3 |
| Status transition after resume | Task 5 |
| mv + project.md update + session_state update | Task 5 |
| Auto-suggest completed when ESTIMATOR complete | Task 5 |
| Legacy flat folders treated as active | Task 2 + Task 5 |
| synthetic-new fixture updated | Task 6 |
| synthetic-resume migrated to bucket structure | Task 7 |
| synthetic-transition fixture created | Task 8 |

All requirements covered. No gaps.
