# Expected Behaviors — synthetic-02-incomplete (StyleMart BLOCK-path)
# Tests ARCH_PROPOSER Readiness Gate against a deliberately incomplete mvp-scope.md.
# Expected: 3 BLOCKs — missing Budget, missing Timeline, unresolved CONFLICT

## Readiness Gate
- [ ] Gate runs without being asked
- [ ] Scope:In — PASS (2 features present)
- [ ] Users — PASS (Primary present)
- [ ] Tech Constraints — PASS (present, even though CONFLICT)
- [ ] Timeline — BLOCK (Timeline: line missing entirely)
- [ ] Budget — BLOCK (Budget: line missing entirely)
- [ ] Unresolved CONFLICT — BLOCK ("Backend: CONFLICT — ... Not resolved.")
- [ ] Gate output block shown with all 3 BLOCKs marked ✗

## BLOCK Resolution (one at a time)
- [ ] First BLOCK asked: Timeline (or Budget — whichever comes first in gate order)
- [ ] Second BLOCK asked after first is resolved or deferred
- [ ] Third BLOCK asked after second is resolved or deferred
- [ ] Consultant can defer any BLOCK → becomes WARN + Open Question
- [ ] Gate clears after all BLOCKs resolved or deferred
- [ ] Skill proceeds to Stack Selection after gate clears

## Post-Gate
- [ ] Skill continues normally after gate clears
- [ ] Deferred BLOCKs appear as WARNs in arch.md Confidence Notes
- [ ] Deferred CONFLICT appears in STRAWMAN Summary
