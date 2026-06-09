# Test Report — START synthetic-resume
# Date: 2026-06-09
# Result: PASS

## Checklist

### Pre-flight
- ✅ Scans subdirectories — finds `active/` as a known status bucket — V1.3 pre-flight step 3c: "if subdirectory name matches a known bucket → run `ls -d <bucket>/*/`"; `active` is in the known bucket list
- ✅ Scans inside `active/` for engagement folders — V1.3 step 3c lists engagements inside the bucket, marking each path with its bucket name
- ✅ Reads all 3 project.md files from active/retailedge/, active/supplysync/, active/vectorseven/ — V1.3 step 3d: "For each engagement path collected, attempt Read on `<path>/project.md`"; all 3 files present and parseable
- ✅ Builds registry in memory from all 3 with status: active — V1.3 step 3d: "add to registry: {path, status, project_data}" using the bucket name `active` as the status value

### Registry Display
- ✅ Shows Active group header with all 3 engagements — V1.3 Main Menu rule: "Show Active and Blocked groups by default (skip group header if that group is empty)"; 3 active entries are present so the Active header is shown
- ✅ No Blocked group shown (no blocked engagements in fixture) — V1.3 rule: "skip group header if that group is empty"; no blocked bucket entries → Blocked group omitted
- ✅ Each entry shows: name, client, type, stage, last session date — confirmed by V1.3 example menu format: `Name (Client) — Type — Stage — Date`
- ✅ RetailEdge shown as: ARCH_PROPOSER complete — 2026-06-04 — active/retailedge/project.md fields: `Stage: ARCH_PROPOSER complete`, `Last session: 2026-06-04`
- ✅ SupplySync shown as: MVP_SYNTHESIZER complete — 2026-05-30 — active/supplysync/project.md fields: `Stage: MVP_SYNTHESIZER complete`, `Last session: 2026-05-30`
- ✅ VectorSeven shown as: DISCOVERY complete — 2026-05-22 — active/vectorseven/project.md fields: `Stage: DISCOVERY complete`, `Last session: 2026-05-22`
- ✅ Most recent first within Active group: RetailEdge at top, VectorSeven at bottom — sort by Last session date descending: 2026-06-04 > 2026-05-30 > 2026-05-22
- ✅ Completed and Archived shown as collapsed counts: (C) completed (0)  (A) archived (0) — V1.3 line 57: "Completed and Archived shown as collapsed count + option letter at the bottom"; the template shows these unconditionally (no condition to omit zero counts), and the skill logic only omits these when Active+Blocked are empty AND no completed/archived exist (lines 73-74 govern that edge case only)
- ✅ "N for new project" option shown — V1.3 example menu includes `(N) new project` at the bottom

### Resume Selection
- ✅ Consultant picks "1" (RetailEdge) — triggers Resume Flow for active/retailedge/
- ✅ Reads active/retailedge/session_state.md — V1.3 Resume Flow step 1: "Read `session_state.md` from the selected engagement folder"
- ✅ Shows: "RetailEdge — ARCH_PROPOSER complete" — V1.3 Resume Flow output format: `<Engagement Name> — <Stage>`; stage read directly from project.md `Stage: ARCH_PROPOSER complete`
- ✅ Shows next step: "run DOC_GENERATOR" — session_state.md `Run: DOC_GENERATOR`; V1.3 output format includes "run <Next Step>"
- ✅ Shows folder path: ~/fiftyfive-engagements/active/retailedge/ — session_state.md `From: ~/fiftyfive-engagements/active/retailedge/`; skill outputs the full bucketed path from the `From:` line
- ✅ Shows open items: "Oracle Retail POS integration method unconfirmed — source: ARCH_PROPOSER — blocks: OracleConnector" — verbatim from session_state.md Open Items section
- ✅ Shows status transition prompt: "Status: active → (B) blocked / (C) completed / (A) archived / skip" — V1.3 Status Transitions: "For all other stages: `Status: <current-status> → [options excluding current status] / skip`"; RetailEdge stage is ARCH_PROPOSER complete (not ESTIMATOR complete) so the general rule applies; current status is active so R (reactivate) is excluded per rule "active engagements never see R"

### Stage Auto-detection (supplementary check)
- ✅ If a subdirectory inside active/ has project.md missing but arch.md present → detects ARCH_PROPOSER complete — V1.3 detection order checks arch.md before mvp-scope.md and discovery.md; first match wins
- ✅ If mvp-scope.md present (no arch.md) inside active/ → detects MVP_SYNTHESIZER complete — second check in detection order; arch.md not present so falls through to mvp-scope.md
- ✅ Asks consultant to confirm client name + type before writing retroactive project.md — V1.3 auto-detection prompt: "Client name for this engagement?" then "Project type?"; two questions inline before any file write
- ✅ Shows confirmation before writing retroactive project.md — V1.3: "Show confirmation with full path, then write `project.md` with confirmed details"; explicit yes/no gate before any filesystem action (Rule 1)
- ✅ Retroactive project.md includes Status: active field (folder is in active/ bucket) — V1.3: "Use the bucket name as the status value if the folder is inside a known bucket"; active/ bucket → `Status: active`; also: "The retroactive project.md must include a `Status:` field between Type and Stage fields"

