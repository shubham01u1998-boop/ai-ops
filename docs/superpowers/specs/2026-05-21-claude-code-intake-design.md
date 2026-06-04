# Design: Ticket Intake in Claude Code
**Date:** 2026-05-21
**Status:** Approved
**Scope:** Enable the full TiffinConnect ticket intake pipeline (FASTPATH + QA_INTAKE + PARKING_LOT) to run in Claude Code (ai-ops session) identically to Claude Enterprise, with an additional debug mode.

---

## Problem

CLAUDE.md currently redirects all ticket creation requests to Claude Enterprise. MCP is available in Claude Code but the skill-driven flow (classifier → duplicate check → mapper → CONFIRM ALL) is blocked by the redirect rule. The user needs the same pipeline to work in both environments.

---

## Solution Overview

Option C — Hybrid load: read `LAYER_0_GLOBAL` + `TEAM_CONTEXT` at session start (always needed, small files), read `LAYER_2_FASTPATH` or `QA_INTAKE` on demand when intake input is detected. PARKING_LOT checklist runs silently at session start. Two modes: production (identical to Claude Enterprise) and debug (full visibility). Mode is auto-detected from input type — no commands needed.

---

## Section 1: Session-Start Protocol

Every Claude Code session in ai-ops runs this automatically, in order:

1. **Silent reads:**
   - Read `context/PARKING_LOT.md` + `context/PARKING_LOT_SPEC.md` → run the full session-start checklist (steps 1–6 per PARKING_LOT_SPEC.md). Surfaces warnings if any; silent if PARKING_LOT is empty.
   - Read `skills/core/LAYER_0_GLOBAL.md` → apply all rules for the session.
   - Read `context/TEAM_CONTEXT.md` → apply routing, tags, priority rules.

2. **Role question (one message, one question):**
   > "Who is this session? Sahil / Vijay / Kunal / Tanu / Shubham"

   Answer is recorded as the session reporter. Populates `by:` in all REQ{} blocks for the session. No intake processing starts until answered. If intake input arrives before the role is answered, ask the role question first, then process the input.

---

## Section 2: Auto-Detect Mode

Claude reads each input and routes to production or debug mode automatically. No command required.

### Intake mode signals
- Bug description, error message, "failing", "broken", "not working"
- Feature or improvement request
- QA batch, numbered findings, TC-NNN test case references
- PRD section or requirement description

### Debug mode signals
- Questions about existing tickets ("what tickets are open for…")
- MCP inspection requests ("list tickets", "get ticket #…", "search for…")
- System questions ("why did S8 flag…", "what's in PARKING_LOT…")
- Explicit debugging ("check if duplicate check works for…")

**Ambiguous input:** default to intake mode. No clarifying question asked about mode.

---

## Section 3: Intake Mode (Production)

When intake mode is detected:

1. Apply LAYER_0 Routing Rule:
   - QA batch / TC-NNN references → read `skills/layers/QA_INTAKE.md`
   - All other input → read `skills/layers/LAYER_2_FASTPATH.md`

2. Follow the active skill file exactly — same sections, same rules, same output format as Claude Enterprise. All MCP tools (create_ticket, list_tickets, search_tickets, etc.) execute normally.

3. Session reporter (from role answer) auto-populates `by:` field — user does not need to include their name in the input.

**Production output restrictions (identical to Claude Enterprise):**
- REQ{} blocks hidden from user
- MCP calls not shown
- Errors shown as 2-line summary
- Duplicate check similarity scores not shown
- Batch review table, CONFIRM ALL, and all commands behave identically

---

## Section 4: Debug Mode

When debug mode is detected, Claude runs the same skill-driven flow with full visibility:

| Extra output | Detail |
|---|---|
| REQ{} blocks shown | Displayed after extraction, before batch table |
| MCP calls logged inline | `→ list_tickets(project_id=58, tag="bug", limit=50)` before results |
| Raw Odoo errors shown | Full error, no 2-line cap |
| Similarity scores shown | `"OTP login failing" — 81% match → #2541` |

All LAYER_0 rules still apply in debug mode (permission, token estimate, output limits on other things). Debug only lifts the visibility restrictions — it does not skip steps or bypass confirmations.

---

## Section 5: CLAUDE.md Changes

**Remove:** The existing redirect rule in `## Working Rules`:
> *"Ticket creation requests → redirect to Claude Enterprise. Do not create tickets directly here. MCP access in ai-ops is for inspection and debugging only, not ticket intake. The full FASTPATH flow (classifier, duplicate check, mapper) only runs in Claude Enterprise."*

**Add:** A new `## Ticket Intake` section with:
- Session-start protocol (Section 1 above)
- Auto-detect signals (Section 2 above)
- Intake mode instructions (Section 3 above)
- Debug mode instructions (Section 4 above)

**Update:** `Locked Design Decisions` list — remove stale redirect note, add:
> *"Ticket intake runs in both Claude Code and Claude Enterprise — same skill files, same flow."*

---

## Section 6: SYSTEM_FLOW.md Changes

**Add** to "What changed vs. old diagram" table:

| Component | Old | Now |
|---|---|---|
| Claude Code entry point | Redirect to Enterprise | Full pipeline runs in Claude Code (same skill files, same MCP, same output) |
| Debug mode | Not available | Auto-detected in Claude Code — shows REQ{}, MCP calls, raw errors, similarity scores |

**Add** one line under "Demo Readiness":
> Claude Code (ai-ops) now runs the same intake flow as Claude Enterprise. Same demo script works in both environments.

---

## Files Changed

| File | Change |
|---|---|
| `CLAUDE.md` | Remove redirect rule, add `## Ticket Intake` section, update Locked Design Decisions |
| `docs/SYSTEM_FLOW.md` | Add Claude Code entry point rows to What Changed table, add demo note |

**No skill files modified.** They remain the single source of truth for both environments.
