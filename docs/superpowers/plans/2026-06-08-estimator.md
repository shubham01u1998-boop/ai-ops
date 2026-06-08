# ESTIMATOR V1.5 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the ESTIMATOR V1.5 skill and patch BACKLOG_GENERATOR to V1.4.1, so the chain writes backlog.md before Odoo creation, optionally runs estimation, and always gives the consultant an explicit Odoo gate.

**Architecture:** Two deliverables — (1) a small patch to `BACKLOG_GENERATOR.md` that separates file writing from Odoo creation and adds a 3-option gate; (2) a new `ESTIMATOR.md` skill file that reads arch.md + backlog.md, converts effort signals to hours/days, optionally computes costs, and writes `estimates.md`. Both are Markdown skill files, not code. "Tests" = running the skill against synthetic fixtures and scoring against `expected_behaviors.md`.

**Tech Stack:** Markdown skill files, Bash tool (basename), Read/Write tools, MCP odoo tools (referenced in Odoo gate only)

---

## Files

| Action | Path | Purpose |
|---|---|---|
| Modify | `skills/project-initiator/BACKLOG_GENERATOR.md` | Patch Preview gate + Write backlog.md + add Next Steps Gate |
| Create | `skills/project-initiator/ESTIMATOR.md` | New skill — full estimation flow |
| Modify | `skills/project-initiator/START.md` | Add ESTIMATOR to stage routing + auto-detection |
| Modify | `CLAUDE.md` | Add V1.5 entry, update chain status |
| Create | `tests/fixtures/estimator/synthetic-01/arch.md` | RetailEdge arch fixture (PASS path) |
| Create | `tests/fixtures/estimator/synthetic-01/backlog.md` | RetailEdge backlog fixture (PASS path) |
| Create | `tests/fixtures/estimator/synthetic-01/expected_behaviors.md` | Fixed hours, no cost, confirmed dates |
| Create | `tests/fixtures/estimator/synthetic-02-range/arch.md` | RetailEdge arch fixture (RANGE path) |
| Create | `tests/fixtures/estimator/synthetic-02-range/backlog.md` | RetailEdge backlog fixture (RANGE path) |
| Create | `tests/fixtures/estimator/synthetic-02-range/expected_behaviors.md` | Range, cost, date override, row adjustment |
| Create | `tests/fixtures/estimator/TEST_SCRIPT.md` | How to run both fixtures |

---

## Task 1: BACKLOG_GENERATOR V1.4.1 — patch Preview + Approval Gate

**Files:**
- Modify: `skills/project-initiator/BACKLOG_GENERATOR.md`

- [ ] **Step 1: Open the file and locate the Preview + Approval Gate section**

  Read `skills/project-initiator/BACKLOG_GENERATOR.md`. Find the block starting with:
  ```
  Create these <N> tickets?
  [A] Yes, create in Odoo now
  ```
  This is in the `## Preview + Approval Gate` section (~line 395).

- [ ] **Step 2: Replace the gate prompt options**

  Find and replace this exact block:
  ```
  Create these <N> tickets?
  [A] Yes, create in Odoo now
  [B] Save to backlog.md only — push to Odoo later
  [C] Edit
  [D] Cancel — do not save
  ```

  Replace with:
  ```
  Approve this ticket set?
  [A] Yes — looks good
  [B] Edit
  [C] Cancel — do not save
  ```

- [ ] **Step 3: Replace the natural-language handling block**

  Find and replace this exact block (including the trailing `Per LAYER_0_GLOBAL` line):
  ```
  **Natural-language handling:**
  - Clear approval ("yes", "create them", "looks good") → [A]
  - "save", "file only", "not now", "mcp is down" → [B]
  - Edit request ("change sprint 5 ticket assignee to Tanu") → [C], apply changes, re-show the full preview table, then re-ask. Repeat until [A], [B], or [D].
  - "cancel" or "stop" → [D], output `Cancelled. No tickets created.` and stop.

  **On [A] or [B]:** write `backlog.md` first (see Write backlog.md section), then proceed accordingly.

  Per LAYER_0_GLOBAL Rule 1: this prompt is the permission gate. Do not write to Odoo
  until the consultant explicitly approves.
  ```

  Replace with:
  ```
  **Natural-language handling:**
  - "yes", "looks good", "approve", "create them" → [A]: write `backlog.md` (DRAFT), then show Next Steps Gate.
  - Edit request ("change sprint 5 ticket assignee to Tanu") → [B]: apply changes, re-show the full preview table, then re-ask. Repeat until [A] or [C].
  - "cancel", "stop" → [C]: output `Cancelled. No tickets or files created.` and stop.

  Per LAYER_0_GLOBAL Rule 1: this prompt is the permission gate. Do not write to Odoo
  until the consultant explicitly approves via the Next Steps Gate.
  ```

- [ ] **Step 4: Replace the backlog.md write logic at the end of `## Write backlog.md`**

  Find and replace this exact block:
  ```
  - If [B] chosen: set `Status: DRAFT — pending Odoo creation`. Output:
    ```
    backlog.md saved.
    To push to Odoo later: reconnect MCP and run BACKLOG_GENERATOR — it will detect backlog.md and offer push mode.
    ```
    Then stop. Do not call any MCP tools.
  - If [A] chosen: set `Status: DRAFT — pending Odoo creation`. Proceed to Bulk Creation. After all MCP calls succeed, update status line to `Status: CREATED — tickets in Odoo (project ID: <id>)`.
  ```

  Replace with:
  ```
  Write `backlog.md` immediately after preview is approved ([A] at the Preview gate).
  Always set `Status: DRAFT — pending Odoo creation` at write time.

  Output:
  ```
  backlog.md written.
  ```

  Then show the **Next Steps Gate** (see next section).
  After all Bulk Creation MCP calls succeed (if [A] chosen at Next Steps Gate), update the
  status line in `backlog.md` to `Status: CREATED — tickets in Odoo (project ID: <id>)`.
  ```

