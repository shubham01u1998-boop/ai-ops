# Expected Behaviors — synthetic-resume (Resume / Registry path)
# Tests START skill reading 3 mock engagement folders in active/ bucket and routing correctly.
# Mock engagements: RetailEdge (ARCH_PROPOSER complete), SupplySync (MVP_SYNTHESIZER complete),
# VectorSeven (DISCOVERY complete) — all in active/ bucket.

## Pre-flight
- [ ] Scans subdirectories — finds `active/` as a known status bucket
- [ ] Scans inside `active/` for engagement folders
- [ ] Reads all 3 project.md files from active/retailedge/, active/supplysync/, active/vectorseven/
- [ ] Builds registry in memory from all 3 with status: active

## Registry Display
- [ ] Shows Active group header with all 3 engagements
- [ ] No Blocked group shown (no blocked engagements in fixture)
- [ ] Each entry shows: name, client, type, stage, last session date
- [ ] RetailEdge shown as: ARCH_PROPOSER complete — 2026-06-04
- [ ] SupplySync shown as: MVP_SYNTHESIZER complete — 2026-05-30
- [ ] VectorSeven shown as: DISCOVERY complete — 2026-05-22
- [ ] Most recent first within Active group: RetailEdge at top, VectorSeven at bottom
- [ ] Completed and Archived shown as collapsed counts: (C) completed (0)  (A) archived (0)
- [ ] "N for new project" option shown

## Resume Selection
- [ ] Consultant picks "1" (RetailEdge)
- [ ] Reads active/retailedge/session_state.md
- [ ] Shows: "RetailEdge — ARCH_PROPOSER complete"
- [ ] Shows next step: "run DOC_GENERATOR"
- [ ] Shows folder path: ~/fiftyfive-engagements/active/retailedge/
- [ ] Shows open items: "Oracle Retail POS integration method unconfirmed — source: ARCH_PROPOSER — blocks: OracleConnector"
- [ ] Shows status transition prompt: "Status: active → (B) blocked / (C) completed / (A) archived / skip"

## Stage Auto-detection (supplementary check)
- [ ] If a subdirectory inside active/ has project.md missing but arch.md present → detects ARCH_PROPOSER complete
- [ ] If mvp-scope.md present (no arch.md) inside active/ → detects MVP_SYNTHESIZER complete
- [ ] Asks consultant to confirm client name + type before writing retroactive project.md
- [ ] Shows confirmation before writing retroactive project.md
- [ ] Retroactive project.md includes Status: active field (folder is in active/ bucket)

## Backward Compat (supplementary check)
- [ ] If a flat folder (no status bucket parent) is found alongside status buckets,
      it is read and shown in the Active group with "(legacy path)" note on status line
- [ ] No automatic migration of legacy folder is triggered

## Test Notes
- Run this simulation by presenting the 3 mock project.md and session_state.md files from
  the active/ bucket to the START skill as if scanned from the parent directory.
- The subagent should verify: 2-level scan behavior, grouped menu format (Active/Blocked),
  collapsed completed/archived, sort order, routing output, and status transition prompt.
- Stage auto-detection and backward compat items are supplementary — simulate an extra
  folder to test each case.
