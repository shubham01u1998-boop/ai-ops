# MVP_SYNTHESIZER — Test Script
# Version: 1.0 | Created: 2026-06-04
# Paste this entire file as your opening message in a new Claude session.
# The session must have access to the ai-ops repo at its working directory.

---

## Your role in this session

You are a fiftyfive-tech consultant testing the MVP_SYNTHESIZER skill (Project Initiator V1.1).
Your job is to run the skill twice — once on a clean fixture (PASS-path) and once on a broken
fixture (BLOCK-path) — playing the consultant role with the scripted responses below.

After each run, score the skill against the checklist provided. Report the final score as:
  synthetic-01: N/30 | synthetic-02-incomplete: N/20 | Total: N/50

Do not skip items. Mark each as PASS or FAIL with one line of evidence.

---

## How to invoke the skill

At the start of each test run, say exactly:
  run MVP_SYNTHESIZER

The skill will load automatically. You play the consultant from that point.

---

# TEST RUN 1 — synthetic-01 (PASS-path / SupplySync)

**Working directory:** `tests/fixtures/mvp-synthesizer/synthetic-01/`

Before invoking: confirm `discovery.md` exists in that folder (it should — it was placed there).

## Scripted consultant responses

Play these responses in the order the skill asks. Do not volunteer information not asked for.

### Pre-flight
- If skill asks for engagement name (generic folder name detected): say "SupplySync"
- If skill shows "Engagement: SupplySync | discovery.md loaded. Running readiness check." → no response needed, wait

### Readiness Gate
- The gate should show all ✓ entries (no ✗ or ⚠ on this fixture)
- Do NOT respond — wait for the framing question

### Framing Selection
- When asked A/B/C: respond **"A"** (Time-boxed)
- Rationale: SupplySync has a hard December 1 deadline from the CFO

### Feature Prioritization — Batch confirm (HIGH features)
- When the skill batches HIGH-confidence features for bulk confirm:
  respond **"All confirmed as IN"**

### Feature Prioritization — WhatsApp Notifications [MED]
- When asked individually: respond **"IN — it was confirmed as v1 during DISCOVERY"**

### Feature Prioritization — Supplier Rating / Scoring [MED]
- When asked individually: respond **"DEFERRED — nice to have but not critical for launch"**

### Feature Prioritization — Procurement Dashboard [MED]
- When asked individually: respond **"IN — the Head of Procurement needs visibility"**

### User Journey Extraction
- When presented with 2-4 journeys for confirmation: respond **"Looks right, keep these"**

### Success Metrics
- When asked: respond **"Invoice reconciliation time drops from 2-3 days/week to under 4 hours/week. Zero double-order incidents in first 30 days."**

### Draft mvp-scope.md review
- Read the draft carefully
- Respond: **"Looks good, save it"**

---

## Scoring Checklist — synthetic-01 (PASS-path)

Score each item PASS or FAIL. Include one line of evidence for FAIL items.

### Readiness Gate (6 points)
- [ ] 1. Gate runs before any skill logic — summary block shown before framing question
- [ ] 2. Core Problem: ✓ shown in gate summary
- [ ] 3. Users: ✓ shown in gate summary
- [ ] 4. Features: ✓ shown in gate summary (8 found)
- [ ] 5. Timeline: ✓ shown in gate summary (explicit date)
- [ ] 6. No ✗ entries in gate summary — skill proceeds without asking inline questions

### Framing Selection (3 points)
- [ ] 7. All 3 options presented (A, B, C) with "Best when:" hints
- [ ] 8. Skill waits for selection — does not auto-pick
- [ ] 9. Time-boxed framing stored and referenced in prioritization (December 1 deadline visible in reasoning)

### Feature Prioritization (6 points)
- [ ] 10. HIGH-confidence features batched in a single confirm block (not asked one-by-one)
- [ ] 11. WhatsApp Notifications [MED] asked individually (non-obvious)
- [ ] 12. Supplier Rating / Scoring proposed as DEFERRED or OUT (MED, single-source, non-core to problem)
- [ ] 13. Procurement Dashboard asked individually (MED confidence)
- [ ] 14. Dependency rule visible: no IN feature requires an OUT feature
- [ ] 15. Final scope table shows IN / OUT / DEFERRED per feature with rationale

### User Journey Extraction (2 points)
- [ ] 16. 2–4 journeys presented as a single list (not one-by-one)
- [ ] 17. One confirmation prompt — not multiple back-and-forth questions

### Success Metrics (2 points)
- [ ] 18. Asked once with explicit "You can defer this" option
- [ ] 19. Provided metrics appear verbatim in mvp-scope.md (not reworded or summarised away)

### Draft Artifact — Sections (7 points)
- [ ] 20. Problem Restatement present (rewritten, not copy-pasted from discovery.md)
- [ ] 21. MVP Framing section: approach + rationale + driving constraint all present
- [ ] 22. Scope: In table — Feature, Description, Confidence, Rationale columns present
- [ ] 23. Scope: Out table — at least 1 feature (Supplier Rating) listed with "Deferred to" column
- [ ] 24. Key User Journeys section: 2–4 journeys in Actor → action → outcome format
- [ ] 25. Open Questions: carries Budget + ERP from discovery.md (not dropped)
- [ ] 26. Effort Signals section shows deferred notice — NO S/M/L/XL numbers

### Save Gate (2 points)
- [ ] 27. Approval prompt shown after draft: [A] / [B] / [C] options
- [ ] 28. "Looks good, save it" (natural language) interpreted as [A] — file saved without asking again