- [ ] **Step 5: Add the Next Steps Gate section after `## Write backlog.md`**

  Append the following new section immediately after the `## Write backlog.md` section
  (before `## Push Mode`):

  ```markdown
  ---

  ## Next Steps Gate

  Shown immediately after `backlog.md` is written. Never shown before.

  ```
  backlog.md written. What's next?
  [A] Create tickets in Odoo now
  [B] Run ESTIMATOR first — generate estimates.md, then decide on Odoo
  [C] Skip — keep backlog.md as draft, push to Odoo manually later
  ```

  - [A]: proceed to Bulk Creation (next section).
  - [B]: output:
    ```
    Run ESTIMATOR from this folder: say "run ESTIMATOR"
    ESTIMATOR will ask about Odoo creation when it finishes.
    ```
    Then stop.
  - [C]: output:
    ```
    backlog.md saved as DRAFT.
    To push to Odoo later: run BACKLOG_GENERATOR — it will detect backlog.md and offer push mode.
    ```
    Then stop.

  Natural-language handling:
  - "create", "odoo now", "yes", "A" → [A]
  - "estimator", "estimate first", "B" → [B]
  - "skip", "later", "not now", "C" → [C]
  ```

- [ ] **Step 6: Update the version header**

  Find:
  ```
  # VERSION: 1.4 | Last updated: 2026-06-04 | Reviewed: pending
  ```
  Replace with:
  ```
  # VERSION: 1.4.1 | Last updated: 2026-06-08 | Reviewed: pending
  ```

- [ ] **Step 7: Commit**

  ```bash
  git add skills/project-initiator/BACKLOG_GENERATOR.md
  git commit -m "feat(backlog-generator): V1.4.1 — write backlog.md first, add 3-option Next Steps Gate"
  ```

---

## Task 2: Test fixture inputs

**Files:**
- Create: `tests/fixtures/estimator/synthetic-01/arch.md`
- Create: `tests/fixtures/estimator/synthetic-01/backlog.md`
- Create: `tests/fixtures/estimator/synthetic-02-range/arch.md`
- Create: `tests/fixtures/estimator/synthetic-02-range/backlog.md`

Both fixture paths share the same arch.md content (RetailEdge). The backlog.md differs only in Status line.

- [ ] **Step 1: Write arch.md for synthetic-01**

  Create `tests/fixtures/estimator/synthetic-01/arch.md` with the following content
  (identical to `tests/fixtures/backlog-generator/synthetic-01/arch.md` — copy it verbatim):

  ```markdown
  # Architecture — RetailEdge
  # Generated: 2026-06-04 | Reviewed by: Shubham Upadhyay
  # Source: mvp-scope.md (Value-first framing, 2 features in scope)

  ## Client Summary
  RetailEdge will give FreshMart store managers a unified operational platform covering daily
  sales visibility and real-time inventory alerts across all 12 stores. The build runs in two
  parallel tracks — a data integration layer first to connect the existing point-of-sale system,
  followed by the dashboard and alerting surfaces that store managers will use day to day. The
  biggest business risk is the POS integration: the technical method has not yet been confirmed
  and could extend the timeline if the system does not expose a direct connection. The platform
  is designed for web access from any store, with offline support for locations with poor
  connectivity, targeting go-live by 2026-09-04.

  ## Tech Stack
  | Layer | Decision | Status |
  |---|---|---|
  | Frontend | React | ✓ Confirmed |
  | Backend | Node.js + Express | ✓ Confirmed |
  | Database | PostgreSQL | [STRAWMAN] — Azure SQL viable if Oracle POS requires it |
  | Infra | Azure App Service | ✓ Confirmed |
  | Cloud | Azure | ✓ Confirmed |

  ## Components
  ### Sales Dashboard
  - DashboardUI (React) — store/date/category filters + chart views
  - SalesAPI (Node.js) — /sales/summary, /sales/by-store, /sales/by-category
  - SalesAggregator (Node.js) — scheduled job: aggregates raw POS data → summary tables
  - OracleConnector (Node.js) — reads from Oracle Retail POS **(shared)**

  ### Inventory Alerts
  - AlertsUI (React) — low-stock/expiry alert feed per store
  - AlertsAPI (Node.js) — /alerts/active, /alerts/dismiss
  - AlertEngine (Node.js) — scheduled job: evaluates SKU thresholds → writes alert records
  - OracleConnector — reused (shared)

  **Shared:** OracleConnector — used by [Sales Dashboard, Inventory Alerts]. Built once in Sprint 1 before either feature's API layer.

  ## Data Model Hints
  ### Sales Dashboard
  - `sales_summary` — store_id, date, category_id, total_sales, units_sold

  ### Inventory Alerts
  - `inventory_alerts` — sku_id, store_id, alert_type (low_stock|expiry), triggered_at, dismissed_at

  ### Shared
  - `oracle_sync_log` — sync_id, synced_at, status, records_processed

  ## Integration Points
  | System | Approach | Risk | Open Questions |
  |---|---|---|---|
  | Oracle Retail POS | Method TBD — deferred | HIGH [STRAWMAN] | API available? DB export? Auth method? |
  | Offline mode | Service Worker + IndexedDB cache; sync on reconnect | MED [STRAWMAN] | Acceptable data staleness window when offline? |

  ## Build Order
  1. OracleConnector — shared; both Sales Dashboard and Inventory Alerts depend on it
  2. SalesAggregator + data model — backend aggregation layer before API
  3. SalesAPI + AlertEngine — business logic before UI surfaces
  4. DashboardUI + AlertsUI — frontend surfaces last
  5. Offline mode layer — additive on top of DashboardUI

  ## Sprint Mapping
  **Team:** 1 FE-senior (React), 1 BE-senior (Node.js), 1 BE-junior (Node.js), 1 QA
  **Timeline:** 24 working days — 6 × 2-week sprints (part-time team)
  **Project start:** 2026-06-04

  | Sprint | Work | Owner |
  |---|---|---|
  | Sprint 1-2 (D1–8) | OracleConnector | BE-senior |
  | Sprint 1-2 (D1–8) | SalesAggregator + data model | BE-junior |
  | Sprint 3-4 (D9–16) | SalesAPI + AlertEngine | BE-senior + BE-junior |
  | Sprint 5 (D17–20) | DashboardUI + AlertsUI | FE-senior |
  | Sprint 6 (D21–24) | Offline mode layer | FE-senior |

  ## Effort Signals
  | Feature | Size | Rationale |
  |---|---|---|
  | OracleConnector | L | Shared connector; integration risk; method TBD |
  | SalesAggregator + data model | M | Scheduled job + schema; moderate complexity |
  | SalesAPI + AlertEngine | L | Two API layers + business logic |
  | DashboardUI + AlertsUI | M | Two React surfaces; no backend risk |
  | Offline mode layer | S | Additive; Service Worker pattern |

  ## Open Questions
  1. Oracle Retail POS integration method not confirmed — API, DB export, or file transfer?
     Blocks: OracleConnector
  2. Acceptable data staleness window for offline mode?
     Blocks: Offline mode layer

  ## STRAWMAN Summary
  - [STRAWMAN] PostgreSQL vs Azure SQL — deferred pending Oracle POS method confirmation
  - [STRAWMAN] Oracle POS integration method — API, DB export, or file transfer; HIGH risk
  - [STRAWMAN] Offline mode staleness window — UX decision needed before Sprint 5
  ```

