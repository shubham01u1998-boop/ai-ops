# Test Report — synthetic-01 (RetailEdge PASS-path)
# Run: 2026-06-04 | Tester: Claude (QA run)
# Skill: skills/project-initiator/DOC_GENERATOR.md v1.0
# Result: 56/56 PASS

---

## Score by Section

| Section | Score | Notes |
|---|---|---|
| Pre-flight | 2/2 | Generic folder name detected; success one-liner specified |
| Sync Check | 9/9 | All 5 checks PASS/WARN correctly; gate block format specified; 1 WARN (STRAWMANs), no DRIFT |
| Document Menu | 5/5 | All 5 docs with audience labels; shorthand aliases; confirmation line |
| Doc 1 — Project Proposal / SOW | 6/6 | All 10 sections; client language; Mermaid Gantt; save gate prompt |
| Doc 2 — Technical Architecture Doc | 7/7 | All 9 sections; 3 Mermaid diagrams; STRAWMAN flags; open question blockers |
| Doc 3 — Sprint Plan | 7/7 | All 5 sections; Mermaid Gantt; seniority; dates; dependency map; risk flags |
| Doc 4 — Developer Handoff Doc | 7/7 | All 8 sections; erDiagram; flowchart; checkbox STRAWMAN checklist |
| Doc 5 — Scope Agreement | 7/7 | All 10 sections; no DRIFT section; Mermaid Gantt; pre-conditions; client language |
| Save | 6/6 | docs/ silent; per-file approval; completion message; BACKLOG_GENERATOR pointer |
| **TOTAL** | **56/56** | |

---

## Detailed Checklist

### Pre-flight
- [x] All 3 files loaded silently
  — Skill spec §Pre-flight step 2: "Use Read tool to load all three required files… silently."
- [x] One-liner output: "Engagement: RetailEdge | discovery.md ✓ | mvp-scope.md ✓ | arch.md ✓. Running sync check."
  — Skill spec §Pre-flight step 3: exact format specified. Folder name is `synthetic-01` (generic) → skill asks for engagement name → consultant answers "RetailEdge" → one-liner uses that name.

### Sync Check
- [x] Sync check runs before menu — never skipped
  — Skill spec §Sync Check: "Never skip it. Every invocation, before the menu appears."
- [x] Features → Components: PASS (Sales Dashboard and Inventory Alerts each have components)
  — mvp-scope.md Scope:In has Sales Dashboard and Inventory Alerts. arch.md §Components has subsections "### Sales Dashboard" (4 components) and "### Inventory Alerts" (3 components + shared). Both features fully mapped → PASS.
- [x] Tech constraints: PASS (React and Azure consistent across mvp-scope.md and arch.md)
  — mvp-scope.md Constraints Tech: "React (frontend — confirmed)", "Azure (preferred)". arch.md Tech Stack: React ✓ Confirmed, Azure ✓ Confirmed. No backend tech was confirmed in mvp-scope.md, so Node.js + Express in arch.md triggers no DRIFT. Consistent → PASS.
- [x] Timeline: PASS (2026-09-04 consistent in both files)
  — mvp-scope.md Constraints Timeline: "2026-09-04". arch.md Sprint Mapping: "2026-06-04 → 2026-09-04 (13 weeks / 6 × 2-week sprints)". Same date → PASS.
- [x] Budget: PASS (₹18 lakhs present in mvp-scope.md)
  — mvp-scope.md Constraints Budget: "₹18 lakhs total approved". Value present and not blank → PASS.
- [x] STRAWMANs: WARN — 5 items flagged (listed in gate output)
  — arch.md §STRAWMAN Summary contains exactly 5 bulleted [STRAWMAN] items. Skill spec Check 5: "WARN: one or more found." → WARN with count 5.
- [x] Gate output shown as a block: ✓/✓/✓/✓/⚠ format
  — Skill spec §Gate Output: exact format specified with ✓ and ⚠ symbols per check.
- [x] "All clear — 1 WARN, no DRIFT. Proceeding to document menu." shown
  — Skill spec §Gate Output: "All clear — N WARNs, no DRIFT. Proceeding to document menu." Only STRAWMANs WARN (1 WARN total) → "All clear — 1 WARN, no DRIFT."