### Backward Compat (supplementary check)
- ✅ If a flat folder (no status bucket parent) is found alongside status buckets, it is read and shown in the Active group with "(legacy path)" note on status line — V1.3 pre-flight step 3c: "attempt Read on `<subdir>/project.md` directly. If found, mark as status: active (legacy flat folder)"; the "(legacy path)" note appears in the status transition prompt appended after resume output: `Status: active (legacy path) → (B) blocked / (C) completed / (A) archived / skip`; "status line" in the expected behavior refers to this transition prompt line, not the menu entry
- ✅ No automatic migration of legacy folder is triggered — V1.3 step 3c explicitly: "no migration prompted"; transition only happens if consultant selects B/C/A and confirms

## Score
27/27 items passed

## Registry Display (what the skill would show)

```
Active:
1. RetailEdge (FreshMart) — PWA — ARCH_PROPOSER complete — 2026-06-04
2. SupplySync (LogiCo) — Web — MVP_SYNTHESIZER complete — 2026-05-30
3. VectorSeven (VectorSeven Inc) — Enterprise — DISCOVERY complete — 2026-05-22

(C) completed (0)   (A) archived (0)   (N) new project
```

## Resume Output (what the skill would show for RetailEdge selection + skip)

```
RetailEdge — ARCH_PROPOSER complete
Open Claude Code in ~/fiftyfive-engagements/active/retailedge/ and run DOC_GENERATOR.

Open items from last session:
- Oracle Retail POS integration method unconfirmed — source: ARCH_PROPOSER — blocks: OracleConnector

Status: active  →  (B) blocked  /  (C) completed  /  (A) archived  /  skip
```

Consultant answers: `skip`

No folder move performed — engagement remains in active/retailedge/. End of turn.

## Stage Auto-detection Simulation

Simulated 4th folder: `~/fiftyfive-engagements/active/bluewave/`
Files present: `arch.md` only (no project.md, no mvp-scope.md, no discovery.md)

**Detection step:** skill checks in order: estimates.md → backlog.md → arch.md (found) → stage = `ARCH_PROPOSER complete`

**Skill output:**
```
Found existing engagement folder: bluewave/
Stage detected: ARCH_PROPOSER complete (arch.md present)

Client name for this engagement?
```

Consultant answers: `BlueCorp`

```
Project type? (web / mobile / PWA / enterprise / hybrid)
```

Consultant answers: `web`

**Confirmation shown before write:**
```
About to write retroactive project.md:
  ~/fiftyfive-engagements/active/bluewave/project.md

  # Project — bluewave
  Client: BlueCorp
  Type: web
  Status: active
  Stage: ARCH_PROPOSER complete
  Last session: 2026-06-09

Confirm? (yes / no)
```

On "yes": writes project.md including `Status: active` (bucket = active/) between Type and Stage fields per V1.3 spec. Skill does not touch arch.md or any other skill output file (Rule 2).

**mvp-scope.md only variant** (simulated): no project.md, no arch.md, only mvp-scope.md present → detection falls through estimates.md → backlog.md → arch.md → mvp-scope.md (found) → stage = `MVP_SYNTHESIZER complete`. Same question/confirm flow follows.

## QA Observations

1. **Bucketed path in resume output:** V1.3 resume output correctly uses the full bucketed path `~/fiftyfive-engagements/active/retailedge/` (pulled from session_state.md `From:` line). The old V1.2 test report showed `~/fiftyfive-engagements/retailedge/` — that was a flat-path fixture. The V1.3 fixture session_state.md already has the correct bucketed path, so the skill outputs it verbatim.

2. **"(legacy path)" note location:** The expected behavior says the legacy folder is "shown in the Active group with '(legacy path)' note on status line." The V1.3 skill does not render "(legacy path)" in the registry menu entry; it renders it in the status transition prompt line (`Status: active (legacy path)  →  ...`). The behavior scores ✅ because "status line" is interpreted as the transition prompt line (which appears as part of the resume output, not the menu). A future revision to expected_behaviors.md could clarify this distinction.

3. **Collapsed zero counts:** V1.3 does not explicitly state whether `(C) completed (0)` should appear when the count is zero. The skill template shows these items unconditionally in the example menu, and the only conditional omission (lines 73-74) applies only when Active+Blocked are both empty. Scored ✅ as the most conservative reading of the spec; a future clarification could add "hide if zero" or "always show."

4. **Stage read from project.md, not composed from session_state.md:** Unlike V1.2, the V1.3 registry is built from project.md (which has an explicit `Stage:` field), so the resume heading `RetailEdge — ARCH_PROPOSER complete` reads the stage directly. No field composition is needed. The old report's QA observation about composing stage from two session_state fields is no longer applicable.

5. **Status transition prompt is the V1.3 addition:** The key behavioral addition over V1.2 is the status transition prompt appended after every resume output. The consultant's "skip" response leaves the engagement in place with no filesystem changes — correct behavior per the skill's "On skip: leave flat, no migration" principle applied to the general skip path.