- [ ] **Step 2: Write backlog.md for synthetic-01**

  Create `tests/fixtures/estimator/synthetic-01/backlog.md`:

  ```markdown
  # Backlog — RetailEdge
  # Generated: 2026-06-04 | BACKLOG_GENERATOR V1.4.1
  # Status: DRAFT — pending Odoo creation

  ---

  ## Project Config

  | Field | Value |
  |---|---|
  | Project name | RetailEdge |
  | Odoo project ID | — (assigned when pushed to Odoo) |

  ### Stages

  | Stage | Semantic role |
  |---|---|
  | Backlog | Blocked items, pre-conditions, STRAWMAN tickets |
  | To Do | Newly created tickets (default for new tickets) |
  | In Progress | Active development |
  | Bug | Reserved |
  | Done | Completed |

  ---

  ## Team Mapping

  | Role | arch.md label | Odoo user ID |
  |---|---|---|
  | BE-senior | BE-senior (Node.js) | 42 |
  | BE-junior | BE-junior (Node.js) | 41 |
  | FE-senior | FE-senior (React) | 50 |
  | QA | QA | 57 |

  ---

  ## Sprint Tags

  | Sprint | Date range | Odoo tag ID |
  |---|---|---|
  | Sprint 1-2 | 2026-06-04 → 2026-07-02 | — |
  | Sprint 3-4 | 2026-07-03 → 2026-07-30 | — |
  | Sprint 5 | 2026-07-31 → 2026-08-13 | — |
  | Sprint 6 | 2026-08-14 → 2026-09-04 | — |

  ---

  ## Parent Tickets — 5

  ### T01 · [Sprint 1-2] OracleConnector

  | Field | Value |
  |---|---|
  | Sprint | Sprint 1-2 |
  | Type | Backend |
  | Stage | To Do |
  | Assignee role | BE-senior |
  | Deadline | 2026-07-02 |
  | Priority | Normal |

  ### T02 · [Sprint 1-2] SalesAggregator + data model

  | Field | Value |
  |---|---|
  | Sprint | Sprint 1-2 |
  | Type | Backend |
  | Stage | To Do |
  | Assignee role | BE-junior |
  | Deadline | 2026-07-02 |
  | Priority | Normal |

  ### T03 · [Sprint 3-4] SalesAPI + AlertEngine

  | Field | Value |
  |---|---|
  | Sprint | Sprint 3-4 |
  | Type | Backend |
  | Stage | To Do |
  | Assignee role | BE-senior |
  | Deadline | 2026-07-30 |
  | Priority | Normal |

  ### T04 · [Sprint 5] DashboardUI + AlertsUI

  | Field | Value |
  |---|---|
  | Sprint | Sprint 5 |
  | Type | Frontend |
  | Stage | To Do |
  | Assignee role | FE-senior |
  | Deadline | 2026-08-13 |
  | Priority | Normal |

  ### T05 · [Sprint 6] Offline mode layer

  | Field | Value |
  |---|---|
  | Sprint | Sprint 6 |
  | Type | Backend |
  | Stage | To Do |
  | Assignee role | FE-senior |
  | Deadline | 2026-09-04 |
  | Priority | Normal |
  ```

- [ ] **Step 3: Copy arch.md to synthetic-02-range**

  Create `tests/fixtures/estimator/synthetic-02-range/arch.md` with identical content to
  `tests/fixtures/estimator/synthetic-01/arch.md` (verbatim copy).

- [ ] **Step 4: Write backlog.md for synthetic-02-range**

  Create `tests/fixtures/estimator/synthetic-02-range/backlog.md` — identical to
  `tests/fixtures/estimator/synthetic-01/backlog.md` except the Status line:

  Change:
  ```
  # Status: DRAFT — pending Odoo creation
  ```
  to:
  ```
  # Status: DRAFT — pending Odoo creation
  ```
  (Same — this fixture also starts as DRAFT so the Odoo gate shows at the end.)

- [ ] **Step 5: Commit**

  ```bash
  git add tests/fixtures/estimator/
  git commit -m "test(estimator): add fixture inputs for synthetic-01 and synthetic-02-range"
  ```

---

## Task 3: Expected behaviors — synthetic-01 (fixed hours, no cost, PASS)

**Files:**
- Create: `tests/fixtures/estimator/synthetic-01/expected_behaviors.md`

