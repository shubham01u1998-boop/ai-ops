# Project Initiator — Design Spec
# fiftyfive-tech internal consulting tooling
# Version: 1.0 | Date: 2026-06-03 | Status: V1.0 approved, in implementation

---

## Context

fiftyfive-tech consultants spend 1–2 weeks on project initiation before real code starts. Architecture planning happens but is undocumented; requirements are extracted informally; every engagement reinvents the same process. This tooling makes that phase repeatable.

**The core principle:** AI advises, human decides. The tool extracts, clarifies, surfaces conflicts, and presents options with arguments. The consultant picks. No silent assumptions, no auto-filled gaps.

---

## V1.0 Scope — DISCOVERY only

V1.0 ships one skill: DISCOVERY.

Everything else — MVP_SYNTHESIZER, ARCH_PROPOSER, BACKLOG_GENERATOR, ROADMAP, the orchestrator, archive lifecycle — is target vision. It is documented here so design coherence is preserved, but none of it is built in V1.0.

The rationale: Phase 1 ticket intake hasn't been stress-tested on real workloads yet. The full 5-skill chain is speculative until there is signal. Build the smallest valuable thing, dogfood it on one real engagement, learn, then decide V1.1.

---

## DISCOVERY Skill Design

### What it does

DISCOVERY reads rough input documents (specs, meeting notes, emails, requirement fragments) and produces a structured `discovery.md` through a guided consultant conversation. It:
- Reads all readable input docs from an `input/` folder
- Extracts project type, users, problem, features, tech, timeline, constraints
- Assigns HIGH/MED/LOW confidence markers per category
- Surfaces conflicts (same topic stated differently across docs) and asks the consultant to resolve them
- Asks critical gap questions one at a time in priority order: conflicts first → critical gaps → detail gaps
- Allows the consultant to defer any question to Open Questions
- Produces a draft discovery.md, lets the consultant review and edit, then saves

### What it explicitly does NOT do

- Choose architecture or tech stack
- Define MVP scope or decompose features
- Create Odoo tickets
- Make any decision the consultant hasn't explicitly confirmed

### Invocation

Run from an engagement folder that has an `input/` subfolder:
```
<engagement-folder>/
  input/
    rough-spec.md
    meeting-notes.md
    stakeholder-email.txt
```
Ask Claude to "run DISCOVERY." The skill auto-detects the `input/` folder.

### Supported input formats

- `.pdf` — read natively via Read tool
- `.txt`, `.md`, `.markdown` — read natively via Read tool
- `.docx`, `.doc` — skipped with a one-line note; convert to PDF or text first

### discovery.md output sections

| Section | Content |
|---|---|
| Project Context | Business context and problem statement |
| Users | Primary users, secondary users, scale if known |
| Core Problem | What is being solved and why it matters |
| Features Mentioned | Bulleted list with per-feature confidence markers |
| Constraints | Timeline, budget, tech constraints, other hard constraints |
| Open Questions | Numbered list of deferred or unanswered questions |
| Confidence Notes | Where confidence is LOW or where conflicts were resolved |
| Source Docs | Input files read with one-line summaries |

### Question loop pattern

Conflicts first (highest priority) → critical gaps → detail gaps.
One question per turn by default.
Batching exception: 2–4 tightly-coupled questions on the same topic (e.g. scale: users + volume + storage) may be grouped.
Consultant can defer any question at any time.

---

## V1.0 Implementation Steps

1. Scaffold directories: `skills/project-initiator/`, `tests/fixtures/discovery/synthetic-01/input/`, `docs/superpowers/specs/`
2. Write this design spec
3. Write `DISCOVERY.md` skill file
4. Create synthetic test fixture (3 input docs with intentional conflicts + gaps, plus `expected_behaviors.md`)
5. Manual test: run DISCOVERY on synthetic fixture, compare against expected behaviors, iterate
6. Dogfood: run on one real or realistic fiftyfive-tech engagement scenario, document friction
7. Write README + update CLAUDE.md

---

## V1.0 Verification

All of these must pass before V1.0 is considered done:

