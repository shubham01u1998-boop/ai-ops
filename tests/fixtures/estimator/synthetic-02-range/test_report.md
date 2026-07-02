# Test Report — ESTIMATOR synthetic-02-range | Score: 45/45 | Date: 2026-06-08
# Run: 2026-06-08 | Tester: Claude (simulated QA run)
# Skill: skills/project-initiator/ESTIMATOR.md V1.5
# Fixture: RetailEdge RANGE path — range hours, cost enabled, date override, row 3 adjusted

---

## Score by Section

| Section | Score | Notes |
|---|---|---|
| Pre-flight | 5/5 | Generic folder name → engagement name prompt; all sections present |
| Date Confirmation | 5/5 | Detected 2026-06-04; override 2026-07-01; +27d offset; all sprint dates shifted; durations preserved |
| Estimation Config | 9/9 | Range style; default ranges accepted; cost Y; blended rate ₹5000/day |
| Compute Estimates | 6/6 | All 5 items matched; correct range hours/days; no unknown rows |
| Review Table | 8/8 | All rows with range values; row 3 adjusted 40–56h; cost recalculated; "done" accepted |
| Write estimates.md | 8/8 | Range values throughout; shifted dates; Section 3 range assumptions; Status: DRAFT |
| Odoo Gate | 4/4 | Gate shown; [B] Skip; correct closing message; no MCP calls |
| **TOTAL** | **45/45** | |

---

## Detailed Checklist

### Pre-flight
- [x] EST2-01: `basename "$PWD"` returns "synthetic-02-range" (generic) → skill asks for engagement name
- [x] EST2-02: Consultant provides "RetailEdge" → stored
- [x] EST2-03: arch.md + backlog.md loaded; no missing sections
- [x] EST2-04: Project start date detected: 2026-06-04
- [x] EST2-05: Outputs: `Engagement: RetailEdge | arch.md + backlog.md loaded. Starting estimation.`

### Date Confirmation
- [x] EST2-06: Shows "Project start date detected: 2026-06-04"
- [x] EST2-07: Consultant enters new date: 2026-07-01
- [x] EST2-08: Offset computed: +27 days
- [x] EST2-09: Sprint dates shifted by 27 days — Sprint 1-2 end: 2026-07-29, Sprint 3-4 end: 2026-08-26, Sprint 5 end: 2026-09-09, Sprint 6 end: 2026-10-01
- [x] EST2-10: Sprint durations preserved (same number of working days per sprint)

### Estimation Config
- [x] EST2-11: Config block shown with 3 questions
- [x] EST2-12: Consultant selects [B] Range
- [x] EST2-13: Range prompt shown with defaults: S=4–8, M=16–32, L=32–48, XL=64–96
- [x] EST2-14: Consultant accepts defaults (press Enter)
- [x] EST2-15: Working hours = 8 (default accepted)
- [x] EST2-16: Cost: Y
- [x] EST2-17: Rate input shown: [A] Blended / [B] Per-role
- [x] EST2-18: Consultant selects [A] Blended, enters: ₹5000/day
- [x] EST2-19: blended_rate = ₹5000/day stored

### Compute Estimates
- [x] EST2-20: OracleConnector → L → 32–48h / 4–6d
- [x] EST2-21: SalesAggregator + data model → M → 16–32h / 2–4d
- [x] EST2-22: SalesAPI + AlertEngine → L → 32–48h / 4–6d (before adjustment)
- [x] EST2-23: DashboardUI + AlertsUI → M → 16–32h / 2–4d
- [x] EST2-24: Offline mode layer → S → 4–8h / 1d (ceil(4/8)=1, ceil(8/8)=1)
- [x] EST2-25: No "[size unknown — review]" rows

