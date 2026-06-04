# Expected Behaviors — synthetic-01 (SupplySync)
# Run DISCOVERY from the synthetic-01/ directory with these 3 input docs.
# All items below must pass before this fixture is considered verified.

## Pre-flight
- [ ] DISCOVERY reads all 3 input files: rough-spec.md, meeting-notes-1.md, stakeholder-email.txt
- [ ] Reports "Read 3 input doc(s)" before starting extraction
- [ ] Does NOT skip any of the 3 files (all are .md or .txt)

## Extraction
- [ ] Extracts project type as web platform / supplier management (HIGH confidence)
- [ ] Extracts primary users as procurement managers + procurement head (HIGH/MED)
- [ ] Extracts core problem: Excel+WhatsApp manual workflow, no audit trail (HIGH)
- [ ] Extracts at least 5 features: supplier onboarding, order management, contract management, invoice reconciliation, notifications (HIGH)
- [ ] Extracts supplier rating/scoring feature from stakeholder-email (MED)
- [ ] Extracts WhatsApp notification as a FEATURE REQUIREMENT (MED) — distinct from WhatsApp in rough-spec problem context. Source: meeting-notes-1.md (Rohan's request). NOT the same as "replace Excel + WhatsApp workflow."

## Conflict Detection
- [ ] Surfaces tech conflict: rough-spec says React, meeting-notes says Vue.js — asks consultant to resolve
- [ ] Surfaces timeline conflict: rough-spec says Q3 2026, meeting-notes says December, stakeholder-email says December 1 — asks consultant to resolve
- [ ] Does NOT silently pick one side of either conflict

## Gaps — Critical
- [ ] Asks about budget (mentioned as unknown in meeting-notes, sensitive per email)
- [ ] Asks about WhatsApp integration (not in spec but mentioned in meeting notes AND would affect architecture)
- [ ] Asks about data residency requirement (India-only — hard constraint from CFO email, affects hosting decisions)

## Gaps — Detail
- [ ] Asks about ERP integration timeline or asks if SAP B1 is confirmed for Phase 2 (mentioned by Rohan)
- [ ] Asks about language support for WhatsApp messages (Hindi/Marathi mentioned in meeting notes)

## Confidence Markers
- [ ] At least 1 extraction category marked LOW confidence (budget is a natural candidate)
- [ ] Confidence markers visible in initial summary block

## Open Questions
- [ ] At least 2 open questions captured in discovery.md output (deferred items from question loop)
- [ ] Budget listed as open question (CFO said to discuss in separate call)

## Output Quality
- [ ] Produces discovery.md with all required sections: Project Context, Users, Core Problem, Features Mentioned, Constraints, Open Questions, Confidence Notes, Source Docs
- [ ] Does NOT invent information not present in the input docs
- [ ] Does NOT recommend a tech stack (that is ARCH_PROPOSER's job)
- [ ] Does NOT scope the MVP (that is MVP_SYNTHESIZER's job)
- [ ] Supplier rating feature is in the output (from CFO email, easy to miss if doc isn't fully read)
- [ ] Data residency constraint (India) is in the output Constraints section

## Conversation Pattern
- [ ] Questions are asked one at a time (not all at once)
- [ ] Batching used only for tightly-coupled questions (e.g. scale questions together)
- [ ] "Defer" responses are accepted without pushback and moved to Open Questions
