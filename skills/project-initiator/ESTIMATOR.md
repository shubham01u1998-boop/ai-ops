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
`estimates.md` — always converted to hours/days before writing.

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

Ask in a single block — wait for one response covering all three:

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
If [B]: display role list from Team Mapping in backlog.md. For each role ask:
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
