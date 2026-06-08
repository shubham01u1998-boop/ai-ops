# Expected Behaviors — ESTIMATOR synthetic-01
# Fixture: RetailEdge PASS path — fixed hours, no cost, confirmed dates, no adjustments

## Pre-flight

- [ ] EST-01: `basename "$PWD"` returns "synthetic-01" (generic) → skill asks for engagement name
- [ ] EST-02: Consultant provides "RetailEdge" → stored and used throughout
- [ ] EST-03: arch.md loaded; required sections present: Effort Signals, Sprint Mapping, Build Order
- [ ] EST-04: backlog.md loaded
- [ ] EST-05: backlog.md Status: DRAFT — no "already in Odoo" warning shown
- [ ] EST-06: estimates.md not found — no overwrite prompt shown
- [ ] EST-07: Project start date detected: 2026-06-04 (from "Project start:" line in Sprint Mapping)
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

- [ ] EST-18: OracleConnector matched to Effort Signal → L → 40h / 5d
- [ ] EST-19: SalesAggregator + data model matched → M → 24h / 3d
- [ ] EST-20: SalesAPI + AlertEngine matched → L → 40h / 5d
- [ ] EST-21: DashboardUI + AlertsUI matched → M → 24h / 3d
- [ ] EST-22: Offline mode layer matched → S → 8h / 1d
- [ ] EST-23: No "[size unknown — review]" rows
- [ ] EST-24: Total: 136h / 17d

## Review Table

- [ ] EST-25: Review table shows all 5 rows
- [ ] EST-26: Size column present in review table (S/M/L/XL shown for reference only)
- [ ] EST-27: Cost column shows "—" for all rows
- [ ] EST-28: Sprint, Start, End columns populated from backlog.md deadlines and sprint dates
- [ ] EST-29: Total line shows 136h / 17d
- [ ] EST-30: Consultant types "done" — no adjustments made
- [ ] EST-31: Skill proceeds without requesting values for unknown rows (none exist)

## Write estimates.md

- [ ] EST-32: estimates.md written to fixture directory
- [ ] EST-33: Status line: `Status: DRAFT`
- [ ] EST-34: Summary table shows: 136h / 17d, 2026-06-04 → 2026-09-04, Cost: N/A
- [ ] EST-35: Section 1 groups by sprint — Sprint 1-2 section contains OracleConnector and SalesAggregator + data model rows
- [ ] EST-36: Section 1 Sprint 3-4 contains SalesAPI + AlertEngine
- [ ] EST-37: Section 1 Sprint 5 contains DashboardUI + AlertsUI
- [ ] EST-38: Section 1 Sprint 6 contains Offline mode layer
- [ ] EST-39: No "S", "M", "L", or "XL" text appears in any table cell in estimates.md
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
