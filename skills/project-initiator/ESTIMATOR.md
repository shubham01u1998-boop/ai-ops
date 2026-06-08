# VERSION: 1.5 | Last updated: 2026-06-08 | Reviewed: pending
# ESTIMATOR — Project Initiator V1.5
# Part of the fiftyfive-tech Project Initiator toolchain.

---

## Purpose

ESTIMATOR reads `arch.md` and `backlog.md` from the engagement folder, converts S/M/L/XL
effort signals to hours/days, maps estimates to sprint dates, and generates `estimates.md` —
a structured estimation document with a Summary block and three sections:
- **Summary:** Key metrics (effort, timeline, team size, cost)
- **Section 1:** Executive summary for the client (grouped by sprint → feature)
- **Section 2:** Detailed breakdown for the internal team or senior reviewer
- **Section 3:** Assumptions and exclusions

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

Store working hours per day from question 2 as `working_hours_per_day` (default 8 if Enter pressed).

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
   - If no parent ticket match is found in backlog.md: flag inline as
     `[backlog entry missing — review]`; set sprint = "—", assignee = "—",
     start = "—", end = "—".

5. **Compute cost (only if `include_cost = true`):**
   - Fixed + blended: `cost = days × blended_rate`
   - Fixed + per-role: `cost = days × role_rates[assignee_role]`
     (if role not in role_rates: show "—" for cost and flag inline)
   - Range + blended: `low_cost = low_days × blended_rate`,
                       `high_cost = high_days × blended_rate`
   - Range + per-role: same logic with `role_rates[assignee_role]`

Store results as ordered list matching Build Order sequence:
`estimates = [{ item, size, hours, days, sprint, assignee, start, end, cost }]`

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
per Build Order item assigned to that sprint. If an item spans multiple sprints
(e.g. "Sprint 1-2"), place it under the section for the sprint in which it starts.

### Sprint 1 — <sprint start date> → <sprint end date>

| Feature | Hours | Days | Cost |
|---|---|---|---|
| <Build Order item name> | <N>h | <N>d | <cost or —> |

**Sprint 1 total: <N>h / <N>d / <cost or N/A>**

(repeat for each sprint)

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
