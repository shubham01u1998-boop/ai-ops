# Expected Behaviors — synthetic-02-drift (RetailEdge DRIFT-path)
# Tests DOC_GENERATOR sync check DRIFT detection and inline resolution.
# DRIFT: mvp-scope.md confirms .NET Core backend; arch.md has Node.js + Express.
# Run from: tests/fixtures/doc-generator/synthetic-02-drift/
# When asked for engagement name, answer: RetailEdge

## Pre-flight
- [ ] All 3 files loaded silently
- [ ] One-liner output: "Engagement: RetailEdge | discovery.md ✓ | mvp-scope.md ✓ | arch.md ✓. Running sync check."

## Sync Check — DRIFT Detection
- [ ] Sync check runs before menu — never skipped
- [ ] Features → Components: PASS
- [ ] Tech constraints: DRIFT detected — mvp-scope.md has .NET Core confirmed; arch.md has Node.js + Express
- [ ] Timeline: PASS
- [ ] Budget: PASS
- [ ] STRAWMANs: WARN
- [ ] Gate output shows ✗ on Tech constraints row with DRIFT label
- [ ] "Cannot proceed — resolve DRIFT items before generating documents." or equivalent shown
- [ ] Inline DRIFT question asked immediately: references both files and states the conflict explicitly

## DRIFT Resolution
- [ ] DRIFT question is specific: names the conflicting values and which files they came from
- [ ] Consultant can resolve (pick one) or defer (becomes WARN)
- [ ] If resolved: skill updates its working state; does NOT modify mvp-scope.md or arch.md
- [ ] If deferred: DRIFT downgraded to WARN; affected doc (Tech Arch) will show a placeholder note
- [ ] After DRIFT resolved/deferred: sync check output re-shown with updated status
- [ ] Menu appears only after all DRIFTs resolved or deferred

## Post-Resolution Menu
- [ ] Menu shown correctly after DRIFT resolved
- [ ] If consultant picks Doc 2 (Tech Arch): resolved/deferred backend decision is used in the doc

## Scope Agreement — "What Changed" Section
- [ ] If Scope Agreement selected and DRIFT was found: "What Changed Since MVP Scope" section present
- [ ] "What Changed" section lists the tech constraint DRIFT and how it was resolved
- [ ] If DRIFT was deferred: "What Changed" notes it as unresolved, pending confirmation

## Test Notes
- This fixture tests the DRIFT path only. Full doc generation behaviors verified in synthetic-01.
- Run from the synthetic-02-drift folder (has all 3 files locally).
- A docs/ folder will be created here during the test run — delete it after.