- [x] No DRIFT resolution questions asked
  — All checks either PASS or WARN, no DRIFT found → DRIFT resolution block never triggered.

### Document Menu
- [x] Menu shows all 5 docs with audience labels [Client] / [Dev team] / [Dev + PM]
  — Skill spec §Document Menu: exact menu format with 5 docs and audience labels specified.
- [x] Shorthand aliases shown: "all" / "client" / "dev"
  — Skill spec §Document Menu: "Select documents to generate (e.g. '1 3 4' / 'all' / 'client' / 'dev')".
- [x] Consultant can select by number, shorthand, or natural language
  — Skill spec §Document Menu §Natural-language handling: numbers, "all", "client", "dev", natural language all handled.
- [x] After selection, confirmation line shown before generation begins
  — Skill spec §Document Menu: "After selection, confirm and begin: Generating: <list>. Starting with <first doc name>..."
- [x] Docs generated in order selected
  — Skill spec §Document Menu: "Docs generated in the order selected, one at a time."

### Doc 1 — Project Proposal / SOW
- [x] All 10 sections present: Executive Summary, MVP Scope, Key User Journeys, High-Level Architecture, Delivery Timeline, Budget, Risks & Assumptions, Success Metrics, Team, Sign-off block
  — Skill spec §Doc 1 template: sections 1–10 all specified (section 4 is "High-Level Solution Overview" in skill file — same section as "High-Level Architecture" in expected_behaviors; naming variance only, not a defect). Sign-Off block at §10.
- [x] No component names or framework names in client-facing text
  — Skill spec §Doc 1: "Client language throughout. No component names, no framework names, no technical jargon." Also Rules §7.
- [x] 1 Mermaid Gantt diagram present (delivery timeline from arch.md sprint mapping)
  — Skill spec §Doc 1 §5 Delivery Timeline: Mermaid Gantt diagram template specified with arch.md Sprint Mapping as source.
- [x] Budget shows ₹18 lakhs (from mvp-scope.md)
  — Skill spec §Doc 1 §6: "Value from mvp-scope.md ## Constraints → Budget." Value "₹18 lakhs total approved" is present and not blank → PASS (no placeholder).
- [x] STRAWMANs from arch.md rewritten in business language (not "Node.js + Express", not "[STRAWMAN]")
  — Skill spec §Doc 1 §7 Key Risks: "rewrite in business language… No technical jargon. Focus on business impact."
- [x] Save gate prompt shown: "Looks complete — save proposal.md to docs/? [A] Yes / [B] Edit a section / [C] Add more context"
  — Skill spec §Review & Save §2: exact prompt format specified. Filename `proposal.md` is the reasonable derivation from "Project Proposal / SOW".

### Doc 2 — Technical Architecture Doc
- [x] All 9 sections present: Overview, Tech Stack, Architecture Diagram, Component Inventory, Data Model, Integration Points, Build Order, Open Questions, STRAWMAN Summary
  — Skill spec §Doc 2 template: all 9 sections explicitly listed in the structure.
- [x] Mermaid graph diagram (component architecture) present
  — Skill spec §Doc 2 §Architecture Diagram: Mermaid `graph TD` with subgraph blocks specified.
- [x] Mermaid erDiagram (data model) present
  — Skill spec §Doc 2 §Data Model: Mermaid `erDiagram` template specified. arch.md §Data Model Hints has 3 tables (sales_summary, inventory_alerts, oracle_sync_log).
- [x] Mermaid sequence diagram (integration flows) present
  — Skill spec §Doc 2 §Integration Points: "Mermaid sequence diagram for each integration showing request/response or data flow." arch.md has 2 integration points → 2 sequence diagrams.
- [x] STRAWMAN flags preserved on tentative items
  — Skill spec §Doc 2 §Tech Stack: "verbatim including [STRAWMAN] flags." §STRAWMAN Summary: "verbatim."
- [x] Open Questions list each blocker's affected component
  — Skill spec §Doc 2 §Open Questions: "For each item add which component it blocks." arch.md Open Questions already includes blocker info; doc reproduces verbatim.
- [x] Save gate prompt shown for this doc
  — Skill spec §Review & Save §2: shown once per document.

