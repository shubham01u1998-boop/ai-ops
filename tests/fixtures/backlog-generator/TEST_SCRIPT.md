# BACKLOG_GENERATOR Test Script
# Skill: skills/project-initiator/BACKLOG_GENERATOR.md
# Version: V1.4

---

## Fixture Overview

| Fixture | Path | Purpose |
|---|---|---|
| synthetic-01 | tests/fixtures/backlog-generator/synthetic-01/ | PASS path — new project, default stages, no duplicates |
| synthetic-02-duplicate | tests/fixtures/backlog-generator/synthetic-02-duplicate/ | DUPLICATE path — existing project + 2 duplicate tickets |

---

## Running synthetic-01 (PASS path)

1. Open Claude Code
2. Set working directory to: `tests/fixtures/backlog-generator/synthetic-01/`
3. Say: `run BACKLOG_GENERATOR`
4. When asked for engagement name: type **RetailEdge**
5. Project setup: accept defaults (type `yes`)
6. Team mapping: enter Odoo user IDs for each role (or `skip`)
7. Duplicate check: should show no duplicates (new project) — proceeds silently
8. Preview: review the ticket hierarchy table, type `yes` to approve
9. Bulk creation: confirm progress messages shown
10. Score against: `tests/fixtures/backlog-generator/synthetic-01/expected_behaviors.md`
11. Clean up: no local files to delete (Odoo tickets created in Odoo, not locally)

---

## Running synthetic-02-duplicate (DUPLICATE path)

1. Open Claude Code
2. Set working directory to: `tests/fixtures/backlog-generator/synthetic-02-duplicate/`
3. Say: `run BACKLOG_GENERATOR`
4. When asked for engagement name: type **RetailEdge**
5. Project setup: accept defaults
6. Existing project check: **RetailEdge** already exists → type `A` to use existing
7. Team mapping: enter user IDs
8. Duplicate check: should show 2 duplicate tickets → type `A` to skip duplicates
9. Preview: should show tickets 1 and 2 marked [EXISTS]
10. Approve and confirm only 3 parent tickets are created
11. Score against: `tests/fixtures/backlog-generator/synthetic-02-duplicate/expected_behaviors.md`

---

## Scoring

Record results in a `test_report.md` file in each fixture folder.
Use the format from `tests/fixtures/arch-proposer/synthetic-01/test_report.md`.

Pass threshold: all checkboxes in expected_behaviors.md marked [x].
