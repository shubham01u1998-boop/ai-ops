# Test Report — ESTIMATOR synthetic-01 | Score: 48/48 | Date: 2026-06-08
# Run: 2026-06-08 | Tester: Claude (simulated QA run)
# Skill: skills/project-initiator/ESTIMATOR.md V1.5
# Fixture: RetailEdge PASS path — fixed hours, no cost, confirmed dates, no adjustments

---

## Score by Section

| Section | Score | Notes |
|---|---|---|
| Pre-flight | 8/8 | Generic folder name → engagement name prompt; all sections present; no warnings triggered |
| Date Confirmation | 3/3 | Detected 2026-06-04; confirmed; no shifts applied |
| Estimation Config | 6/6 | Single-block 3-question prompt; Fixed style; default hours; cost N |
| Compute Estimates | 7/7 | All 5 items matched; correct hours/days; total 136h/17d |
| Review Table | 7/7 | All 5 rows; size column present; "—" for cost; "done" accepted |
| Write estimates.md | 12/12 | All sections correct; no S/M/L/XL in cells; Status: DRAFT |
| Odoo Gate | 5/5 | Gate shown; [B] Skip accepted; correct closing message; no MCP calls |
| **TOTAL** | **48/48** | |

---

## Detailed Checklist

### Pre-flight
- [x] EST-01: `basename "$PWD"` returns "synthetic-01" (generic) → skill asks for engagement name
- [x] EST-02: Consultant provides "RetailEdge" → stored and used throughout
- [x] EST-03: arch.md loaded; required sections present: Effort Signals, Sprint Mapping, Build Order
- [x] EST-04: backlog.md loaded
- [x] EST-05: backlog.md Status: DRAFT — no "already in Odoo" warning shown
- [x] EST-06: estimates.md not found — no overwrite prompt shown
- [x] EST-07: Project start date detected: 2026-06-04 (from "Project start:" line in Sprint Mapping)
- [x] EST-08: Outputs: `Engagement: RetailEdge | arch.md + backlog.md loaded. Starting estimation.`

### Date Confirmation
- [x] EST-09: Shows "Project start date detected: 2026-06-04"
- [x] EST-10: Consultant confirms (press Enter / "yes") → date stored as 2026-06-04
- [x] EST-11: No sprint date shifts applied (no override)

### Estimation Config
- [x] EST-12: Config block shown as single message with 3 questions (style / hours per day / cost)
- [x] EST-13: Consultant selects [A] Fixed
- [x] EST-14: Hours prompt shown with defaults: S=8, M=24, L=40, XL=80
- [x] EST-15: Consultant accepts defaults → hours_map = { S:8, M:24, L:40, XL:80 }
- [x] EST-16: Working hours per day = 8 (default accepted)
- [x] EST-17: Cost: N → include_cost = false

### Compute Estimates
- [x] EST-18: OracleConnector matched to Effort Signal → L → 40h / 5d
- [x] EST-19: SalesAggregator + data model matched → M → 24h / 3d
- [x] EST-20: SalesAPI + AlertEngine matched → L → 40h / 5d
- [x] EST-21: DashboardUI + AlertsUI matched → M → 24h / 3d
- [x] EST-22: Offline mode layer matched → S → 8h / 1d
- [x] EST-23: No "[size unknown — review]" rows
- [x] EST-24: Total: 136h / 17d

### Review Table
- [x] EST-25: Review table shows all 5 rows
- [x] EST-26: Size column present in review table (S/M/L/XL shown for reference only)
- [x] EST-27: Cost column shows "—" for all rows
- [x] EST-28: Sprint, Start, End columns populated from backlog.md deadlines and sprint dates
- [x] EST-29: Total line shows 136h / 17d
- [x] EST-30: Consultant types "done" — no adjustments made
- [x] EST-31: Skill proceeds without requesting values for unknown rows (none exist)

### Write estimates.md
- [x] EST-32: estimates.md written to fixture directory
- [x] EST-33: Status line: `Status: DRAFT`
- [x] EST-34: Summary table shows: 136h / 17d, 2026-06-04 → 2026-09-04, Cost: N/A
- [x] EST-35: Section 1 groups by sprint — Sprint 1-2 section contains OracleConnector and SalesAggregator + data model rows
- [x] EST-36: Section 1 Sprint 3-4 contains SalesAPI + AlertEngine
- [x] EST-37: Section 1 Sprint 5 contains DashboardUI + AlertsUI
- [x] EST-38: Section 1 Sprint 6 contains Offline mode layer
- [x] EST-39: No "S", "M", "L", or "XL" text appears in any table cell in estimates.md
- [x] EST-40: Section 2 detailed table has 5 rows — one per ticket from backlog.md
- [x] EST-41: Section 2 Cost column shows "—" for all rows
- [x] EST-42: Section 3 Assumptions shows: S=8h, M=24h, L=40h, XL=80h, 8h/day, Cost: N/A
- [x] EST-43: Outputs: `estimates.md written — 5 items, 136h / 17d`

### Odoo Gate
- [x] EST-44: Gate shown (backlog.md was DRAFT)
- [x] EST-45: Options shown: [A] Create tickets in Odoo now / [B] Skip
- [x] EST-46: Consultant selects [B] Skip
- [x] EST-47: Outputs: "backlog.md and estimates.md saved as DRAFT."
- [x] EST-48: No MCP create calls made

---

## QA Observations

### Generic folder name handling
`basename "$PWD"` returns "synthetic-01" which matches the generic-name check. Skill correctly
prompts for engagement name. "RetailEdge" is stored and used consistently in all output
headers, section titles, and the summary block.

### Date detection
`**Project start:** 2026-06-04` in arch.md Sprint Mapping is parsed correctly. No date override
provided → no sprint shifts. All start/end dates in Section 2 pull directly from backlog.md
Sprint Tags (start = sprint first day, end = deadline field).

### Arithmetic verification
- S=8h → ceil(8/8)=1d ✓
- M=24h → ceil(24/8)=3d ✓
- L=40h → ceil(40/8)=5d ✓
- Total: 2×L + 2×M + 1×S = 80+48+8 = 136h / 10+6+1 = 17d ✓

### Size labels in estimates.md
Verified: no "S", "M", "L", or "XL" appears in any table cell. Size column only appears in
the review table (shown to consultant, never written to file). Section 3 Assumptions uses
`S=8h` form (label before equals sign, not in a table cell) — this is correct per spec.

### Note on cleanup
estimates.md was written to this fixture folder as part of the simulated run. Delete before
re-running this fixture to avoid triggering the overwrite prompt (EST-06).
