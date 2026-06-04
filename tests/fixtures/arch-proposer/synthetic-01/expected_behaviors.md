# Expected Behaviors — synthetic-01 (RetailEdge PASS-path)
# Tests ARCH_PROPOSER against a clean mvp-scope.md with all gate fields present.

## Readiness Gate
- [ ] Gate runs without being asked
- [ ] All 6 fields PASS: Scope:In ✓, Users ✓, Tech ✓, Timeline ✓, Budget ✓, No CONFLICTs ✓
- [ ] Gate output shown as a block before any conversation
- [ ] No BLOCKs triggered

## Stack Selection
- [ ] Confirmed tech shown as locked (React ✓, Azure ✓)
- [ ] Only unconfirmed layers presented (backend, database, infra)
- [ ] Each layer shows 2–3 options with trade-offs
- [ ] Recommendation stated with rationale
- [ ] One layer at a time — backend before database before infra
- [ ] Consultant can pick A/B/C or propose own option

## Component Design
- [ ] Components derived for Sales Dashboard and Inventory Alerts
- [ ] Shared components (e.g. OracleConnector) identified and flagged as shared
- [ ] Data model hints included (table names + key fields)
- [ ] Presented as single block, one confirmation prompt
- [ ] Consultant can add/rename/split/remove components

## Integration Points
- [ ] Oracle Retail POS surfaced as integration point
- [ ] Offline mode surfaced as integration point
- [ ] Risk ratings present (HIGH/MED/LOW)
- [ ] STRAWMAN marker on Oracle POS (integration method unconfirmed)
- [ ] Open Questions captured for unknowns
- [ ] One confirmation block

## Build Order + Sprint Mapping
- [ ] Team composition asked before sprint mapping
- [ ] Technical dependency order shown (e.g. OracleConnector first)
- [ ] Sprint mapping shown in table: Sprint | Work | Owner
- [ ] Seniority reflected in assignments (senior = complex/integration components)
- [ ] Timeline (2026-06-04 → 2026-09-04) drives sprint count
- [ ] [STRAWMAN] on sprint split if assumptions made
- [ ] One confirmation block

## Effort Signals
- [ ] S/M/L/XL per feature shown in table
- [ ] Rationale column present
- [ ] OracleConnector (HIGH risk) auto-flagged as STRAWMAN
- [ ] Consultant confirms or adjusts sizes

## Draft arch.md
- [ ] Client Summary section present and non-technical
- [ ] Client Summary generated after all technical sections
- [ ] Tech Stack table with confirmed/STRAWMAN status
- [ ] Components section per feature with shared components flagged
- [ ] Data Model Hints section
- [ ] Integration Points table
- [ ] Build Order numbered list
- [ ] Sprint Mapping table with team assignments
- [ ] Effort Signals table
- [ ] Open Questions section
- [ ] STRAWMAN Summary section listing all tentative decisions
- [ ] Confidence Notes section
- [ ] Source Artifacts section
- [ ] Save gate prompt shown before writing file

## Save
- [ ] arch.md written only after explicit consultant approval
- [ ] arch.md written to current working directory
- [ ] Completion message shown with "Next: run BACKLOG_GENERATOR"