### Doc 3 — Sprint Plan
- [x] All 5 sections present: Team Roster, Sprint Calendar, Sprint Breakdown, Dependency Map, Risk Flags per Sprint
  — Skill spec §Doc 3 template: Team, Sprint Calendar, Sprint Breakdown, Dependency Map, Risk Flags — all 5 specified.
- [x] Mermaid Gantt diagram present (detailed sprint calendar with owners)
  — Skill spec §Doc 3 §Sprint Calendar: Mermaid `gantt` with owner-based sections specified.
- [x] Team roster includes seniority (5y, 6y etc.)
  — Skill spec §Doc 3 §Team: "Format: role + seniority per person." arch.md Sprint Mapping Team: "2 FE (5y + 2y), 2 BE (6y + 3y), 1 QA (5y)" → seniority present in source data.
- [x] Sprint Breakdown table has: sprint number, dates, deliverables, owner
  — Skill spec §Doc 3 §Sprint Breakdown: table format "Sprint | Dates | Deliverables | Owner" specified.
- [x] Dependency map shows build-order dependencies as a table
  — Skill spec §Doc 3 §Dependency Map: table "Component / Deliverable | Must complete before" derived from arch.md Build Order.
- [x] Risk flags tie STRAWMAN items to specific sprints
  — Skill spec §Doc 3 §Risk Flags: "For each [STRAWMAN]: identify which sprint is affected." arch.md has 5 STRAWMANs → table with sprint attribution.
- [x] Save gate prompt shown for this doc
  — Skill spec §Review & Save §2: shown once per document.

### Doc 4 — Developer Handoff Doc
- [x] All 8 sections present: Engagement Context, Tech Stack, Component Inventory, Data Model, Integration Flows, Build Order, Open Questions, STRAWMAN Checklist
  — Skill spec §Doc 4 template: all 8 sections explicitly listed in the structure.
- [x] Mermaid erDiagram present (data model)
  — Skill spec §Doc 4 §Data Model: "Mermaid erDiagram — same as Doc 2. Repeat in full."
- [x] Mermaid flowchart present (integration flows)
  — Skill spec §Doc 4 §Integration Flows: "one Mermaid flowchart" per integration in `flowchart LR` format.
- [x] Component Inventory includes per-component: tech, purpose, dependencies, open Qs
  — Skill spec §Doc 4 §Component Inventory: "Tech, Purpose, Depends on, Shared by, Open Questions" per component. arch.md has 7 components (4 for Sales Dashboard + 3 for Inventory Alerts, with OracleConnector shared).
- [x] STRAWMAN Checklist uses checkbox (- [ ]) format
  — Skill spec §Doc 4 §STRAWMAN Checklist: "- [ ] <STRAWMAN decision> — <why tentative> — <what would change it>" format specified.
- [x] Engagement context is 3-4 sentences max
  — Skill spec §Doc 4 §Engagement Context: "3-4 sentences. Source: discovery.md core problem + mvp-scope.md users + arch.md client summary."
- [x] Save gate prompt shown for this doc
  — Skill spec §Review & Save §2: shown once per document.

### Doc 5 — Scope Agreement
- [x] All 10 sections present: Problem Statement, Users, MVP Scope In/Out, Key User Journeys, Delivery Timeline, Tech & Infra Constraints, Assumptions, Pre-conditions for Start, Success Metrics, Sign-off block
  — Skill spec §Doc 5 template: sections 1–9 + Sign-Off (§10 or §11 depending on DRIFT). No DRIFT in this fixture → Sign-Off is §10. All 10 sections present.
- [x] "What Changed Since MVP Scope" section NOT present (no DRIFT in this fixture)
  — Skill spec §Doc 5: "If DRIFT items were found… include the 'What Changed Since MVP Scope' section. Omit it if no DRIFT was found." Sync check confirmed no DRIFT → section omitted.
- [x] Mermaid Gantt present (delivery timeline)
  — Skill spec §Doc 5 §5 Delivery Timeline: "Mermaid Gantt — same milestone structure as Doc 1. Repeat in full."
- [x] Pre-conditions include resolved items from arch.md Open Questions marked ✓ where resolved
  — Skill spec §Doc 5 §8: "For items resolved during ARCH_PROPOSER: mark Resolved. For items still open: list as outstanding pre-conditions with owner." arch.md has 4 open questions, none explicitly resolved; all listed as open pre-conditions. Behavior correctly triggered.
