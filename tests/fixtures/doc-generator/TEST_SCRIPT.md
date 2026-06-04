# DOC_GENERATOR Test Script
# Skill: skills/project-initiator/DOC_GENERATOR.md
# Version: V1.3.5

---

## Fixture Overview

| Fixture | Path | Purpose |
|---|---|---|
| synthetic-01 | tests/fixtures/doc-generator/synthetic-01/ | PASS path — all inputs clean, no DRIFT |
| synthetic-02-drift | tests/fixtures/doc-generator/synthetic-02-drift/ | DRIFT path — backend tech mismatch |

---

## Running synthetic-01 (PASS path)

1. Open Claude Code
2. Set working directory to: `tests/fixtures/doc-generator/synthetic-01/`
3. Say: `run DOC_GENERATOR`
4. When asked for engagement name: type **RetailEdge**
5. Sync check should show: 1 WARN (STRAWMANs), 0 DRIFT
6. At menu: type `all` to generate all 5 docs
7. For each doc: review the draft, type `yes` to save
8. After all 5 docs saved: verify `docs/` folder created with 5 files
9. Score against: `tests/fixtures/doc-generator/synthetic-01/expected_behaviors.md`
10. Clean up: delete `tests/fixtures/doc-generator/synthetic-01/docs/` after testing

---

## Running synthetic-02-drift (DRIFT path)

1. Open Claude Code
2. Set working directory to: `tests/fixtures/doc-generator/synthetic-02-drift/`
3. Say: `run DOC_GENERATOR`
4. When asked for engagement name: type **RetailEdge**
5. Sync check should detect DRIFT: .NET Core vs Node.js + Express
6. Resolve DRIFT by typing: **Node.js + Express** (choosing arch.md as current)
7. Menu should appear after DRIFT resolution
8. Type `5` to generate only the Scope Agreement (it should include "What Changed" section)
9. Review doc and save
10. Score against: `tests/fixtures/doc-generator/synthetic-02-drift/expected_behaviors.md`
11. Clean up: delete `tests/fixtures/doc-generator/synthetic-02-drift/docs/` after testing

---

## Scoring

Record results in a `test_report.md` file in each fixture folder.
Use the format from `tests/fixtures/arch-proposer/synthetic-01/test_report.md`.

Pass threshold: all checkboxes in expected_behaviors.md marked [x].
