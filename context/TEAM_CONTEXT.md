# TEAM_CONTEXT — TiffinConnect
# VERSION: 1.3 | Last reviewed: 2026-05-14 | Lines: 128/200 | Maintained by: Lead
# Rules: Do not edit Section 5 manually — auto-populated during sessions.
# When Section 5 hits 25 lines (per LAYER_0 Rule 8), review and prune before continuing.

---

## LOAD GUIDE
Bug/Feature intake:    load sections 2, 3, 4
PRD breakdown:         load sections 1, 2, 3, 4
Triage session:        load sections 1, 4, 5
Full session:          load all sections

---

## Section 1: Team Roster
Format: Name | Job Title | Odoo User ID | Area of ownership

Sahil Tyagi      | Frontend Developer | 50 | UI, components, frontend bugs
Vijay Mehrotra   | Frontend Developer | 62 | UI, components, frontend bugs
Kunal Sharma     | Backend Developer  | 41 | API, database, backend bugs
Tanu Lamba       | Backend Developer  | 57 | API, database, backend bugs
Shubham Upadhyay | QA Engineer        | 42 | Testing, bug verification, QA intake

## System Roles
QA Lead:      Shubham Upadhyay (Odoo ID: 42)
PM:           (vacant — update when hired)
Lead:         Shubham Upadhyay (Odoo ID: 42)
Triage Owner: Shubham Upadhyay (Phase 3 — TRIAGE_AGENT)

Escalation path: QA Lead → PM (when joined)
Skills and skill files reference roles, not names. Update this block when roles change.

---

## Section 2: Project Routing Rules
Format: keyword(s) → Project name (Odoo Project ID)

One active project for all tickets:
default         → TiffinConnect (58)
frontend        → TiffinConnect (58)
backend         → TiffinConnect (58)
api             → TiffinConnect (58)
ui              → TiffinConnect (58)
database        → TiffinConnect (58)
bug             → TiffinConnect (58)

Project stages:
TiffinConnect (ID: 58) — opening stage: Backlog (ID: 347)

Rule: All tickets go to TiffinConnect regardless of domain.
When more projects are added, update this section.

---

## Section 3: Tag Taxonomy
Format: tag name | Odoo tag ID | when to use
Note: duplicate tag IDs exist in Odoo. Primary IDs listed below — always use these.
Fallback IDs (use only if primary returns "record not found"):
  frontend fallback: 43 | backend fallback: 1 | bug fallback: 45

frontend | 2  | UI changes, component issues, anything visible in browser
backend  | 44 | API, database, server-side logic, integrations
bug      | 4  | Confirmed broken behaviour — not questions or enhancements

Combination rules:
A broken UI element         → tags: frontend, bug
A broken API endpoint       → tags: backend, bug
A new UI feature            → tags: frontend (no bug)
A new API endpoint          → tags: backend (no bug)
Unknown domain              → tag: bug only, domain tag added at triage

---

## Section 4: Priority Rules
Format: condition → priority value (0=normal, 1=urgent)

customer-blocked + any environment    → urgent (1)
production down + any reporter        → urgent (1)
data loss risk + any environment      → urgent (1)
security issue + any environment      → urgent (1)
bug + production environment          → urgent (1)
bug + staging environment             → normal (0)
feature request + any environment     → normal (0)
improvement + any environment         → normal (0)
ci-failure + main/master branch       → urgent (1)
ci-failure + feature branch           → normal (0)

Conflict resolution: if multiple rules match, highest priority wins.
customer-blocked always overrides type-based rules.
Example: customer-blocked feature request → urgent (1), not normal (0).
Default: when in doubt → normal (0), flag for triage owner to review.

---

## Section 5: Learned Decisions
Auto-populated by Claude after sessions. Human reviews weekly.
Format: [date] | decision | confirmed by
Hard limit: 25 lines. Prune when Rule 8 flags it.

Empty — populated during sessions.

---

## Section 6: Known Exceptions
Things that break the normal rules.

No exceptions defined yet. Add as they are discovered.
Format: "situation → exception rule"

---

## Section 7: Out of Scope
Requirement types this system should NOT create tickets for.

infrastructure-only changes with no code impact
meeting notes and action points
third-party vendor tasks with no internal dev work
duplicate of an existing open ticket — link instead
questions that need a product decision before becoming a requirement

When input matches an out-of-scope category:
Do not create a ticket.
Respond in one line: "This is out of scope for ticket creation — [reason]. [Optional: suggest alternative]"
Example: "Meeting notes are out of scope — consider adding action points as Task tickets instead."
