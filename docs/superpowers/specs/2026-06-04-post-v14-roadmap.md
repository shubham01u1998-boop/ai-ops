# Post-V1.4 Next Steps — Project Initiator Roadmap
**Date:** 2026-06-04
**Author:** Shubham Upadhyay

---

## Where we are

```
START (V1.2, done) → DISCOVERY (done) → MVP_SYNTHESIZER (done) → ARCH_PROPOSER (done) → DOC_GENERATOR (done) → BACKLOG_GENERATOR (V1.4, spec + plan ready, not built) → ROADMAP (future)
```

BACKLOG_GENERATOR V1.4 is fully specced and planned. Implementation is ready to hand off
to a new Claude session using `docs/superpowers/plans/2026-06-04-backlog-generator.md`.

---

## Immediate Next (this session can continue here)

### 1. Implement BACKLOG_GENERATOR (V1.4)
**Trigger:** Hand plan to new Claude session.
**File:** `docs/superpowers/plans/2026-06-04-backlog-generator.md`
**Handoff prompt:**

> Read CLAUDE.md first, then read docs/superpowers/plans/2026-06-04-backlog-generator.md.
> Use superpowers:subagent-driven-development to implement this plan task-by-task.
> The Cold-Start Brief in the plan tells you what else to read before starting.

**Estimated tasks:** 11 tasks, 72+22 QA behaviors to validate.

---

## After BACKLOG_GENERATOR is done

### 2. First Live Engagement Run
Run the full chain — START → DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → DOC_GENERATOR → BACKLOG_GENERATOR — on a real engagement (not a synthetic fixture).

**Why before building more skills:** The chain has never been run end-to-end on real data. A live run will surface UX gaps, prompt improvements, and edge cases that synthetic fixtures miss.

**Recommended first engagement:** Use a TiffinConnect feature (not a new client) since the team knows the domain and can evaluate output quality quickly.

**Output of live run:**
- Real arch.md → real Odoo tickets created
- List of improvements to feed back into skill files
- Confidence that the chain works before adding BACKLOG_GENERATOR to Claude Enterprise

---

### 3. Deploy to Claude Enterprise
Upload BACKLOG_GENERATOR.md to Claude Enterprise (Project: Ticket Intake — TiffinConnect).
Test that MCP connectivity from Enterprise → Odoo is intact before announcing to team.

**Pre-deployment checklist:**
- [ ] BACKLOG_GENERATOR.md tested on 2 fixtures (72/72 + 22/22)
- [ ] Live run completed (at least once)
- [ ] MCP tools tested: `create_project`, `bulk_create_tickets`, `add_subtasks`, `create_tag`

---

### 4. ROADMAP Skill (V1.5) — Design when ready
Not designed yet. Proposed purpose: takes the completed Odoo board and produces a public-facing delivery roadmap (phases, milestones, go-live dates) as a standalone document.

**Inputs (proposed):** arch.md + session_state.md + Odoo project (via MCP read)
**Output (proposed):** `roadmap.md` — quarterly view, milestone format, client-ready

**Design questions to answer before speccing:**
- Is this a separate skill or part of DOC_GENERATOR (a 6th document)?
- Who is the audience — client only, or also internal?
- Does it pull live ticket status from Odoo or use arch.md only?

**Recommendation:** Hold off until after the live engagement run. The live run will clarify whether a roadmap doc is needed or if DOC_GENERATOR's Sprint Plan already covers this.

---

## Parallel track: Phase 2 Skills (separate from Project Initiator)

These are independent of the V1.x chain and can be started any time:

| # | Skill | Status | Depends on |
|---|---|---|---|
| 1 | `LAYER_1_CLASSIFIER.md` | Not started | — |
| 2 | `LAYER_2_PREPROCESSOR.md` | Not started | CLASSIFIER |
| 3 | `REQUIREMENT_BRIDGE.md` | Not started | — |
| 4 | `LAYER_3_TICKET_MAPPER.md` | Not started | CLASSIFIER |
| 5 | `PRD_BREAKDOWN.md` | Not started | — |

These skills serve the Ticket Intake pipeline (TiffinConnect bug/feature intake), not the
Project Initiator chain. They can be designed and built independently.

**Recommended priority:** Start LAYER_1_CLASSIFIER after BACKLOG_GENERATOR is deployed.
CLASSIFIER unblocks the rest of Phase 2.

---

## Technical Debt / Open Items

| Item | Priority | Notes |
|---|---|---|
| Session_state.md consistency across skills | Low | Each skill writes it slightly differently — standardise format in V1.5 |
| START skill resume logic with real engagements | Medium | Test against non-synthetic folders |
| MCP `odoo-mcp/PENDING_CHANGES.md` | Low | Review pending MCP changes before BACKLOG_GENERATOR deployment |
| DISCOVERY + MVP_SYNTHESIZER retested after chain updates | Low | Last tested 2026-06-03; no regressions expected |

---

## Summary

| What | When | Owner |
|---|---|---|
| Implement BACKLOG_GENERATOR | Next session | New Claude session (hand the plan) |
| Live engagement run | After BACKLOG_GENERATOR built | Shubham |
| Deploy to Claude Enterprise | After live run passes | Shubham |
| ROADMAP skill design | After live run | Next planning session |
| Phase 2 — CLASSIFIER | After BACKLOG_GENERATOR deployed | New planning session |