- [ ] **Step 1: Write expected_behaviors.md**

  ```markdown
  # Expected Behaviors — ESTIMATOR synthetic-01
  # Fixture: RetailEdge PASS path — fixed hours, no cost, confirmed dates, no adjustments

  ## Pre-flight

  - [ ] EST-01: `basename "$PWD"` returns "synthetic-01" (generic) → skill asks for engagement name
  - [ ] EST-02: Consultant provides "RetailEdge" → stored and used throughout
  - [ ] EST-03: arch.md loaded; required sections present: Effort Signals, Sprint Mapping, Build Order
  - [ ] EST-04: backlog.md loaded
  - [ ] EST-05: backlog.md Status: DRAFT — no "already in Odoo" warning shown
  - [ ] EST-06: estimates.md not found — no overwrite prompt shown
  - [ ] EST-07: Project start date detected: 2026-06-04 (from Sprint Mapping "Project start:")
  - [ ] EST-08: Outputs: `Engagement: RetailEdge | arch.md + backlog.md loaded. Starting estimation.`

  ## Date Confirmation

  - [ ] EST-09: Shows "Project start date detected: 2026-06-04"
  - [ ] EST-10: Consultant confirms (press Enter / "yes") → date stored as 2026-06-04
  - [ ] EST-11: No sprint date shifts applied (no override)

  ## Estimation Config

  - [ ] EST-12: Config block shown as single message with 3 questions (style / hours per day / cost)
  - [ ] EST-13: Consultant selects [A] Fixed
  - [ ] EST-14: Hours prompt shown with defaults: S=8, M=24, L=40, XL=80
  - [ ] EST-15: Consultant accepts defaults → hours_map = { S:8, M:24, L:40, XL:80 }
  - [ ] EST-16: Working hours per day = 8 (default accepted)
  - [ ] EST-17: Cost: N → include_cost = false

  ## Compute Estimates

  - [ ] EST-18: OracleConnector matched to Effort Signal "OracleConnector" → L → 40h / 5d
  - [ ] EST-19: SalesAggregator + data model matched → M → 24h / 3d
  - [ ] EST-20: SalesAPI + AlertEngine matched → L → 40h / 5d
  - [ ] EST-21: DashboardUI + AlertsUI matched → M → 24h / 3d
  - [ ] EST-22: Offline mode layer matched → S → 8h / 1d
  - [ ] EST-23: No "[size unknown — review]" rows
  - [ ] EST-24: Total: 136h / 17d

  ## Review Table

  - [ ] EST-25: Review table shows all 5 rows
  - [ ] EST-26: Size column present in review table (S/M/L/XL shown for reference)
  - [ ] EST-27: Cost column shows "—" for all rows
  - [ ] EST-28: Sprint, Start, End columns populated from backlog.md deadlines
  - [ ] EST-29: Total line shows: "Total: 136h / 17d | 2026-06-04 → 2026-09-04 | Cost: N/A"
  - [ ] EST-30: Consultant types "done" — no adjustments
  - [ ] EST-31: Skill proceeds without asking to re-enter values (no unknown rows)

  ## Write estimates.md

  - [ ] EST-32: estimates.md written to fixture directory
  - [ ] EST-33: Status line: `Status: DRAFT`
  - [ ] EST-34: Summary table shows: 136h / 17d, 2026-06-04 → 2026-09-04, Cost: N/A
  - [ ] EST-35: Section 1 groups by sprint — Sprint 1-2 section contains OracleConnector and SalesAggregator rows
  - [ ] EST-36: Section 1 Sprint 3-4 contains SalesAPI + AlertEngine
  - [ ] EST-37: Section 1 Sprint 5 contains DashboardUI + AlertsUI
  - [ ] EST-38: Section 1 Sprint 6 contains Offline mode layer
  - [ ] EST-39: No "S", "M", "L", or "XL" text appears anywhere in estimates.md
  - [ ] EST-40: Section 2 detailed table has 5 rows — one per ticket from backlog.md
  - [ ] EST-41: Section 2 Cost column shows "—" for all rows
  - [ ] EST-42: Section 3 Assumptions shows: S=8h, M=24h, L=40h, XL=80h, 8h/day, Cost: N/A
  - [ ] EST-43: Outputs: `estimates.md written — 5 items, 136h / 17d`

  ## Odoo Gate

  - [ ] EST-44: Gate shown (backlog.md was DRAFT)
  - [ ] EST-45: Options shown: [A] Create tickets in Odoo now / [B] Skip
  - [ ] EST-46: Consultant selects [B] Skip
  - [ ] EST-47: Outputs: "backlog.md and estimates.md saved as DRAFT."
  - [ ] EST-48: No MCP create calls made
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add tests/fixtures/estimator/synthetic-01/expected_behaviors.md
  git commit -m "test(estimator): expected behaviors for synthetic-01 PASS path"
  ```

---

## Task 4: Expected behaviors — synthetic-02-range (range, cost, date override, row adjustment)

**Files:**
- Create: `tests/fixtures/estimator/synthetic-02-range/expected_behaviors.md`

- [ ] **Step 1: Write expected_behaviors.md**

  ```markdown
  # Expected Behaviors — ESTIMATOR synthetic-02-range
  # Fixture: RetailEdge RANGE path — range hours, cost enabled, date override, row 3 adjusted

  ## Pre-flight

  - [ ] EST2-01: `basename "$PWD"` → "synthetic-02-range" (generic) → skill asks engagement name
  - [ ] EST2-02: Consultant provides "RetailEdge" → stored
  - [ ] EST2-03: arch.md + backlog.md loaded; no missing sections
  - [ ] EST2-04: Project start date detected: 2026-06-04
  - [ ] EST2-05: Outputs: `Engagement: RetailEdge | arch.md + backlog.md loaded. Starting estimation.`

  ## Date Confirmation

  - [ ] EST2-06: Shows "Project start date detected: 2026-06-04"
  - [ ] EST2-07: Consultant enters new date: 2026-07-01
  - [ ] EST2-08: Offset computed: +27 days
  - [ ] EST2-09: All sprint start/end dates shifted by 27 days:
    - Sprint 1-2: 2026-07-01 → 2026-07-29
    - Sprint 3-4: 2026-07-30 → 2026-08-26
    - Sprint 5: 2026-08-27 → 2026-09-09
    - Sprint 6: 2026-09-10 → 2026-10-01
  - [ ] EST2-10: Sprint durations preserved (same number of days per sprint)

  ## Estimation Config

  - [ ] EST2-11: Config block shown with 3 questions
  - [ ] EST2-12: Consultant selects [B] Range
  - [ ] EST2-13: Range prompt shown with defaults: S=4–8, M=16–32, L=32–48, XL=64–96
  - [ ] EST2-14: Consultant accepts defaults
  - [ ] EST2-15: Working hours = 8 (default)
  - [ ] EST2-16: Cost: Y
  - [ ] EST2-17: Rate input shown: [A] Blended / [B] Per-role
  - [ ] EST2-18: Consultant selects [A] Blended, enters: ₹5000/day
  - [ ] EST2-19: blended_rate = ₹5000/day stored

  ## Compute Estimates

  - [ ] EST2-20: OracleConnector → L → 32–48h / 4–6d / ₹20000–30000
  - [ ] EST2-21: SalesAggregator → M → 16–32h / 2–4d / ₹10000–20000
  - [ ] EST2-22: SalesAPI + AlertEngine → L → 32–48h / 4–6d / ₹20000–30000
  - [ ] EST2-23: DashboardUI + AlertsUI → M → 16–32h / 2–4d / ₹10000–20000
  - [ ] EST2-24: Offline mode → S → 4–8h / 1d / ₹5000 (ceil: 1d)
  - [ ] EST2-25: No "[size unknown — review]" rows
  - [ ] EST2-26: Total: 100–168h / 13–21d / ₹65000–1,05,000

  ## Review Table

  - [ ] EST2-27: Review table shows all 5 rows with range hours, days, and cost
  - [ ] EST2-28: Size column present in review table
  - [ ] EST2-29: Sprint dates shown with +27 day offset
  - [ ] EST2-30: Consultant enters adjustment: "3: 40–56h"
  - [ ] EST2-31: Row 3 updated: 40–56h / 5–7d / ₹25000–35000
  - [ ] EST2-32: Inline confirmation shown: `Row 3 updated: 40–56h / 5–7d`
  - [ ] EST2-33: Consultant types 'done'
  - [ ] EST2-34: Totals recomputed: 108–176h / 14–22d / ₹70000–1,10,000

  ## Write estimates.md

  - [ ] EST2-35: estimates.md written with range values
  - [ ] EST2-36: Summary shows range: 108–176h / 14–22d / ₹70000–1,10,000
  - [ ] EST2-37: Timeline uses shifted dates: 2026-07-01 → 2026-10-01
  - [ ] EST2-38: Section 1 Sprint 3-4 shows T03 row with 40–56h / 5–7d (post-adjustment)
  - [ ] EST2-39: No "S", "M", "L", or "XL" text in estimates.md
  - [ ] EST2-40: Section 3 shows: S=4–8h, M=16–32h, L=32–48h, XL=64–96h, 8h/day, ₹5000/day blended
  - [ ] EST2-41: Outputs: `estimates.md written — 5 items, 108–176h / 14–22d`

  ## Odoo Gate

  - [ ] EST2-42: Gate shown (backlog.md Status: DRAFT)
  - [ ] EST2-43: Consultant selects [B] Skip
  - [ ] EST2-44: Outputs: "backlog.md and estimates.md saved as DRAFT."
  - [ ] EST2-45: No MCP calls made
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add tests/fixtures/estimator/synthetic-02-range/expected_behaviors.md
  git commit -m "test(estimator): expected behaviors for synthetic-02-range RANGE path"
  ```

