# DOC_GENERATOR — Design Spec
**Skill version:** V1.3.5
**Date:** 2026-06-04
**Author:** Shubham Upadhyay

---

## Position in Chain

```
DISCOVERY (V1.0) → MVP_SYNTHESIZER (V1.1) → ARCH_PROPOSER (V1.3) → DOC_GENERATOR (V1.3.5) → BACKLOG_GENERATOR (V1.4)
```

---

## Purpose

DOC_GENERATOR reads `discovery.md`, `mvp-scope.md`, and `arch.md` from the engagement
folder, cross-validates that all three are in sync, presents a menu of deliverable
documents, generates each selected doc as a professional draft with Mermaid diagrams,
gets consultant approval per doc, and saves to the `docs/` subfolder.

No new questions are asked — all data is captured upstream. DOC_GENERATOR is a
presentation layer only: it never modifies `discovery.md`, `mvp-scope.md`, or `arch.md`.

---

## Inputs

| File | Produced by | Required |
|---|---|---|
| `discovery.md` | DISCOVERY | Yes |
| `mvp-scope.md` | MVP_SYNTHESIZER | Yes |
| `arch.md` | ARCH_PROPOSER | Yes |

All three must be present in the current working directory. If any are missing, skill
stops with a message listing what is missing and which skill produces it.

---

## Outputs

All files saved to `<current working directory>/docs/` — the engagement folder where the
skill is run.

| # | Document | Filename | Audience |
|---|---|---|---|
| 1 | Project Proposal / SOW | `docs/proposal.md` | Client |
| 2 | Technical Architecture Doc | `docs/tech-arch.md` | Dev team |
| 3 | Sprint Plan | `docs/sprint-plan.md` | Dev team + PM |
| 4 | Developer Handoff Doc | `docs/dev-handoff.md` | Dev team |
| 5 | Scope Agreement | `docs/scope-agreement.md` | Client |

Example for a RetailEdge engagement:
```
~/fiftyfive-engagements/retailedge/
  discovery.md
  mvp-scope.md
  arch.md
  docs/
    proposal.md
    tech-arch.md
    sprint-plan.md
    dev-handoff.md
    scope-agreement.md
```

---

## Pre-flight

Before anything else, silently:

1. Run `basename "$PWD"` via Bash tool. Use result as engagement name throughout.
   - If name is generic (e.g. `test`, `temp`, `folder`): ask once — "What's the engagement
     or client name?" Store the answer. Do not ask again.

2. Check for all three required files using Read tool.
   - If any missing: surface which files are missing, which skill produces each, and stop.

3. After loading all three successfully, output one line:
```
Engagement: <name> | discovery.md ✓ | mvp-scope.md ✓ | arch.md ✓. Running sync check.
```

---

## Sync Check (Cross-Artifact Validation)

The sync check is the safety net for DOC_GENERATOR. It runs before the menu appears and
cross-validates all three artifacts for consistency. Never skip it.

### Checks

| Check | What it validates | PASS | WARN | DRIFT |
|---|---|---|---|---|
| Features → Components | Every feature in `mvp-scope.md ## Scope: In` has ≥1 component in `arch.md ## Components` | All mapped | Component exists for out-of-scope feature (arch may add shared infra) | Feature in scope has zero components |
| Tech constraints | Tech items confirmed in `mvp-scope.md ## Constraints → Tech:` appear in `arch.md ## Tech Stack` with matching decision | Consistent | — | Arch chose different tech for a confirmed item |
| Timeline | `mvp-scope.md ## Constraints → Timeline:` date matches `arch.md ## Sprint Mapping` end date | Consistent | Minor variance (±1 week) | Different date |
| Budget | Present in `mvp-scope.md ## Constraints → Budget:` | Present | "not specified" or blank | Line missing entirely |
| STRAWMANs | Any `[STRAWMAN]` items present in `arch.md` | None found | STRAWMANs present — listed in gate output | — |

### DRIFT vs WARN

**WARN** — does not block. Flagged inline in the relevant generated doc (e.g. budget not
specified → SOW shows a placeholder).

**DRIFT** — blocks doc generation for the affected doc only. Consultant resolves inline,
one item at a time, before the menu appears:
```
DRIFT: arch.md shows Node.js backend but mvp-scope.md had .NET confirmed. Which is current?
```
The answer updates the skill's working state for doc generation. It does NOT modify any
source file.

### Gate Output

Shown once after checking all fields:
```
Sync check: discovery.md ✓ | mvp-scope.md ✓ | arch.md ✓
✓ Features → Components — all N features mapped
✓ Tech — <confirmed items> consistent
✓ Timeline — <date> consistent
⚠ Budget — not specified (SOW will show placeholder)
⚠ STRAWMAN — N items flagged (listed in docs where relevant)

All clear — N WARNs, no DRIFT. Proceeding to document menu.
```