### Boundary Enforcement (2 points)
- [ ] 29. No architecture decisions made anywhere in the session
- [ ] 30. If skill was asked about tech stack or architecture → redirected to ARCH_PROPOSER

**synthetic-01 subtotal: __ / 30**

---

# TEST RUN 2 — synthetic-02-incomplete (BLOCK-path / RetailEdge)

**Working directory:** `tests/fixtures/mvp-synthesizer/synthetic-02-incomplete/`

Before invoking: confirm `discovery.md` exists. This fixture is deliberately broken:
- No `## Core Problem` section
- No `Timeline:` line in Constraints
- Unresolved CONFLICT in Confidence Notes (Tech frontend: React vs Vue.js)

## Scripted consultant responses

### Pre-flight
- If skill asks for engagement name: say "RetailEdge"
- Wait for gate summary

### Readiness Gate — BLOCK 1: Core Problem missing
- When the skill asks for the core problem: respond **"FreshMart store managers have no unified view of sales, inventory, and staffing — they switch between 3 separate tools and miss stockouts and scheduling conflicts."**

### Readiness Gate — BLOCK 2: Timeline missing
- When the skill asks for timeline: respond **"Q2 2027 soft target, no hard deadline yet"**

### Readiness Gate — BLOCK 3: Unresolved CONFLICT (Tech frontend)
- When the skill presents the React vs Vue.js conflict: respond **"Vue.js — the CTO email is more recent and authoritative"**

### After gate clears — Framing Selection
- When asked A/B/C: respond **"C"** (Value-first)
- Rationale: deadline is flexible ("Q2 2027 soft target"), value delivery to store managers is the priority

### Feature Prioritization — Batch confirm (HIGH features)
- When batched HIGH features presented: respond **"All IN"**

### Feature Prioritization — Staff Scheduling [MED]
- When asked individually: respond **"IN — it's a daily use case for managers"**

### Feature Prioritization — Supplier Reorder [LOW]
- When asked individually: respond **"DEFERRED — low confidence, single mention"**

### Feature Prioritization — Mobile App [MED]
- When asked: respond **"IN but defer platform decision — iOS first"**

### User Journey Extraction
- When journeys presented: respond **"Add one more: Store manager checks expiry alerts → removes items from shelf → logs disposal. Otherwise looks right."**

### Success Metrics
- When asked: respond **"defer"**

### Draft review
- Respond: **"Save it"**

---

## Scoring Checklist — synthetic-02-incomplete (BLOCK-path)

### Readiness Gate — BLOCK detection (6 points)
- [ ] 1. Gate runs and shows at least 3 ✗ entries (Core Problem, Timeline, Tech CONFLICT)
- [ ] 2. Core Problem: ✗ BLOCK shown — section absent from discovery.md
- [ ] 3. Timeline: ✗ BLOCK shown — line missing entirely from Constraints
- [ ] 4. Tech CONFLICT: ✗ BLOCK shown — "CONFLICT" line present without "resolved"
- [ ] 5. Supplier Reorder LOW: ⚠ WARN shown (not a BLOCK — feature is present, just LOW confidence)
- [ ] 6. Skill does NOT proceed to Framing until all BLOCKs are resolved

### Readiness Gate — Inline BLOCK resolution (6 points)
- [ ] 7. BLOCK 1 (Core Problem): skill asks for it with one question — not multi-part
- [ ] 8. BLOCK 2 (Timeline): asked as a separate question after Core Problem resolved
- [ ] 9. BLOCK 3 (Tech CONFLICT): skill names both sides ("spec says React, CTO email says Vue.js") and asks which is current
- [ ] 10. BLOCKs asked one at a time (not all in one message)
- [ ] 11. After all 3 BLOCKs resolved: gate clears and framing question appears
- [ ] 12. Resolved Core Problem appears in mvp-scope.md Problem Restatement

### Post-gate skill behavior (4 points)
- [ ] 13. Supplier Reorder [LOW] proposed as DEFERRED or OUT in prioritization (LOW confidence → default DEFERRED)
- [ ] 14. "defer" response to Success Metrics → Open Questions (not auto-generated)
- [ ] 15. "Save it" (natural language) interpreted as [A] — mvp-scope.md written
- [ ] 16. Confidence Notes in mvp-scope.md flags Supplier Reorder LOW + any WARN items

### mvp-scope.md output quality (4 points)
- [ ] 17. Problem Restatement reflects the inline answer (not blank, not "missing")
- [ ] 18. Timeline shows "Q2 2027 soft target" (from inline resolution)
- [ ] 19. Tech constraint shows Vue.js (conflict resolved to Vue.js)
- [ ] 20. Open Questions includes Success Metrics (deferred) and any other deferrals

**synthetic-02-incomplete subtotal: __ / 20**

---

# Final Report Format

Paste this block at the end of your session with results filled in:

```
MVP_SYNTHESIZER TEST REPORT
Date: 2026-06-04
Tester: Claude [session ID or model]

synthetic-01 (SupplySync / PASS-path):   __ / 30
synthetic-02-incomplete (RetailEdge / BLOCK-path): __ / 20
Total: __ / 50

FAILED ITEMS:
- [item number] [PASS-path or BLOCK-path]: [what happened vs what was expected]
(leave blank if all passed)

OBSERVATIONS:
- [anything surprising, unexpected, or worth flagging for V1.1 iteration]

VERDICT: PASS (50/50 or minor failures) / NEEDS FIXES (list items)
```
