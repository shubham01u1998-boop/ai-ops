# Expected Behaviors — synthetic-02 (StyleMart)
# Run DISCOVERY from the synthetic-02/ directory with these 4 input docs.
# All items below must pass before this fixture is considered verified.
# Stress-test angles vs synthetic-01: 4 docs, messier/informal notes, resolved conflict in latest doc, India-specific requirements.

## Pre-flight
- [ ] DISCOVERY reads all 4 input files: rough-notes.md, product-brief.txt, dev-team-email.md, client-followup.txt
- [ ] Reports "Read 4 input doc(s)" before starting extraction
- [ ] Does NOT skip any of the 4 files (all are .md or .txt)

## Extraction
- [ ] Extracts project type as branded e-commerce / online retail platform [HIGH]
- [ ] Extracts primary users as end customers + admin staff [HIGH]
- [ ] Extracts core problem: manual Instagram DM + UPI QR code workflow, no order tracking [HIGH]
- [ ] Extracts at least 6 features: product catalog, cart/checkout, order tracking, customer accounts, returns management, loyalty points, notifications [HIGH/MED]
- [ ] Extracts COD as confirmed essential requirement (~60–70% of orders) [HIGH — client-followup]
- [ ] Extracts loyalty points with specific rules (1pt/₹100, 100pts=₹50, 12mo expiry) [HIGH — client-followup]

## Conflict Detection
- [ ] Surfaces tech conflict: product-brief/rough-notes say Shopify; dev-team-email says custom React+Node.js — asks consultant to resolve
- [ ] Surfaces mobile app conflict: product-brief says Phase 2; dev-team-email says Phase 1; client-followup says web+app together — asks consultant to resolve
- [ ] Surfaces timeline conflict: product-brief says Q3 2026 soft launch (July–September); client-followup says end of October 2026 (before Diwali Oct 20) — these are incompatible
- [ ] Multi-vendor is handled correctly — either (a) not asked (client-followup resolves it), OR (b) if asked, captured as "single brand only" in output. NOT ignored entirely.
- [ ] Does NOT silently pick one side of any unresolved conflict

## Gaps — Critical
- [ ] Asks about payment gateway vendor (all docs say TBD or "open to suggestions")
- [ ] Asks about budget (explicitly withheld pending NDA + CFO call)
- [ ] Asks about Tally/inventory system — client has existing Tally setup; no integration plan exists but order management depends on inventory source

## Gaps — Detail
- [ ] Asks about return/exchange workflow details (feature listed but no process defined)
- [ ] Asks about catalog/SKU size (affects architecture and performance planning)

## Confidence Markers
- [ ] At least 1 extraction category marked LOW (budget is primary candidate; tech also LOW due to conflict)
- [ ] Confidence markers visible in initial summary block

## Open Questions
- [ ] At least 2 open questions in discovery.md output
- [ ] Budget listed as open question (CFO, NDA required)
- [ ] Payment gateway vendor listed as open question (or captured as "Razorpay / TBD" if consultant answers)

## Output Quality
- [ ] Produces discovery.md with all required sections: Project Context, Users, Core Problem, Features Mentioned, Constraints, Open Questions, Confidence Notes, Source Docs
- [ ] Does NOT invent information not present in input docs or consultant answers
- [ ] Does NOT recommend a tech stack (surface options and conflicts only — ARCH_PROPOSER's job)
- [ ] Does NOT scope the MVP
- [ ] Loyalty points rules (earn/redeem/expiry) are in the Features section
- [ ] COD confirmation and estimated volume (~60–70%) are in the Constraints section
- [ ] Multi-vendor resolution (single brand only) is captured in output — either Constraints or Confidence Notes

## Conversation Pattern
- [ ] Questions asked one at a time
- [ ] Batching used only for tightly-coupled questions
- [ ] "Defer" responses accepted without pushback and moved to Open Questions
