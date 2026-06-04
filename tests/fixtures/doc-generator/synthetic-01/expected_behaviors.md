# Expected Behaviors — synthetic-01 (RetailEdge PASS-path)
# Tests DOC_GENERATOR against clean inputs: discovery.md + mvp-scope.md + arch.md all present, no DRIFT.
# Run from: tests/fixtures/doc-generator/synthetic-01/
# When asked for engagement name, answer: RetailEdge

## Pre-flight
- [ ] All 3 files loaded silently
- [ ] One-liner output: "Engagement: RetailEdge | discovery.md ✓ | mvp-scope.md ✓ | arch.md ✓. Running sync check."

## Sync Check
- [ ] Sync check runs before menu — never skipped
- [ ] Features → Components: PASS (Sales Dashboard and Inventory Alerts each have components)
- [ ] Tech constraints: PASS (React and Azure consistent across mvp-scope.md and arch.md)
- [ ] Timeline: PASS (2026-09-04 consistent in both files)
- [ ] Budget: PASS (₹18 lakhs present in mvp-scope.md)
- [ ] STRAWMANs: WARN — 5 items flagged (listed in gate output)
- [ ] Gate output shown as a block: ✓/✓/✓/✓/⚠ format
- [ ] "All clear — 1 WARN, no DRIFT. Proceeding to document menu." shown
- [ ] No DRIFT resolution questions asked

## Document Menu
- [ ] Menu shows all 5 docs with audience labels [Client] / [Dev team] / [Dev + PM]
- [ ] Shorthand aliases shown: "all" / "client" / "dev"
- [ ] Consultant can select by number, shorthand, or natural language
- [ ] After selection, confirmation line shown before generation begins
- [ ] Docs generated in order selected

## Doc 1 — Project Proposal / SOW
- [ ] All 10 sections present: Executive Summary, MVP Scope, Key User Journeys, High-Level Architecture, Delivery Timeline, Budget, Risks & Assumptions, Success Metrics, Team, Sign-off block
- [ ] No component names or framework names in client-facing text
- [ ] 1 Mermaid Gantt diagram present (delivery timeline from arch.md sprint mapping)
- [ ] Budget shows ₹18 lakhs (from mvp-scope.md)
- [ ] STRAWMANs from arch.md rewritten in business language (not "Node.js + Express", not "[STRAWMAN]")
- [ ] Save gate prompt shown: "Looks complete — save proposal.md to docs/? [A] Yes / [B] Edit a section / [C] Add more context"

## Doc 2 — Technical Architecture Doc
- [ ] All 9 sections present: Overview, Tech Stack, Architecture Diagram, Component Inventory, Data Model, Integration Points, Build Order, Open Questions, STRAWMAN Summary
- [ ] Mermaid graph diagram (component architecture) present
- [ ] Mermaid erDiagram (data model) present
- [ ] Mermaid sequence diagram (integration flows) present
- [ ] STRAWMAN flags preserved on tentative items
- [ ] Open Questions list each blocker's affected component
- [ ] Save gate prompt shown for this doc

## Doc 3 — Sprint Plan
- [ ] All 5 sections present: Team Roster, Sprint Calendar, Sprint Breakdown, Dependency Map, Risk Flags per Sprint
- [ ] Mermaid Gantt diagram present (detailed sprint calendar with owners)
- [ ] Team roster includes seniority (5y, 6y etc.)
- [ ] Sprint Breakdown table has: sprint number, dates, deliverables, owner
- [ ] Dependency map shows build-order dependencies as a table
- [ ] Risk flags tie STRAWMAN items to specific sprints
- [ ] Save gate prompt shown for this doc

## Doc 4 — Developer Handoff Doc
- [ ] All 8 sections present: Engagement Context, Tech Stack, Component Inventory, Data Model, Integration Flows, Build Order, Open Questions, STRAWMAN Checklist
- [ ] Mermaid erDiagram present (data model)
- [ ] Mermaid flowchart present (integration flows)
- [ ] Component Inventory includes per-component: tech, purpose, dependencies, open Qs
- [ ] STRAWMAN Checklist uses checkbox (- [ ]) format
- [ ] Engagement context is 3-4 sentences max
- [ ] Save gate prompt shown for this doc

## Doc 5 — Scope Agreement
- [ ] All 10 sections present: Problem Statement, Users, MVP Scope In/Out, Key User Journeys, Delivery Timeline, Tech & Infra Constraints, Assumptions, Pre-conditions for Start, Success Metrics, Sign-off block
- [ ] "What Changed Since MVP Scope" section NOT present (no DRIFT in this fixture)
- [ ] Mermaid Gantt present (delivery timeline)
- [ ] Pre-conditions include resolved items from arch.md Open Questions marked ✓ where resolved
- [ ] Sign-off block has client name placeholder + fiftyfive technologies
- [ ] Client language — no component names, no framework names
- [ ] Save gate prompt shown for this doc

## Save
- [ ] docs/ subfolder created if not already present — silently
- [ ] Files written to <CWD>/docs/ — not to the skill folder or anywhere else
- [ ] Each file written only after explicit approval
- [ ] Completion message shown after last doc saved
- [ ] Completion message includes list of saved filenames
- [ ] "Next: run BACKLOG_GENERATOR" pointer in completion message

## Test Notes
- **Basename behavior:** Fixture folder is `synthetic-01` — generic name. DOC_GENERATOR will ask for engagement name. Answer: **RetailEdge**. This is correct behavior; not a defect.
- **Run from:** `tests/fixtures/doc-generator/synthetic-01/` — all 3 input files are present there. The `docs/` folder will be created there during the test run — delete it after testing.
