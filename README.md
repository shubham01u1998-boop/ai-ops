# ai-ops

AI operations repository for the fiftyfive-tech Odoo ticket pipeline and Project Initiator toolchain.
Contains all Claude skill files, context, CI scripts, and templates.

## Structure

```
skills/core         — global rules (LAYER_0_GLOBAL), always loaded
skills/layers       — skill layers loaded on demand (FASTPATH, QA_INTAKE)
skills/commands     — slash command skills (/ticket-context)
skills/project-initiator — Project Initiator toolchain (DISCOVERY V1.0)
context             — TEAM_CONTEXT.md, PARKING_LOT.md, session state
docs                — system flow diagrams, demo scripts, test suites, design specs
tests/fixtures      — synthetic engagement fixtures for DISCOVERY testing
templates           — generic project template for replication
ci                  — GitLab CI scripts
CLAUDE.md           — Claude Code instructions for this repo
```

## Quick start

See `CLAUDE.md` for session-start protocol, skill routing, and MCP server setup.

## Running DISCOVERY

DISCOVERY extracts structured requirements from rough input documents and produces a `discovery.md` artifact through a guided conversation.

**Setup:**
1. Create an engagement folder anywhere on disk.
2. Inside it, create an `input/` subfolder.
3. Drop your input docs into `input/` — supported formats: `.pdf`, `.txt`, `.md`, `.markdown`.
4. Open Claude Code in that engagement folder.
5. Say: `run DISCOVERY`

**What DISCOVERY does:**
- Reads all docs in `input/` and extracts: project type, users, core problem, features, tech, timeline, constraints
- Surfaces conflicts (same topic stated differently across docs) and asks you to resolve them one at a time
- Asks critical gap questions — things downstream phases need that aren't in the docs
- Lets you defer any question to Open Questions
- Produces a draft `discovery.md`, lets you review and edit, then saves on your approval

**Output:** `discovery.md` written to the engagement folder root with sections: Project Context, Users, Core Problem, Features Mentioned, Constraints, Open Questions, Confidence Notes, Source Docs.

**Test fixtures:** `tests/fixtures/discovery/synthetic-01/` (SupplySync) and `tests/fixtures/discovery/synthetic-02/` (StyleMart) — run DISCOVERY against these to verify the skill before deploying changes.
