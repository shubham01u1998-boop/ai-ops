# Project Initiator — README
# fiftyfive-tech internal consulting tooling
# V1.0: DISCOVERY skill only

## What this is

Project Initiator is a Claude Code toolchain for fiftyfive-tech consultants. It takes rough, partial, or informal input documents from a client engagement and converts them into structured project-starting documentation through a guided conversation.

V1.0 ships one skill: **DISCOVERY**. It reads your input docs, extracts everything it can, surfaces conflicts and gaps, asks you one question at a time to fill those gaps, and produces a clean `discovery.md` you've reviewed and approved.

## Folder convention

Each engagement gets its own named folder. If you work on multiple projects, they stay completely separate:

```
~/fiftyfive-engagements/
  supplysync/          ← one engagement
    input/             ← raw docs go here
    discovery.md       ← generated here when done
  retail-platform/     ← another engagement
    input/
    discovery.md
```

## 3-step quickstart

1. **Create a named engagement folder** at `~/fiftyfive-engagements/<client-name>/`. Inside it, create an `input/` subfolder and drop in your raw docs (PDFs, text files, markdown files). Word docs need to be converted to PDF or text first.

2. **Open Claude Code** in the `<client-name>/` folder (not in `input/`, not in `fiftyfive-engagements/`).

3. **Say:** `run DISCOVERY` — Claude detects the engagement name from the folder, reads your docs, and begins the guided conversation.

When done, `discovery.md` is saved in the engagement folder alongside `input/`.

## Current scope (V1.0)

V1.0 = DISCOVERY skill only.

DISCOVERY does **not**:
- Scope the MVP
- Recommend or choose a tech stack
- Create Odoo tickets
- Decide anything — it advises and asks, you decide

Those capabilities are planned for future phases (see design spec).

## Target vision

Full chain when complete: DISCOVERY → MVP_SYNTHESIZER → ARCH_PROPOSER → BACKLOG_GENERATOR → ROADMAP.

Design spec: `docs/superpowers/specs/2026-06-03-project-initiator-design.md`

## How to give feedback / report friction

After running DISCOVERY on a real engagement, note:
- What the tool did that surprised you (positively or negatively)
- Where you had to rephrase a question or answer
- What the output was missing that you had to add manually
- Where the question order felt wrong

Bring this to the next planning session. The friction list drives V1.1 scope.
