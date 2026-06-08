# Expected Behaviors — ESTIMATOR synthetic-02-range
# Fixture: RetailEdge RANGE path — range hours, cost enabled, date override, row 3 adjusted

## Pre-flight

- [ ] EST2-01: `basename "$PWD"` returns "synthetic-02-range" (generic) → skill asks for engagement name
- [ ] EST2-02: Consultant provides "RetailEdge" → stored
- [ ] EST2-03: arch.md + backlog.md loaded; no missing sections
- [ ] EST2-04: Project start date detected: 2026-06-04
- [ ] EST2-05: Outputs: `Engagement: RetailEdge | arch.md + backlog.md loaded. Starting estimation.`

## Date Confirmation

- [ ] EST2-06: Shows "Project start date detected: 2026-06-04"
- [ ] EST2-07: Consultant enters new date: 2026-07-01
- [ ] EST2-08: Offset computed: +27 days
- [ ] EST2-09: Sprint dates shifted by 27 days — Sprint 1-2 end: 2026-07-29, Sprint 3-4 end: 2026-08-26, Sprint 5 end: 2026-09-09, Sprint 6 end: 2026-10-01
- [ ] EST2-10: Sprint durations preserved (same number of working days per sprint)

## Estimation Config

- [ ] EST2-11: Config block shown with 3 questions
- [ ] EST2-12: Consultant selects [B] Range
- [ ] EST2-13: Range prompt shown with defaults: S=4–8, M=16–32, L=32–48, XL=64–96
- [ ] EST2-14: Consultant accepts defaults (press Enter)
- [ ] EST2-15: Working hours = 8 (default accepted)
- [ ] EST2-16: Cost: Y
- [ ] EST2-17: Rate input shown: [A] Blended / [B] Per-role
- [ ] EST2-18: Consultant selects [A] Blended, enters: ₹5000/day
- [ ] EST2-19: blended_rate = ₹5000/day stored

## Compute Estimates

- [ ] EST2-20: OracleConnector → L → 32–48h / 4–6d
- [ ] EST2-21: SalesAggregator + data model → M → 16–32h / 2–4d
- [ ] EST2-22: SalesAPI + AlertEngine → L → 32–48h / 4–6d (before adjustment)
- [ ] EST2-23: DashboardUI + AlertsUI → M → 16–32h / 2–4d
- [ ] EST2-24: Offline mode layer → S → 4–8h / 1d (ceil(4/8)=1, ceil(8/8)=1)
- [ ] EST2-25: No "[size unknown — review]" rows

## Review Table

- [ ] EST2-26: Review table shows all 5 rows with range hours, days, and cost
- [ ] EST2-27: Size column present in review table (S/M/L/XL for reference only)
- [ ] EST2-28: Sprint dates shown with +27 day offset applied
- [ ] EST2-29: Consultant enters adjustment: "3: 40–56h"
- [ ] EST2-30: Row 3 updated: 40–56h / 5–7d
- [ ] EST2-31: Inline confirmation shown: `Row 3 updated: 40–56h / 5–7d`
- [ ] EST2-32: Cost for row 3 recalculated: ₹25000–35000 (5d × ₹5000, 7d × ₹5000)
- [ ] EST2-33: Consultant types "done"

## Write estimates.md

- [ ] EST2-34: estimates.md written with range values
- [ ] EST2-35: Status: DRAFT
- [ ] EST2-36: Summary shows range totals
- [ ] EST2-37: Timeline uses shifted dates: 2026-07-01 → 2026-10-01
- [ ] EST2-38: Section 1 Sprint 3-4 shows T03 row with 40–56h / 5–7d (post-adjustment)
- [ ] EST2-39: No "S", "M", "L", or "XL" text appears in any table cell in estimates.md
- [ ] EST2-40: Section 3 shows range assumptions: S=4–8h, M=16–32h, L=32–48h, XL=64–96h, 8h/day, ₹5000/day blended
- [ ] EST2-41: Outputs: `estimates.md written — 5 items, ...`

## Odoo Gate

- [ ] EST2-42: Gate shown (backlog.md Status: DRAFT)
- [ ] EST2-43: Consultant selects [B] Skip
- [ ] EST2-44: Outputs: "backlog.md and estimates.md saved as DRAFT."
- [ ] EST2-45: No MCP calls made