- [ ] `skills/project-initiator/DISCOVERY.md` exists and follows Phase 1 skill file structure
- [ ] DISCOVERY runs on synthetic fixture; all `expected_behaviors.md` items pass
- [ ] DISCOVERY runs on one real-or-realistic engagement scenario; friction documented
- [ ] discovery.md output has all required sections
- [ ] At least 1 conflict surfaced and resolved in fixture test
- [ ] At least 2 open questions captured in fixture test
- [ ] No invented information (every fact traces to an input doc or consultant answer)
- [ ] Question loop is one-at-a-time; batched only for tightly-coupled topics
- [ ] Confidence markers (HIGH/MED/LOW) present on at least 3 extraction categories
- [ ] README written and CLAUDE.md updated

---

## Target Vision — Future Phases

Documented for design coherence. Build order is approximate and subject to V1.0 signal.

| Phase | Skill | What it adds |
|---|---|---|
| V1.1 | MVP_SYNTHESIZER | discovery.md → mvp-scope.md with framing-based decomposition (time-boxed / risk-first / value-first) |
| V1.2 | PROJECT_INITIATOR (thin orchestrator) | Routes between specialists. SESSION_STATE.md. Transition prompts between skills. |
| V1.3 | ARCH_PROPOSER | mvp-scope.md → arch.md with constrained stack menu + STRAWMAN markers. Build-order decision made here. |
| V1.4 | BACKLOG_GENERATOR | Odoo tickets per template. PL- cross-refs for open questions. Option ζ: incomplete-arch ticket marking. Reuses existing MCP create_ticket. |
| V1.5 | Archive lifecycle | `/archive-engagement` command. Active=local, completed=shared GitLab repo. input/ excluded from archive for NDA safety. |
| V2.0 | ROADMAP, bidirectional PL↔ticket refresh, LLM-as-judge tests, centralized deploy |

### Per-engagement folder structure (V1.2+)
```
~/fiftyfive-engagements/<client-name>/
  input/           # rough docs (local only — NDA)
  artifacts/       # discovery.md, mvp-scope.md, arch.md
  shared-to-client/ # sanitized deliverables
  SESSION_STATE.md # phase tracker
  PARKING_LOT.md   # deferred questions + PL- IDs
  TEAM_CONTEXT.md  # engagement-specific context
```

### Ticket template (BACKLOG_GENERATOR, V1.4+)
Fields: Description, Acceptance Criteria, Tech Context (from arch.md), Open Questions (with PL- IDs), Testing Considerations, Security Considerations, Dependencies, Related Artifacts, Effort.

### Option ζ — incomplete-arch ticket handling (V1.4+)
When architecture is incomplete: tool asks "Proceed with marked-incomplete tickets, or wait for full arch?"
If proceed: tickets are created with explicit "⚠ Arch incomplete" markers and PL- cross-references so devs know what is missing.
If wait: skill pauses and returns to ARCH_PROPOSER to fill gaps.

---

## Deferred Decisions

These were explicitly not decided in V1.0 design:

- **parse_docx MCP tool** — build only when a real engagement has .docx inputs that can't be converted
- **Full vs split build order for ARCH_PROPOSER** — decide when V1.3 is being built
- **Whether engagement repo is per-consultant local or team-shared** — currently per-consultant; revisit if collaboration friction emerges
- **Anthropic Enterprise API availability** — verify before V2.0 deploy script work

---

## Open Risks (V1.0)

1. **Extraction quality varies by doc quality.** Mitigated by confidence markers + explicit gap surfacing + human review. Residual: GIGO — bad input produces low-confidence output the consultant must correct.
2. **Long sessions may lose context.** DISCOVERY is single-skill with no checkpointing. If the session expires mid-run, the consultant restarts. Accepted for V1.0; SESSION_STATE.md is V1.2.
3. **Synthetic fixture may not represent real engagements.** Mitigated by V1.0 dogfood step (real or realistic engagement scenario after fixture passes).

---

## V1.0 Success Criteria

- V1.0 dogfooded on 1 real engagement, friction list captured
- No Phase 1 files modified (LAYER_0_GLOBAL, LAYER_2_FASTPATH, QA_INTAKE, BUG_REPORT_TEMPLATE, TEAM_CONTEXT)
- No orchestrator built
- No SESSION_STATE.md built
- No archive lifecycle built
- No parse_docx MCP tool built unless a real engagement required it
- Friction list reviewed → V1.1 scope decided