---

## Task 5: Write ESTIMATOR.md — Part 1 (Pre-flight through Compute)

**Files:**
- Create: `skills/project-initiator/ESTIMATOR.md`

- [ ] **Step 1: Create the skill file with header through Compute Estimates section**

  Create `skills/project-initiator/ESTIMATOR.md` with this exact content:

  ```markdown
  # VERSION: 1.5 | Last updated: 2026-06-08 | Reviewed: pending
  # ESTIMATOR — Project Initiator V1.5
  # Part of the fiftyfive-tech Project Initiator toolchain.

  ---

  ## Purpose

  ESTIMATOR reads `arch.md` and `backlog.md` from the engagement folder, converts S/M/L/XL
  effort signals to hours/days, maps estimates to sprint dates, and generates `estimates.md` —
  a structured estimation document with two sections:
  - **Section 1:** Executive summary for the client (grouped by sprint → feature)
  - **Section 2:** Detailed breakdown for the internal team or senior reviewer

  The file is format-neutral Markdown with flat tables (no merged cells, no nested rows),
  suitable for conversion to Excel, PDF, or Word. S/M/L/XL size labels never appear in
  `estimates.md` — always convert to hours/days first.

  V1.5 scope: this skill only. No SESSION_STATE.md dependencies.

  ---

  ## Pre-flight

  Before asking anything, silently:

  1. Run `basename "$PWD"` via Bash tool. Use result as engagement name.
     - If name is generic (e.g. `test`, `temp`, `folder`, `synthetic-01`, `synthetic-02-range`):
       ask once — "What's the engagement or client name?" Store the answer. Do not ask again.

  2. Read `arch.md` from current directory. Validate required sections:
     - `## Effort Signals` — S/M/L/XL per feature
     - `## Sprint Mapping` — sprint dates and owners
     - `## Build Order` — ordered item list

     If any are missing → stop:
     ```
     arch.md is missing required sections: [list]
     Re-run ARCH_PROPOSER to regenerate arch.md.
     ```

  3. Read `backlog.md` from current directory.
     - If not found → stop:
     ```
     No backlog.md found in this folder.

     ESTIMATOR requires a completed backlog.md as input.
     Run BACKLOG_GENERATOR first from this engagement folder, then run ESTIMATOR.

     Expected folder structure:
       ~/fiftyfive-engagements/<client-name>/
         arch.md        ← produced by ARCH_PROPOSER
         backlog.md     ← produced by BACKLOG_GENERATOR V1.4.1
     ```

  4. Check backlog.md Status line:
     - If `Status: CREATED` → warn and ask:
       ```
       ⚠ backlog.md Status: CREATED — tickets already exist in Odoo.
         Estimates can still be generated. Odoo gate will be skipped at the end.
         Proceed? (Y/N)
       ```
       On N: stop. On Y: set internal flag `skip_odoo_gate = true`.
     - If `Status: DRAFT` or no status line: continue silently.

  5. Check if `estimates.md` already exists in current directory:
     - If found:
       ```
       estimates.md already exists (Status: <current status>).
       [A] Overwrite — re-run full estimation
       [B] Cancel
       ```
       [B]: stop. [A]: continue (will overwrite).
     - If not found: continue silently.

  6. Extract from arch.md:
     - `## Effort Signals` → map of feature name → size label (S/M/L/XL)
     - `## Sprint Mapping` → sprint labels, date ranges, owners; detect project start date
       (look for a line matching `**Project start:**` or the first sprint's start date)
     - `## Build Order` → ordered list of items

  7. Extract from backlog.md:
     - Parent ticket list: title, sprint tag, assignee role, deadline for each ticket

  8. Detect project start date from step 6. If not parseable: note "start date unknown".

  Output one line on success:
  ```
  Engagement: <name> | arch.md + backlog.md loaded. Starting estimation.
  ```

  ---

  ## Date Confirmation

  Show detected date and ask for confirmation:

  ```
  Project start date detected: <date>   (or: "Start date not found in arch.md")
  Confirm, or enter a new start date (YYYY-MM-DD):
  ```

  - If user presses Enter or confirms with "yes" / "ok" / "correct": use detected date. No shifts.
  - If user enters a new date: compute `offset = new_date − original_start_date` in days.
    Shift all sprint start/end dates by the same offset. Sprint durations (days between start
    and end) are preserved — only the anchor moves.
  - If no date was detected and user does not provide one: set start = unknown.
    Start and End columns in estimates.md will show "—".

  Store confirmed start date for all downstream computations.

  ---

  ## Estimation Config

  Ask in a single block — wait for a single response that addresses all three:

  ```
  Estimation config for <engagement name>:

    1. Estimate style:
       [A] Fixed hours — one value per size
       [B] Range — low and high per size

    2. Working hours per day: [default: 8]

    3. Include cost estimates? (Y/N)
  ```

  After response, follow up based on answers:

  **If [A] Fixed:**
  ```
  Enter hours for each size (press Enter to accept defaults):
    S  = ?h   [default: 8]
    M  = ?h   [default: 24]
    L  = ?h   [default: 40]
    XL = ?h   [default: 80]
  ```
  Store: `hours_map = { S: N, M: N, L: N, XL: N }`.
  Pressing Enter for any field uses its default.

  **If [B] Range:**
  ```
  Enter hour ranges for each size (press Enter to accept defaults):
    S  = ?–?h   [default: 4–8]
    M  = ?–?h   [default: 16–32]
    L  = ?–?h   [default: 32–48]
    XL = ?–?h   [default: 64–96]
  ```
  Store: `hours_map = { S: [lo,hi], M: [lo,hi], L: [lo,hi], XL: [lo,hi] }`.
  Pressing Enter for any field uses its default.

  **If cost Y — ask immediately after hours config:**
  ```
  Rate input:
    [A] One blended day rate for all roles (e.g. ₹5000/day or $100/day)
    [B] Per-role rates
  ```
  If [A]: ask `Day rate? (e.g. ₹5000/day)`. Store as `blended_rate`.
  If [B]: display role list from Team Mapping in backlog.md. For each role ask one at a time:
  `Rate for <role>? (per day)`. Store as `role_rates = { role: rate }`.

  If cost N: set `include_cost = false`. Cost column shows "—" in all output.

  ---

  ## Compute Estimates

  For each item in `## Build Order` (in sequence order):

  1. **Match to Effort Signal:** Look up the item name against entries in `## Effort Signals`.
     Use case-insensitive partial match (e.g. "OracleConnector" matches "OracleConnector (shared)").
     - If no match found: flag this item as `[size unknown — review]`.
       Assign hours = 0 for fixed, or [0, 0] for range. These rows must be filled before 'done'.

  2. **Apply hours mapping:**
     - Fixed: `hours = hours_map[size]`
     - Range: `low_hours = hours_map[size][0]`, `high_hours = hours_map[size][1]`

  3. **Convert to days:**
     - Fixed: `days = ceil(hours / working_hours_per_day)`
     - Range: `low_days = ceil(low_hours / working_hours_per_day)`,
               `high_days = ceil(high_hours / working_hours_per_day)`

  4. **Look up sprint and dates from backlog.md:**
     - Match Build Order item name to parent ticket by title (case-insensitive, partial match).
     - Extract: sprint tag, assignee role, deadline (= end date for that sprint).
     - Compute start date: confirmed project start date + days from start to this sprint's
       first day (derived from Sprint Mapping offsets).
     - If deadline blank or sprint dates unknown: set start = "—", end = "—".

  5. **Compute cost (only if `include_cost = true`):**
     - Fixed + blended: `cost = days × blended_rate`
     - Fixed + per-role: `cost = days × role_rates[assignee_role]`
       (if role not in role_rates: show "—" for cost and flag inline)
     - Range + blended: `low_cost = low_days × blended_rate`,
                         `high_cost = high_days × blended_rate`
     - Range + per-role: same logic with `role_rates[assignee_role]`

  Store results as ordered list matching Build Order sequence:
  `estimates = [{ item, size, hours, days, sprint, assignee, start, end, cost }]`
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add skills/project-initiator/ESTIMATOR.md
  git commit -m "feat(estimator): V1.5 Part 1 — pre-flight, date confirmation, config, compute"
  ```

---

## Task 6: Write ESTIMATOR.md — Part 2 (Review Table through Rules)

**Files:**
- Modify: `skills/project-initiator/ESTIMATOR.md`

- [ ] **Step 1: Append Review Table section**

  Append to the end of `skills/project-initiator/ESTIMATOR.md`:

  ```markdown
  ---

  ## Review Table

  Show the full computed estimate table. The `Size` column appears here for the consultant's
  reference only — it is never written to `estimates.md`.

  ```
  Estimate preview — <engagement name> (<N> items):

  | #  | Feature / Build Order item       | Size | Hours     | Days    | Sprint     | Start      | End        | Cost             |
  |----|----------------------------------|------|-----------|---------|------------|------------|------------|------------------|
  | 1  | OracleConnector                  | L    | 40h       | 5d      | Sprint 1-2 | 2026-06-04 | 2026-07-02 | —                |
  | 2  | SalesAggregator + data model     | M    | 24h       | 3d      | Sprint 1-2 | 2026-06-04 | 2026-07-02 | —                |
  ...

  Total: <N>h / <N>d   |   <start> → <end>   |   Cost: <total or N/A>
  ```

  For range-based: Hours = `16–32h`, Days = `2–4d`, Cost = `₹X–Y`.
  For `[size unknown — review]` rows: Hours = `0h`, Days = `0d` — highlight clearly.

  ```
  Adjust any row? Enter row# and new hours (e.g. "3: 32h" or "3: 24–40h"), or type 'done'.
  ```

  **Adjustment loop:**
  - On adjustment input (e.g. "3: 32h"):
    - Parse row number and new hours (or range).
    - Recompute days: `days = ceil(hours / working_hours_per_day)`.
    - Recompute cost for that row if `include_cost = true`.
    - Output inline: `Row 3 updated: 32h / 4d` (and cost if applicable).
    - Accept next input.
  - On 'done' with `[size unknown — review]` rows still at 0h:
    - Output: `Rows [N] still have 0h — enter values before proceeding.`
    - Do not proceed until all rows have non-zero hours.
  - On 'done' with all rows valid:
    - Recompute totals across all rows.
    - Output summary line: `Total: <N>h / <N>d | <start> → <end> | Cost: <total or N/A>`
    - Proceed to write.

  ---

  ## Write estimates.md

  Write `estimates.md` to the current working directory. Use the template below exactly.
  Set `Status: DRAFT`.

  Output on success:
  ```
  estimates.md written — <N> items, <total hours>h / <total days>d
  ```

  ---

  ## Odoo Gate

  If `skip_odoo_gate = true` (backlog.md was Status: CREATED at pre-flight): skip this
  section entirely — output nothing, stop.

  Otherwise show:
  ```
  What's next?
  [A] Create tickets in Odoo now
  [B] Skip — keep backlog.md as DRAFT, push to Odoo manually later
  ```

  Natural-language handling:
  - "yes", "create now", "go ahead", "A" → [A]
  - "no", "skip", "later", "not now", "B" → [B]

  **[A] — execute Bulk Creation:**
  Follow the identical flow defined in BACKLOG_GENERATOR `## Bulk Creation`:
  1. If Odoo user IDs in backlog.md Team Mapping show "—": ask consultant to fill in or skip.
  2. Call `create_project(name=<project_name>, stages=[<stage_list>])`.
  3. Call `create_tag` for each sprint label. Store sprint label → tag_id map.
  4. Call `list_tickets(project_id=<id>, limit=100)` for duplicate check.
  5. Show preview table from backlog.md. Ask: `Create these <N> tickets in Odoo? [A] Yes / [B] Cancel`
  6. On Yes: execute `bulk_create_tickets` and `add_subtasks` per BACKLOG_GENERATOR spec.
  7. Update `backlog.md` Status line to `Status: CREATED — tickets in Odoo (project ID: <id>)`.

  **[B] — skip:**
  ```
  backlog.md and estimates.md saved as DRAFT.
  To push to Odoo later: run BACKLOG_GENERATOR — it will detect backlog.md and offer push mode.
  ```
  Stop.

  ---

  ## estimates.md Template

  Write `estimates.md` using this structure exactly. No merged cells. No nested rows.
  No S/M/L/XL labels in any column.

  ```markdown
  # Estimates — <Engagement Name>
  # Generated: <YYYY-MM-DD> | ESTIMATOR V1.5
  # Status: DRAFT

  ---

  ## Summary

  | Field | Value |
  |---|---|
  | Total effort | <N>h / <N>d  (range: <low>–<high>h / <low>–<high>d) |
  | Timeline | <project start> → <project end> |
  | Team size | <N> people (from backlog.md Team Mapping) |
  | Sprints | <N> sprints (from Sprint Mapping) |
  | Cost estimate | <total or range, or N/A> |

  ---

  ## Section 1 — Client Summary

  One subsection per sprint, in sprint order. Within each sprint subsection, one row
  per Build Order item assigned to that sprint.

  ### Sprint 1 — <sprint start date> → <sprint end date>

  | Feature | Hours | Days | Cost |
  |---|---|---|---|
  | <Build Order item name> | <N>h | <N>d | <cost or —> |

  **Sprint 1 total: <N>h / <N>d / <cost or N/A>**

  (repeat for each sprint — Sprint 2, Sprint 3, etc.)

  ---
  **Project total: <N>h / <N>d / <cost or N/A>**

  ---

  ## Section 2 — Detailed Breakdown

  One row per parent ticket from backlog.md, in Build Order sequence.

  | # | Ticket | Sprint | Assignee | Hours | Days | Start | End | Cost |
  |---|---|---|---|---|---|---|---|---|
  | T01 | <ticket title from backlog.md> | Sprint 1 | <role> | <N>h | <N>d | <date> | <date> | <cost or —> |

  **Total: <N>h / <N>d / <cost or N/A>**

  ---

  ## Section 3 — Assumptions

  - Estimate basis: S=<N>h, M=<N>h, L=<N>h, XL=<N>h
    (or: S=<low>–<high>h, M=<low>–<high>h, L=<low>–<high>h, XL=<low>–<high>h for range)
  - Working hours per day: <N>
  - Cost rates: <per-role table or blended rate statement, or N/A>
  - Excludes: QA effort beyond sprint allocation, DevOps setup time,
    client feedback cycles, post-go-live support
  ```

  ---

  ## Rules for this skill

  1. S/M/L/XL size labels never appear in `estimates.md` — always convert to hours/days first.
  2. All tables in `estimates.md` must be flat: no merged cells, no nested rows.
  3. Cost column always shows "—" when cost is opted out — never leave blank.
  4. Never modify `arch.md`, `backlog.md`, or any other upstream file.
  5. If `estimates.md` already exists and [A] overwrite is chosen: previous file is fully replaced.
  6. Odoo gate is skipped if `backlog.md` Status was CREATED at pre-flight.
  7. LAYER_0_GLOBAL Rule 4 output limits and Rule 5 (no narration) apply.
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add skills/project-initiator/ESTIMATOR.md
  git commit -m "feat(estimator): V1.5 Part 2 — review table, write file, Odoo gate, template, rules"
  ```

---

## Task 7: START.md and CLAUDE.md updates

**Files:**
- Modify: `skills/project-initiator/START.md`
- Modify: `CLAUDE.md`

- [ ] **Step 1: Update START.md stage routing**

  In `skills/project-initiator/START.md`, find the "Next skill by stage" block:
  ```
  - `DOC_GENERATOR complete` → BACKLOG_GENERATOR (not yet built — V1.4)
  ```

  Replace with:
  ```
  - `DOC_GENERATOR complete` → BACKLOG_GENERATOR
  - `BACKLOG_GENERATOR complete` → ESTIMATOR
  - `ESTIMATOR complete` → (chain complete — ROADMAP in future)
  ```

- [ ] **Step 2: Update START.md stage auto-detection**

  Find:
  ```
  Detect stage from files present (check in order — stop at first match):
  - `arch.md` present → `ARCH_PROPOSER complete`
  - `mvp-scope.md` present → `MVP_SYNTHESIZER complete`
  - `discovery.md` present → `DISCOVERY complete`
  - None of the above → `not started`
  ```

  Replace with:
  ```
  Detect stage from files present (check in order — stop at first match):
  - `estimates.md` present → `ESTIMATOR complete`
  - `backlog.md` present → `BACKLOG_GENERATOR complete`
  - `arch.md` present → `ARCH_PROPOSER complete`
  - `mvp-scope.md` present → `MVP_SYNTHESIZER complete`
  - `discovery.md` present → `DISCOVERY complete`
  - None of the above → `not started`
  ```

- [ ] **Step 3: Update CLAUDE.md Phase Status table**

  In `CLAUDE.md`, find:
  ```
  | PI V1.4 | BACKLOG_GENERATOR skill — built, tested on 2 synthetic fixtures (72/72 + 22/22) | Done |
  ```

  Replace with:
  ```
  | PI V1.4 | BACKLOG_GENERATOR skill — built, tested on 2 synthetic fixtures (72/72 + 22/22) | Done |
  | PI V1.4.1 | BACKLOG_GENERATOR patch — write backlog.md first, 3-option Next Steps Gate | Done |
  | PI V1.5 | ESTIMATOR skill — built, tested on 2 synthetic fixtures | Not started |
  ```

- [ ] **Step 4: Update CLAUDE.md chain status**

  Find:
  ```
  START (V1.2, done) → DISCOVERY (done) → MVP_SYNTHESIZER (done) → ARCH_PROPOSER (done) → DOC_GENERATOR (done) → BACKLOG_GENERATOR (done) → ROADMAP (future)
  ```

  Replace with:
  ```
  START (V1.2, done) → DISCOVERY (done) → MVP_SYNTHESIZER (done) → ARCH_PROPOSER (done) → DOC_GENERATOR (done) → BACKLOG_GENERATOR (V1.4.1, done) → ESTIMATOR (V1.5, in progress) → ROADMAP (future)
  ```

- [ ] **Step 5: Add ESTIMATOR section to CLAUDE.md Project Initiator block**

  Find:
  ```
  MCP tools pending for future phases: `odoo-mcp/PENDING_CHANGES.md`
  ```

  Insert before that line:
  ```markdown
  ### ESTIMATOR (V1.5 — in progress)

  Skill location: `skills/project-initiator/ESTIMATOR.md`

  **How to run:** Open Claude Code in the `<client-name>/` folder (with `arch.md` and
  `backlog.md` present). Say `run ESTIMATOR`. Reads both files, runs pre-flight,
  asks for estimate style (fixed/range), optional cost rates, and project start date.
  Generates `estimates.md` with executive summary + detailed breakdown. Ends with Odoo gate.

  **Readiness Gate:** Checks for both `arch.md` (requires Effort Signals + Sprint Mapping +
  Build Order) and `backlog.md`. BLOCKs with clear message pointing to missing upstream skill.

  Design spec: `docs/superpowers/specs/2026-06-08-estimator-design.md`

  Test fixtures: `tests/fixtures/estimator/synthetic-01/` (fixed hours PASS),
  `tests/fixtures/estimator/synthetic-02-range/` (range + cost + date override)

  ```

- [ ] **Step 6: Commit**

  ```bash
  git add skills/project-initiator/START.md CLAUDE.md
  git commit -m "feat(chain): wire ESTIMATOR V1.5 into START routing and CLAUDE.md"
  ```

---

## Task 8: TEST_SCRIPT.md

**Files:**
- Create: `tests/fixtures/estimator/TEST_SCRIPT.md`

- [ ] **Step 1: Write TEST_SCRIPT.md**

  ```markdown
  # ESTIMATOR Test Script
  # Skill: skills/project-initiator/ESTIMATOR.md
  # Version: V1.5

  ---

  ## Fixture Overview

  | Fixture | Path | Purpose |
  |---|---|---|
  | synthetic-01 | tests/fixtures/estimator/synthetic-01/ | PASS path — fixed hours, no cost, confirmed dates, no adjustments |
  | synthetic-02-range | tests/fixtures/estimator/synthetic-02-range/ | RANGE path — range hours, cost enabled, date override, row 3 adjusted |

  ---

  ## Running synthetic-01 (PASS path)

  1. Open Claude Code
  2. Set working directory to: `tests/fixtures/estimator/synthetic-01/`
  3. Say: `run ESTIMATOR`
  4. When asked for engagement name: type **RetailEdge**
  5. Date confirmation: press Enter to accept **2026-06-04**
  6. Estimation config:
     - Style: **A** (Fixed)
     - Hours: press Enter to accept defaults (S=8, M=24, L=40, XL=80)
     - Hours per day: press Enter for **8**
     - Cost: **N**
  7. Review table: type **done** (no adjustments)
  8. Odoo gate: **B** (Skip)
  9. Check that `estimates.md` was written to the fixture folder
  10. Score against: `tests/fixtures/estimator/synthetic-01/expected_behaviors.md`
  11. Clean up: delete `estimates.md` from fixture folder after scoring

  ---

  ## Running synthetic-02-range (RANGE path)

  1. Open Claude Code
  2. Set working directory to: `tests/fixtures/estimator/synthetic-02-range/`
  3. Say: `run ESTIMATOR`
  4. When asked for engagement name: type **RetailEdge**
  5. Date confirmation: enter **2026-07-01** (override)
  6. Estimation config:
     - Style: **B** (Range)
     - Ranges: press Enter to accept defaults (S=4–8, M=16–32, L=32–48, XL=64–96)
     - Hours per day: press Enter for **8**
     - Cost: **Y**
     - Rate type: **A** (Blended)
     - Day rate: **₹5000/day**
  7. Review table: type **3: 40–56h** to adjust row 3, then type **done**
  8. Odoo gate: **B** (Skip)
  9. Check that `estimates.md` was written to the fixture folder
  10. Score against: `tests/fixtures/estimator/synthetic-02-range/expected_behaviors.md`
  11. Clean up: delete `estimates.md` from fixture folder after scoring

  ---

  ## Scoring

  Record results in a `test_report.md` file in each fixture folder.
  Use format: `# Test Report — ESTIMATOR <fixture-name> | Score: N/N | Date: YYYY-MM-DD`
  followed by the checked-off expected_behaviors.md content.

  Pass threshold: all checkboxes marked [x].
  ```

- [ ] **Step 2: Commit**

  ```bash
  git add tests/fixtures/estimator/TEST_SCRIPT.md
  git commit -m "test(estimator): TEST_SCRIPT.md for synthetic-01 and synthetic-02-range"
  ```

---

## Self-Review Checklist

- [x] **Spec coverage:** All 7 spec sections covered — pre-flight (Task 5), date confirmation (Task 5), estimation config (Task 5), compute (Task 5), review table (Task 6), write file (Task 6), Odoo gate (Task 6). BACKLOG_GENERATOR patch (Task 1). Readiness gate (covered in pre-flight rules). Error handling (covered in pre-flight stop conditions). Chain updates (Task 7). Testing (Tasks 3, 4, 8).
- [x] **Placeholders:** None. All tasks have exact file content or exact string replacements.
- [x] **Type consistency:** No function calls — skill files only. Section names referenced consistently: "Estimation Config" in spec → `## Estimation Config` in skill. "Review Table" → `## Review Table`. "Odoo Gate" → `## Odoo Gate`.
- [x] **Prerequisite order:** Task 1 (BACKLOG_GENERATOR patch) before Task 5 (ESTIMATOR references Next Steps Gate concept). Fixtures (Task 2) before expected behaviors (Tasks 3, 4). Skill Parts 1+2 (Tasks 5, 6) before doc updates (Task 7).
