# ESTIMATOR — Design Spec
# Project Initiator V1.5
# Date: 2026-06-08 | Author: Shubham Upadhyay

---

## Purpose

ESTIMATOR is V1.5 of the Project Initiator chain. It sits after BACKLOG_GENERATOR and
generates `estimates.md` — a structured estimation document with two sections:
an executive summary for the client and a detailed breakdown for the internal team or
senior reviewer.

The file is format-neutral Markdown with flat tables (no merged cells, no nested rows),
suitable for direct conversion to Excel, PDF, or Word without reformatting.

S/M/L/XL effort signals from `arch.md` are internal inputs only — they never appear in
the output document. The skill always converts them to hours/days before writing.

---

## Position in Chain

```
START → DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → DOC_GENERATOR
  → BACKLOG_GENERATOR (V1.4.1) → ESTIMATOR (V1.5) → [Odoo creation gate]
```

ESTIMATOR can also be run standalone from any engagement folder that has both
`arch.md` and `backlog.md` present.

---

## BACKLOG_GENERATOR V1.4.1 Change (prerequisite)

A small patch to BACKLOG_GENERATOR is required before ESTIMATOR can be introduced.

**Current behaviour:** Preview gate offers [A] Create on Odoo | [B] Save to backlog.md
only | [C] Edit | [D] Cancel — file write is optional.

**New behaviour:**
1. After the preview is approved, always write `backlog.md` (Status: DRAFT) immediately.
2. Then show the gate:
   ```
   backlog.md written. What's next?
   [A] Create tickets in Odoo now
   [B] Run ESTIMATOR first — generate estimates.md, then decide on Odoo
   [C] Skip — keep backlog.md as draft, push to Odoo manually later
   ```
3. [A]: proceed with existing bulk-create flow (no change to that logic).
4. [B]: invoke ESTIMATOR. After ESTIMATOR completes, ESTIMATOR shows its own Odoo gate.
5. [C]: stop. backlog.md is saved as DRAFT.

No other changes to BACKLOG_GENERATOR V1.4.

---

## Inputs

| File | Required | Sections used |
|---|---|---|
| `arch.md` | Yes | `## Effort Signals`, `## Sprint Mapping`, `## Build Order` |
| `backlog.md` | Yes | Ticket list, sprint tags, deadlines, Status field |

If either file is missing → stop with message pointing to the upstream skill to run.

If `backlog.md` has `Status: CREATED` (tickets already pushed to Odoo) → warn:
```
⚠ backlog.md Status: CREATED — tickets already exist in Odoo.
  Estimates can still be generated. Odoo creation gate will be skipped at the end.
  Proceed? (Y/N)
```

---

## Output

**File:** `estimates.md` in the engagement folder root.

**Status field:** `Status: DRAFT` until manually updated.

---

## Skill Flow

### Step 1 — Pre-flight

Silently:
1. Run `basename "$PWD"` → engagement name.
2. Read `arch.md` → validate `## Effort Signals`, `## Sprint Mapping`, `## Build Order` present.
3. Read `backlog.md` → validate ticket list present.
4. Extract project start date from `## Sprint Mapping` (first sprint start date) or from
   earliest deadline in `backlog.md`.

If required sections missing → stop with specific message listing what is absent.

Output one line on success:
```
Engagement: <name> | arch.md + backlog.md loaded. Starting estimation config.
```

### Step 2 — Date confirmation

Show detected date and ask for confirmation:
```
Project start date detected: <date>
Confirm, or enter a new start date (YYYY-MM-DD):
```

If user provides a new date: shift all sprint start/end dates by the same offset
(new_start − original_start). Sprint durations from `## Sprint Mapping` are preserved.
Store confirmed start date for use in Section 2 of estimates.md.

### Step 3 — Estimation config

Ask the following in a single block:

```
Estimation config for <engagement name>:

  1. Estimate style:
     [A] Fixed hours — one value per size (e.g. S=8h)
     [B] Range — low and high per size (e.g. S=4–8h)

  2. Working hours per day: [default: 8]

  3. Include cost estimates? (Y/N)
     If Y: provide rate per role, or one blended day rate.
```

**If Fixed [A]:** Ask once:
```
Enter hours for each size:
  S = ?h   M = ?h   L = ?h   XL = ?h
(Defaults: S=8, M=24, L=40, XL=80 — press Enter to accept)
```

**If Range [B]:** Ask once:
```
Enter hour ranges for each size:
  S = ?–?h   M = ?–?h   L = ?–?h   XL = ?–?h
(Defaults: S=4–8, M=16–32, L=32–48, XL=64–96 — press Enter to accept)
```

**If cost Y:** Ask:
```
Rate input:
  [A] One blended day rate for all roles (e.g. ₹5000/day)
  [B] Per-role rates
```
If [B]: show role list from Team Mapping in backlog.md, ask rate per role.
Store all rates. Used only in estimates.md — no Odoo field is populated with cost.

### Step 4 — Compute estimates

For each Build Order item in `arch.md ## Build Order`:
1. Look up its Effort Signal in `## Effort Signals` (case-insensitive match on feature name).
   - If no match found: flag as `[size unknown — review]` and assign 0h as placeholder.
2. Apply the confirmed size → hours mapping.
3. Convert hours → days: `ceil(hours / working_hours_per_day)`.
4. For ranges: compute low_days and high_days separately.
5. Look up the corresponding ticket in `backlog.md` → extract sprint tag + deadline.
6. Compute start date: sprint start from confirmed project start + sprint offset.
7. If cost: compute `hours × (rate / working_hours_per_day)` per role, or `days × blended_rate`.