### Review Table
- [x] EST2-26: Review table shows all 5 rows with range hours, days, and cost
- [x] EST2-27: Size column present in review table (S/M/L/XL for reference only)
- [x] EST2-28: Sprint dates shown with +27 day offset applied
- [x] EST2-29: Consultant enters adjustment: "3: 40–56h"
- [x] EST2-30: Row 3 updated: 40–56h / 5–7d
- [x] EST2-31: Inline confirmation shown: `Row 3 updated: 40–56h / 5–7d`
- [x] EST2-32: Cost for row 3 recalculated: ₹25000–₹35000 (5d × ₹5000, 7d × ₹5000)
- [x] EST2-33: Consultant types "done"

### Write estimates.md
- [x] EST2-34: estimates.md written with range values
- [x] EST2-35: Status: DRAFT
- [x] EST2-36: Summary shows range totals (108–176h / 14–22d / ₹70000–₹110000)
- [x] EST2-37: Timeline uses shifted dates: 2026-07-01 → 2026-10-01
- [x] EST2-38: Section 1 Sprint 3-4 shows T03 row with 40–56h / 5–7d (post-adjustment)
- [x] EST2-39: No "S", "M", "L", or "XL" text appears in any table cell in estimates.md
- [x] EST2-40: Section 3 shows range assumptions: S=4–8h, M=16–32h, L=32–48h, XL=64–96h, 8h/day, ₹5000/day blended
- [x] EST2-41: Outputs: `estimates.md written — 5 items, 108–176h / 14–22d`

### Odoo Gate
- [x] EST2-42: Gate shown (backlog.md Status: DRAFT)
- [x] EST2-43: Consultant selects [B] Skip
- [x] EST2-44: Outputs: "backlog.md and estimates.md saved as DRAFT."
- [x] EST2-45: No MCP calls made

---

## QA Observations

### Date offset arithmetic
Original start: 2026-06-04. New start: 2026-07-01. Offset = +27 days.
Shift verification:
- Sprint 1-2: 2026-06-04+27=2026-07-01 start ✓, 2026-07-02+27=2026-07-29 end ✓
- Sprint 3-4: 2026-07-03+27=2026-07-30 start ✓, 2026-07-30+27=2026-08-26 end ✓
- Sprint 5:   2026-07-31+27=2026-08-27 start ✓, 2026-08-13+27=2026-09-09 end ✓
- Sprint 6:   2026-08-14+27=2026-09-10 start ✓, 2026-09-04+27=2026-10-01 end ✓
All sprint durations preserved (28d, 27d, 13d, 21d).

### Range arithmetic
Default ranges: S=4–8h, M=16–32h, L=32–48h, XL=64–96h. days = ceil(h/8).
- S: ceil(4/8)=1, ceil(8/8)=1 → 1d (both round to same day — shows "1d" not "1–1d") ✓
- M: ceil(16/8)=2, ceil(32/8)=4 → 2–4d ✓
- L: ceil(32/8)=4, ceil(48/8)=6 → 4–6d ✓

### Row 3 adjustment
Pre-adjustment: SalesAPI + AlertEngine → L → 32–48h / 4–6d / ₹20000–₹30000
Input: "3: 40–56h"
Post-adjustment: 40–56h / ceil(40/8)=5d – ceil(56/8)=7d / 5×5000=₹25000 – 7×5000=₹35000 ✓

### Total recomputation after adjustment
Low:  32+16+40+16+4  = 108h / 4+2+5+2+1=14d / ₹20000+₹10000+₹25000+₹10000+₹5000=₹70000 ✓
High: 48+32+56+32+8  = 176h / 6+4+7+4+1=22d / ₹30000+₹20000+₹35000+₹20000+₹5000=₹110000 ✓

### Offline mode layer cost
Low days = high days = 1d → cost ₹5000 (not a range). Displayed as ₹5000, not ₹5000–₹5000.
This is correct behavior — no need to show a range when both values are equal.

### Note on cleanup
estimates.md was written to this fixture folder as part of the simulated run. Delete before
re-running this fixture to avoid triggering the overwrite prompt.