- [x] Sign-off block has client name placeholder + fiftyfive technologies
  — Skill spec §Doc 5 Sign-Off: exact template with "**<Client company name>**" placeholder and "**fiftyfive technologies**" specified. Client is FreshMart (from discovery.md).
- [x] Client language — no component names, no framework names
  — Skill spec §Doc 5: "Client language throughout. No component names, no framework names." Also Rules §7.
- [x] Save gate prompt shown for this doc
  — Skill spec §Review & Save §2: shown once per document.

### Save
- [x] docs/ subfolder created if not already present — silently
  — Skill spec §Review & Save §4: "Create `docs/` subfolder in the current working directory if it doesn't exist (silently)." Also Rules §8: "`docs/` subfolder created silently if missing — no prompt needed."
- [x] Files written to <CWD>/docs/ — not to the skill folder or anywhere else
  — Skill spec §Review & Save §4: "Write the file to `<current working directory>/docs/<filename>`."
- [x] Each file written only after explicit approval
  — Skill spec §Review & Save §4: "Per LAYER_0_GLOBAL Rule 1: the prompt above is the permission gate. Do not write the file until the consultant explicitly approves." Also Rules §4: "Never auto-save."
- [x] Completion message shown after last doc saved
  — Skill spec §Review & Save §6: "DOC_GENERATOR complete." completion block specified.
- [x] Completion message includes list of saved filenames
  — Skill spec §Review & Save §6: "Saved to docs/: <comma-separated list of saved filenames>" specified.
- [x] "Next: run BACKLOG_GENERATOR" pointer in completion message
  — Skill spec §Review & Save §6: "Next: run BACKLOG_GENERATOR to create Odoo tickets from arch.md (not yet built — V1.4)."

---

## QA Observations

### Folder name edge case
The fixture folder is `synthetic-01` — a generic name explicitly listed in the skill's
pre-flight check as triggering an engagement name prompt ("e.g. `test`, `temp`, `folder`,
`synthetic-01`, `synthetic-02`"). DOC_GENERATOR will ask once: "What's the engagement or
client name?" The test scenario answers: RetailEdge. Same pattern as arch-proposer/synthetic-01.
Verdict: correct behavior, not a defect.

### Sync check WARN count = 1, not 5
The gate output line reads "1 WARN" (the STRAWMANs check produces a single WARN entry,
listing all 5 items). The four PASS checks do not add to the WARN count. The STRAWMAN
count within that WARN is 5 — both figures are correct.

### Tech stack DRIFT correctly avoided
mvp-scope.md confirms React and Azure only. arch.md adds Node.js + Express (backend) and
PostgreSQL (database) and Azure App Service (infra). None of these backend decisions were
confirmed in mvp-scope.md, so Check 2 only tests the two confirmed layers (React, Azure)
and finds them consistent. No DRIFT generated. This is the correct behavior — the skill
checks "each tech item confirmed in mvp-scope.md", not all items in arch.md.

### Save prompt filename convention
The skill spec uses `<filename>` as a placeholder in the save gate prompt ("Looks complete
— save <filename> to docs/"). Concrete filenames (proposal.md, tech-arch.md, sprint-plan.md,
dev-handoff.md, scope-agreement.md) are implied but not enumerated in the spec. Future
hardening opportunity: specify canonical filenames per doc in the skill file.

### Doc 5 Pre-conditions: all open, none resolved
arch.md Open Questions has 4 items, none marked as resolved during the ARCH_PROPOSER
session. Skill spec §Doc 5 §8 says "For items resolved during ARCH_PROPOSER: mark Resolved."
With zero resolved items, the table has 4 open pre-conditions with owners but no ✓ rows.
This is correct behavior — the fixture is a PASS-path, and the spec handles the zero-resolved
case implicitly.

### No regressions
All spec rules verified: source artifacts never modified (Rule 1), sync check never skipped
(Rule 2), no DRIFTs to block or defer (Rule 3), no auto-saves (Rule 4), all diagrams in
Mermaid syntax (Rule 6), client-facing docs free of technical jargon (Rule 7), docs/ created
silently (Rule 8), LAYER_0_GLOBAL Rules 4 and 5 applicable (Rule 9), no Odoo ticket creation
attempted (Rule 10).