### Step 5 — Review table

Show the full computed estimate table. The `Size` column (S/M/L/XL) appears here for
the consultant's reference only — it is never written to estimates.md.

```
Estimate preview — <engagement name> (<N> items):

| # | Feature / Build Order item      | Size | Hours | Days  | Sprint   | Start      | End        | Cost   |
|---|--------------------------------|------|-------|-------|----------|------------|------------|--------|
| 1 | AuthService + PostgreSQL schema | S    | 8h    | 1d    | Sprint 1 | 2026-06-22 | 2026-06-24 | —      |
| 2 | MediaService                    | S    | 8h    | 1d    | Sprint 1 | 2026-06-22 | 2026-06-24 | —      |
...

Total: <N>h / <N>d  |  <start> → <end>  |  Cost: <total or N/A>

Adjust any row? Enter row# and new hours (e.g. "3: 32h"), or type 'done'.
```

- For range-based: Hours column shows `16–32h`, Days shows `2–4d`, Cost shows range.
- `[size unknown — review]` rows shown with `0h` — consultant must enter a value before 'done' is accepted.
- Consultant can adjust multiple rows. Each adjustment is acknowledged inline:
  `Row 3 updated: 40h / 5d`
- When 'done': re-display totals, then proceed.

### Step 6 — Write estimates.md

Write the file immediately. Output:
```
estimates.md written — <N> items, <total hours>h / <total days>d
```

### Step 7 — Odoo gate

If `backlog.md` Status is `CREATED` → skip this gate (tickets already exist).

Otherwise:
```
What's next?
[A] Create tickets in Odoo now
[B] Skip — keep backlog.md as DRAFT, push to Odoo manually later
```

[A]: execute bulk-create flow from BACKLOG_GENERATOR (same MCP calls, same output format).
[B]: stop. Both backlog.md and estimates.md saved as DRAFT.

---

## estimates.md Structure

```markdown
# Estimates — <Engagement Name>
# Generated: <date> | ESTIMATOR V1.5
# Status: DRAFT

---

## Summary

| Field | Value |
|---|---|
| Total effort | <N>h / <N>d  (or <low>–<high>h / <low>–<high>d for range) |
| Timeline | <project start> → <project end> |
| Team size | <N> people |
| Sprints | <N> × <N>-week sprints |
| Cost estimate | <total or range, or N/A> |

---

## Section 1 — Client Summary

Grouped by sprint. One subsection per sprint. Within each sprint, one row per
Build Order item assigned to that sprint.

### Sprint 1 — <start date> → <end date>

| Feature | Hours | Days | Cost |
|---|---|---|---|
| <Build Order item name> | <N>h | <N>d | <cost or —> |

**Sprint 1 total: <N>h / <N>d / <cost or N/A>**

(repeat for each sprint)

---
**Project total: <N>h / <N>d / <cost or N/A>**

---

## Section 2 — Detailed Breakdown

One row per ticket from backlog.md. Matches the Build Order sequence.

| # | Ticket | Sprint | Assignee | Hours | Days | Start | End | Cost |
|---|---|---|---|---|---|---|---|---|
| T01 | <ticket title> | Sprint 1 | <role> | <N>h | <N>d | <date> | <date> | <cost or —> |

**Total: <N>h / <N>d / <cost or N/A>**

---

## Section 3 — Assumptions

- Estimate basis: S=<N>h, M=<N>h, L=<N>h, XL=<N>h
  (or: S=<low>–<high>h, M=..., L=..., XL=...)
- Working hours per day: <N>
- Cost rates: <per-role table or blended rate, or N/A>
- Excludes: QA effort beyond sprint allocation, DevOps setup time,
  client feedback cycles, post-go-live support
```

---

## Readiness Gate

Required sections in arch.md:
- `## Effort Signals` — S/M/L/XL per feature
- `## Sprint Mapping` — sprint dates and owners
- `## Build Order` — ordered list of items

Required in backlog.md:
- Ticket list with sprint tags and deadlines

Optional (used if present):
- `## Integration Points` — noted for awareness; not included in estimates.md output

---

## Error Handling

| Condition | Behaviour |
|---|---|
| arch.md missing | Stop. Message: "Run ARCH_PROPOSER first." |
| backlog.md missing | Stop. Message: "Run BACKLOG_GENERATOR first." |
| Effort Signal missing for a Build Order item | Flag `[size unknown — review]`, assign 0h, require manual entry before 'done' |
| Sprint dates not parseable | Skip date computation for that sprint, flag inline |
| backlog.md Status: CREATED | Warn, allow proceeding, skip Odoo gate at end |

---

## Chain Updates Required

| Skill | Change | Version bump |
|---|---|---|
| BACKLOG_GENERATOR | Write backlog.md before Odoo gate; new 3-option gate | V1.4 → V1.4.1 |
| ESTIMATOR | New skill | V1.5 (new file) |
| START | Add ESTIMATOR to chain registry + `run ESTIMATOR` routing | Minor update |
| CLAUDE.md | Add V1.5 entry to Phase Status + Project Initiator section | Doc update |

---

## Testing

Two synthetic fixtures required:

| Fixture | Path | Scenario |
|---|---|---|
| synthetic-01 (PASS) | tests/fixtures/estimator/synthetic-01/ | Full fixed-hours run, no cost, confirmed dates |
| synthetic-02-range | tests/fixtures/estimator/synthetic-02-range/ | Range-based, cost enabled, date override, one row adjustment |

Each fixture includes: `arch.md`, `backlog.md`, `expected_behaviors.md`, `test_report.md`.