If DRIFT items exist: resolve them first, re-show the check output, then show the menu.

---

## Document Menu

After sync check clears, show once:

```
Documents available for <engagement name>:

  1  Project Proposal / SOW      — scope, timeline, budget, risks        [Client]
  2  Technical Architecture Doc  — stack, components, diagrams            [Dev team]
  3  Sprint Plan                 — delivery calendar, team, milestones    [Dev + PM]
  4  Developer Handoff Doc       — components, data model, integrations   [Dev team]
  5  Scope Agreement             — updated scope with arch-phase deltas   [Client]

Select documents to generate (e.g. "1 3 4" / "all" / "client" / "dev"):
```

**Shorthand aliases:**
- `all` → 1 2 3 4 5
- `client` → 1 5
- `dev` → 2 3 4

After selection, confirm and begin:
```
Generating: Proposal (1), Developer Handoff (4), Scope Agreement (5).
Starting with Proposal...
```

Docs generated in the order selected, one at a time.

---

## Per-Document Content

All documents are synthesized from the three source artifacts. No new questions are asked
during generation. Documents must be professional quality — suitable for sharing directly
with clients or internal team members without further editing.

### Doc 1 — Project Proposal / SOW `[Client-facing]`

Client language throughout. No component names, no framework names, no technical jargon.

| Section | Source |
|---|---|
| Executive Summary | `discovery.md` core problem + `mvp-scope.md` problem restatement |
| MVP Scope | `mvp-scope.md` Scope: In/Out — features in business terms |
| Key User Journeys | `mvp-scope.md` Key User Journeys |
| High-Level Architecture | `arch.md` Client Summary — non-technical 3-5 sentences |
| Delivery Timeline | `arch.md` Sprint Mapping — **Mermaid Gantt** |
| Budget | `mvp-scope.md` Constraints → Budget (placeholder if WARN) |
| Risks & Assumptions | `arch.md` STRAWMAN Summary + `mvp-scope.md` Assumptions — STRAWMANs rewritten in business language |
| Success Metrics | `mvp-scope.md` Success Metrics |
| Team | `arch.md` Sprint Mapping → Team roster |
| Sign-off block | Engagement name, fiftyfive technologies branding |

**Diagrams:** 1 × Mermaid Gantt (delivery timeline)

---

### Doc 2 — Technical Architecture Doc `[Dev team]`

| Section | Source |
|---|---|
| Overview | `discovery.md` core problem + `mvp-scope.md` problem restatement — 2-3 sentences |
| Tech Stack | `arch.md` Tech Stack table — confirmed items + STRAWMAN flags |
| Architecture Diagram | `arch.md` Components — **Mermaid graph** — all components, dependencies, shared services |
| Component Inventory | `arch.md` Components — name, tech, purpose, shared flag |
| Data Model | `arch.md` Data Model Hints — **Mermaid erDiagram** — tables, fields, relationships |
| Integration Points | `arch.md` Integration Points — table + **Mermaid sequence diagram** for key flows |
| Build Order | `arch.md` Build Order — ordered list with rationale |
| Open Questions | `arch.md` Open Questions — flagged with which component each blocks |
| STRAWMAN Summary | `arch.md` STRAWMAN Summary — full list, "verify before Sprint 1" |

**Diagrams:** Mermaid graph (component architecture) + Mermaid erDiagram (data model) + Mermaid sequence (integration flows)

---

### Doc 3 — Sprint Plan `[Dev team + PM]`

| Section | Source |
|---|---|
| Team Roster | `arch.md` Sprint Mapping → Team — name, seniority, role |
| Sprint Calendar | `arch.md` Sprint Mapping table — **Mermaid Gantt** per sprint, per owner |
| Sprint Breakdown | `arch.md` Sprint Mapping — sprint number, dates, deliverables, owner |
| Dependency Map | `arch.md` Build Order + Components — table: what must complete before what |
| Risk Flags per Sprint | `arch.md` STRAWMAN items — which STRAWMANs affect which sprint |

**Diagrams:** 1 × Mermaid Gantt (detailed sprint calendar)

---

### Doc 4 — Developer Handoff Doc `[Dev team]`

| Section | Source |
|---|---|
| Engagement Context | `discovery.md` + `mvp-scope.md` — 3-4 sentences to orient a new developer |
| Tech Stack | `arch.md` Tech Stack — confirmed decisions + STRAWMAN flags |
| Component Inventory | `arch.md` Components — per component: tech, purpose, dependencies, open Qs |
| Data Model | `arch.md` Data Model Hints — **Mermaid erDiagram** |
| Integration Flows | `arch.md` Integration Points — **Mermaid flowchart** per integration system |
| Build Order | `arch.md` Build Order — with dependency rationale |
| Open Questions | `arch.md` Open Questions — per blocker, which component it blocks |
| STRAWMAN Checklist | `arch.md` STRAWMAN Summary — checklist format, verify before Sprint 1 |

