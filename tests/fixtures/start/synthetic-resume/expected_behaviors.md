# Expected Behaviors — synthetic-resume (Resume / Registry path)
# Tests START skill reading 3 mock engagement folders and routing correctly.
# Mock engagements: RetailEdge (ARCH_PROPOSER complete), SupplySync (MVP_SYNTHESIZER complete),
# VectorSeven (DISCOVERY complete)

## Pre-flight
- [ ] Scans subdirectories for project.md files
- [ ] Reads all 3 project.md files successfully
- [ ] Builds registry in memory from all 3

## Registry Display
- [ ] Shows all 3 engagements in the menu
- [ ] Each entry shows: name, client, type, stage, last session date
- [ ] RetailEdge shown as: ARCH_PROPOSER complete — 2026-06-04
- [ ] SupplySync shown as: MVP_SYNTHESIZER complete — 2026-05-30
- [ ] VectorSeven shown as: DISCOVERY complete — 2026-05-22
- [ ] Most recent first: RetailEdge (2026-06-04) at top, VectorSeven (2026-05-22) at bottom
- [ ] "N for new project" option shown at bottom of menu

## Resume Selection
- [ ] Consultant picks "1" (RetailEdge)
- [ ] Reads retailedge/session_state.md
- [ ] Shows: "RetailEdge — ARCH_PROPOSER complete"
- [ ] Shows next step: "run DOC_GENERATOR"
- [ ] Shows folder path: ~/fiftyfive-engagements/retailedge/
- [ ] Shows open items: "Oracle Retail POS integration method unconfirmed — source: ARCH_PROPOSER — blocks: OracleConnector"

## Stage Auto-detection (supplementary check)
- [ ] If a subdirectory has project.md missing but arch.md present → detects ARCH_PROPOSER complete
- [ ] If mvp-scope.md present (no arch.md) → detects MVP_SYNTHESIZER complete
- [ ] If only discovery.md present → detects DISCOVERY complete
- [ ] Asks consultant to confirm client name + type before writing retroactive project.md
- [ ] Shows confirmation before writing retroactive project.md

## Test Notes
- Run this simulation by presenting the 3 mock project.md and session_state.md files to
  the START skill as if they were found by scanning the parent directory.
- The subagent should verify registry display format, sort order (most recent first),
  and routing output for the selected engagement.
- Stage auto-detection items are supplementary — simulate a 4th folder with only arch.md
  present to test detection logic.
