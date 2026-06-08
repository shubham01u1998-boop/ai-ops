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
Use format:
```
# Test Report — ESTIMATOR <fixture-name> | Score: N/N | Date: YYYY-MM-DD
```
followed by the checked-off expected_behaviors.md content.

Pass threshold: all checkboxes marked [x].
