# Expected Behaviors — MVP_SYNTHESIZER / synthetic-02-incomplete (RetailEdge)
# BLOCK-path fixture: missing Core Problem section + unresolved CONFLICT in Confidence Notes
# Purpose: verify the Readiness Gate actually blocks and asks inline to resolve
# Run MVP_SYNTHESIZER from this folder (discovery.md present). Play consultant role.

---

## What makes this fixture incomplete (intentional gaps)

1. **Core Problem section missing** — no `## Core Problem` section in discovery.md → gate BLOCKS
2. **Unresolved CONFLICT** — Confidence Notes contains "CONFLICT" line for Tech frontend that does NOT contain "resolved" → gate BLOCKS
3. **Timeline missing entirely** — no Timeline line in Constraints → gate BLOCKS
4. **Bonus WARN** — Supplier Reorder feature is LOW confidence → gate should flag as WARN (not BLOCK)

---

## Readiness Gate

- [ ] Gate runs before any skill logic — not skipped
- [ ] Core Problem: ✗ BLOCK — `## Core Problem` section is absent from discovery.md
- [ ] Users: ✓ PASS — Primary users present
- [ ] Features: ✓ PASS — 5 features listed (≥2)
- [ ] Timeline: ✗ BLOCK — no Timeline line in ## Constraints section
- [ ] Conflicts: ✗ BLOCK — Confidence Notes contains "CONFLICT" line without "resolved"
- [ ] Gate summary block shows at least 3 ✗ entries (Core Problem, Timeline, Conflict)
- [ ] Gate does NOT proceed to Framing Selection until all BLOCKs resolved

---

## Inline BLOCK Resolution (gate asks one at a time)

**BLOCK 1: Core Problem missing**
- [ ] Gate asks for core problem statement before proceeding
- [ ] Example: "## Core Problem is missing from discovery.md. What's the central problem or business need RetailEdge is solving?"
- [ ] Consultant answers → gate stores it, marks BLOCK resolved
- [ ] Consultant says "defer" → becomes WARN + Open Question; gate proceeds to next BLOCK

**BLOCK 2: Timeline missing**
- [ ] Gate asks for timeline before proceeding to next BLOCK
- [ ] Consultant answers → gate stores it, marks BLOCK resolved
- [ ] Consultant says "defer" → WARN; if consultant later picks time-boxed framing, skill asks for date again before prioritization

**BLOCK 3: Unresolved CONFLICT (Tech frontend)**
- [ ] Gate presents the conflict: "Confidence Notes shows an unresolved CONFLICT on Tech frontend — spec says React, CTO email says Vue.js. Which is current?"
- [ ] Consultant answers → gate marks conflict resolved, updates internal extraction
- [ ] Consultant says "defer" → becomes WARN + Open Question; proceeds

---

## After Gate Clears

- [ ] Once all BLOCKs resolved or deferred, gate proceeds normally
- [ ] Framing Selection presented as in synthetic-01
- [ ] Feature prioritization accounts for LOW confidence Supplier Reorder (should be DEFERRED or OUT by default)
- [ ] Mobile App [MED] questioned individually (platform decision missing — iOS/Android/both?)
- [ ] WARN items (Supplier Reorder LOW, any deferred BLOCKs) appear in mvp-scope.md Confidence Notes

---

## mvp-scope.md Output

- [ ] Core Problem section reflects consultant's inline answer (not left blank)
- [ ] Timeline reflects consultant's inline answer OR shows "deferred" if not provided
- [ ] Tech frontend conflict resolution reflected in Constraints section
- [ ] Confidence Notes flags all WARNs explicitly
- [ ] Open Questions includes any BLOCKs the consultant deferred
- [ ] All other sections present per standard format

---

## Gate Boundary Enforcement

- [ ] Gate never auto-fills a BLOCK — always asks
- [ ] Gate asks one BLOCK question at a time (not all at once)
- [ ] Gate does not show the framing options until all BLOCKs are cleared
- [ ] WARNs (Supplier Reorder LOW) do not block — gate proceeds with WARN flagged
