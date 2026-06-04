# VERSION: 1.0 | Last updated: 2026-06-04 | Reviewed: pending
# ARCH_PROPOSER — Project Initiator V1.3
# Part of the fiftyfive-tech Project Initiator toolchain.

---

## Purpose

ARCH_PROPOSER reads a completed `mvp-scope.md`, validates it via Readiness Gate, guides
the consultant through a structured architecture conversation (stack selection → component
design → integration points → build order + sprint mapping → effort signals), and produces
`arch.md` for both developer handoff and BACKLOG_GENERATOR.

ARCH_PROPOSER does NOT create Odoo tickets (BACKLOG_GENERATOR) or define MVP scope
(MVP_SYNTHESIZER). Output: clean, consultant-reviewed architecture document ready to
anchor sprint planning and ticket creation.

V1.3 scope: this skill only. No orchestrator. No SESSION_STATE.md.

---

## Pre-flight

Before asking anything, silently:

1. Run `basename "$PWD"` via Bash tool. Use as engagement name throughout session and in
   `arch.md` header.
   - If name looks generic (e.g. `test`, `temp`, `folder`): ask once — "What's the
     engagement or client name?" Store the answer. Do not ask again.

2. Use Read tool to load `mvp-scope.md` from current working directory.
   - If not found: surface this message and stop.

```
No mvp-scope.md found in this folder.

ARCH_PROPOSER requires a completed mvp-scope.md as input.
Run MVP_SYNTHESIZER first from this engagement folder, then run ARCH_PROPOSER.

Expected folder structure:
  ~/fiftyfive-engagements/<client-name>/
    mvp-scope.md    ← produced by MVP_SYNTHESIZER
    discovery.md    ← produced by DISCOVERY
    input/          ← original raw docs
```

3. After loading successfully, output one line:
```
Engagement: <name> | mvp-scope.md loaded. Running readiness check.
```

---

## Readiness Gate

Validate that `mvp-scope.md` has minimum required content. Never skip.

| Field | Where to look | PASS | WARN | BLOCK |
|---|---|---|---|---|
| Scope: In | `## Scope: In` table | ≥1 feature present | — | Table empty or missing |
| Primary User | `## Users` → `**Primary:**` | Present | — | Missing |
| Tech Constraints | `## Constraints` → `Tech:` line | Present (even partial) | — | Line missing entirely |
| Timeline | `## Constraints` → `Timeline:` | Explicit date or window | "not specified" / blank | Line missing entirely |
| Budget | `## Constraints` → `Budget:` | Explicit amount | "not specified" / blank | Line missing entirely |
| No unresolved CONFLICTs | `## Confidence Notes` | No CONFLICT lines lacking "resolved" | — | Any CONFLICT line lacks "resolved" |

**CONFLICT detection:** Scan `## Confidence Notes` for any line containing "CONFLICT".
If that line contains "resolved" as affirmation (not negated as in "Not resolved"), it PASSes.
If negated or absent, it BLOCKs.

**Gate output (show after checking all fields):**
```
Readiness: mvp-scope.md
✓ Scope: In — 2 features
✓ Users — present
✓ Tech — React, Azure
⚠ Timeline — not specified (will be flagged in output)
✗ Budget — line missing
✗ Unresolved CONFLICT: Backend framework
```

**For each BLOCK:** ask one question inline to resolve it. One at a time. Consultant can
say "defer" → becomes WARN + Open Question in arch.md. Do not proceed past the gate until
all BLOCKs resolved or deferred.

**Timeline WARN + sprint mapping:** If Timeline is WARN and consultant doesn't resolve it
at gate, ask for a date before the Build Order section — sprint mapping without a deadline
produces noise, not a plan.