**Diagrams:** Mermaid erDiagram (data model) + Mermaid flowchart (integration flows)

---

### Doc 5 — Scope Agreement `[Client-facing]`

Builds on the MVP_SYNTHESIZER-generated scope agreement and adds arch-phase information.
If a DRIFT was found in the sync check, a "What Changed Since MVP Scope" section is
included automatically — no extra prompting needed.

| Section | Source | What's New vs MVP_SYNTHESIZER version |
|---|---|---|
| Problem Statement | `discovery.md` + `mvp-scope.md` | Same |
| Users | `mvp-scope.md` | Same |
| MVP Scope In/Out | `mvp-scope.md` | Same |
| Key User Journeys | `mvp-scope.md` | Same |
| Delivery Timeline | `arch.md` Sprint Mapping | New — Mermaid Gantt added from arch phase |
| Tech & Infra Constraints | `arch.md` Tech Stack | Confirmed tech decisions added |
| Assumptions | `mvp-scope.md` + `arch.md` STRAWMAN Summary | Updated with arch-phase assumptions |
| Pre-conditions for Start | `arch.md` Open Questions | Updated — resolved items marked ✓, unresolved carried forward |
| What Changed Since MVP Scope | Sync check DRIFT items | New section — only appears if DRIFT was found |
| Success Metrics | `mvp-scope.md` | Same |
| Sign-off block | — | Client name from engagement folder, fiftyfive branding |

**Diagrams:** 1 × Mermaid Gantt (delivery timeline)

---

## Review & Save Flow

Same pattern as upstream skills in the chain. Applied once per selected doc.

1. Show full draft (all sections + all diagrams)
2. Prompt:
```
Looks complete — save <filename> to docs/? [A] Yes / [B] Edit a section / [C] Add more context
```
3. Natural-language handling:
   - Clear approval ("yes", "save it", "looks good", "correct", "perfect") → interpret as [A]. Save and move to next doc.
   - Edit offered ("change X to Y", "section 3 is wrong") → interpret as [B]. Apply change, re-show affected section, re-prompt.
   - New info offered → interpret as [C]. Update draft silently, re-show with changes applied.
   - Ambiguous → "Reading this as [A] Save — correct? (yes / [B] edit a section / [C] add more context)"

4. On save: create `docs/` subfolder if it doesn't exist. Write file to
   `<current working directory>/docs/<filename>`. Confirm:
```
docs/proposal.md saved.
```

5. If more docs selected: `Next: generating Technical Architecture Doc...` and repeat.

6. After all selected docs saved:
```
DOC_GENERATOR complete.
Saved to docs/: <list of saved files>

Next: run BACKLOG_GENERATOR to create Odoo tickets from arch.md (not yet built — V1.4).
```

---

## Rules

1. Never modify `discovery.md`, `mvp-scope.md`, or `arch.md` — docs are presentation layer only.
2. Sync check: never skip it. Every invocation, before the menu appears.
3. DRIFT items block the menu — all DRIFTs resolved before generation begins. Consultant can defer any DRIFT to WARN; deferred DRIFTs are flagged as placeholders in the affected doc.
4. Never auto-save — explicit consultant approval required for every doc.
5. Docs must be professional quality — ready to share with clients or team without further editing.
6. All diagrams use Mermaid syntax embedded in the markdown output.
7. Client-facing docs (1 and 5): no component names, no framework names, no technical jargon.
8. `docs/` subfolder created silently if missing — no prompt needed.
9. LAYER_0_GLOBAL Rule 4 output limits apply. Rule 5 (no narration) applies.
10. V1.3.5 boundary: never create Odoo tickets (BACKLOG_GENERATOR), never modify upstream artifacts.

---

## Test Fixtures

Location: `tests/fixtures/doc-generator/`

Fixtures use the RetailEdge synthetic engagement from ARCH_PROPOSER:
- Input: `tests/fixtures/arch-proposer/synthetic-01/` (discovery.md + mvp-scope.md + arch.md)
- Expected: `tests/fixtures/doc-generator/synthetic-01/expected_behaviors.md`

Two fixture paths needed:
- `synthetic-01` — PASS path: all 5 docs generated, one WARN (budget), no DRIFT
- `synthetic-02-drift` — DRIFT path: one sync check DRIFT item, consultant resolves inline, Scope Agreement includes "What Changed" section
