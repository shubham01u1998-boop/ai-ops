# Expected Behaviors — MVP_SYNTHESIZER / synthetic-01 (SupplySync)
# PASS-path fixture: clean discovery.md, all gate fields present, no unresolved conflicts
# Run MVP_SYNTHESIZER from this folder (discovery.md present). Play consultant role.

---

## Readiness Gate

- [ ] Gate runs before any skill logic — not skipped
- [ ] Core Problem: ✓ PASS — section present, HIGH confidence
- [ ] Users: ✓ PASS — Primary users present
- [ ] Features: ✓ PASS — 8 features listed
- [ ] Timeline: ✓ PASS — "December 1, 2026" is an explicit date
- [ ] Conflicts: ✓ PASS — both CONFLICT lines in Confidence Notes contain "resolved"
- [ ] Gate summary block shown with all ✓ (no ⚠ or ✗ on this fixture)
- [ ] Skill proceeds immediately after gate — no inline questions asked

---

## Framing Selection

- [ ] All 3 framing options presented (A: Time-boxed, B: Risk-first, C: Value-first)
- [ ] Each option shows a "Best when:" hint
- [ ] Skill waits for selection — does NOT auto-pick a framing
- [ ] Natural language response ("deadline-driven") accepted and confirmed before proceeding
- [ ] Chosen framing stored and referenced in prioritization

---

## Feature Prioritization

- [ ] Internal draft built silently before presenting to consultant
- [ ] HIGH-confidence features batched in a single "confirm or edit" block (not asked one-by-one)
  — Supplier Onboarding, Order Management, Contract Management, Invoice Reconciliation, Email Notifications all HIGH
- [ ] WhatsApp Notifications [MED] presented individually (non-obvious given MED confidence)
- [ ] Supplier Rating / Scoring [MED — CFO email only] proposed as DEFERRED (MED, single-source, non-core)
- [ ] Procurement Dashboard [MED — meeting notes only] questioned individually
- [ ] At least 1 feature ends up DEFERRED in the final scope
- [ ] Dependency rule applied: no feature marked IN that requires an OUT feature
- [ ] "Defer all remaining" accepted mid-loop — remaining features move to DEFERRED

---

## User Journey Extraction

- [ ] 2–4 journeys presented as a single list (not one-by-one)
- [ ] Journeys based on IN features + Primary users (procurement managers)
- [ ] One confirmation prompt — consultant can edit, add, or remove in one response
- [ ] Each journey format: <Actor> → <action> → <outcome> — [feature(s) required]

---

## Success Metrics

- [ ] Asked once with "You can defer this" option
- [ ] If consultant defers: goes to Open Questions — not auto-generated
- [ ] If consultant provides metrics: appears in mvp-scope.md Success Metrics section verbatim

---

## "Stop" Handling

- [ ] If consultant says "stop" after framing but before journeys: skill skips to draft immediately
- [ ] Skipped sections appear as [INCOMPLETE — consultant stopped before this section]
- [ ] Save gate still applies — approval required before writing

---

## Draft mvp-scope.md

- [ ] All required sections present:
  - [ ] Problem Restatement (rewritten from discovery.md, not copy-pasted)
  - [ ] Users (Primary + Secondary)
  - [ ] MVP Framing (approach + rationale + driving constraint)
  - [ ] Scope: In (table with Feature, Description, Confidence, Rationale)
  - [ ] Scope: Out (table with Feature, Why Out, Deferred to)
  - [ ] Key User Journeys (2–4 journeys)
  - [ ] Success Metrics
  - [ ] Constraints (Timeline, Budget, Tech, Other — inherited from discovery.md)
  - [ ] Assumptions (at least 1 explicit assumption)
  - [ ] Open Questions (carries forward Budget + ERP from discovery.md; new ones added if any)
  - [ ] Effort Signals (deferred notice — NOT fabricated S/M/L/XL values)
  - [ ] Confidence Notes (Budget: LOW flagged; WARNs flagged)
  - [ ] Source Artifacts (discovery.md referenced)

- [ ] Scope: In — Rationale column sufficient to draft SOW deliverables list
- [ ] Scope: Out — Deferred to column identifies Phase 2 / TBD clearly
- [ ] Open Questions sufficient to brief ARCH_PROPOSER without re-running DISCOVERY
- [ ] Budget: LOW flagged in Confidence Notes (carried from discovery.md)
- [ ] Effort Signals section shows deferred notice, not S/M/L/XL numbers

---

## Save Gate

- [ ] Approval prompt shown after draft: [A] Yes / [B] Edit / [C] Add context
- [ ] Natural language approval ("yes", "save it", "looks good") interpreted as [A]
- [ ] Edit instruction interpreted as [B] — change applied, affected section re-shown, prompt re-offered
- [ ] Ambiguous response surfaces intent before saving
- [ ] mvp-scope.md NOT written until explicit approval received

---

## Boundary Enforcement

- [ ] If asked about tech stack → "That's handled by ARCH_PROPOSER — MVP_SYNTHESIZER focuses on scope only."
- [ ] If asked to create tickets → "That's handled by BACKLOG_GENERATOR — a later skill."
- [ ] No architecture decision made anywhere in the session
- [ ] No effort numbers produced (S/M/L/XL deferred to ARCH_PROPOSER)
