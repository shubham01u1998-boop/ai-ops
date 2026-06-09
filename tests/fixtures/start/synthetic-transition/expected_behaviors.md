# Expected Behaviors — synthetic-transition (Status Transition path)
# Tests START skill status transition: chain-complete auto-suggest + move execution.
# Mock engagement: DecorConnect (ESTIMATOR complete) in active/ bucket.
# Simulated consultant answers: select DecorConnect (1), then C (completed), then yes.

## Pre-flight
- [ ] Finds active/ bucket, reads active/decorconnect/project.md
- [ ] Builds registry with 1 engagement: DecorConnect, status: active

## Registry Display
- [ ] Shows DecorConnect in Active group: ESTIMATOR complete — 2026-06-08
- [ ] Numbers: 1. DecorConnect

## Resume Output
- [ ] Consultant picks "1" (DecorConnect)
- [ ] Shows: "DecorConnect — ESTIMATOR complete"
- [ ] Shows: "Chain complete — all skills done."
- [ ] Shows chain-complete status prompt (only C and skip offered):
      "Status: active  →  (C) mark completed  /  skip"
- [ ] Does NOT offer (B) blocked or (A) archived in the chain-complete prompt

## Transition — Consultant selects C
- [ ] Shows confirmation block:
      Move ~/fiftyfive-engagements/active/decorconnect/
        → ~/fiftyfive-engagements/completed/decorconnect/
      Update project.md  Status: active → completed
      Update session_state.md  From: path
      Confirm? (yes / no)
- [ ] Does NOT move any files before confirmation

## Transition — Consultant confirms yes
- [ ] Runs: mkdir -p ~/fiftyfive-engagements/completed/
- [ ] Runs: mv ~/fiftyfive-engagements/active/decorconnect ~/fiftyfive-engagements/completed/decorconnect
- [ ] Reads project.md after move and updates Status: active → completed
- [ ] Reads session_state.md after move and updates From: path (active → completed)
- [ ] Outputs: "Done. ~/fiftyfive-engagements/completed/decorconnect/ — Status updated to completed."

## Transition — Consultant selects skip
- [ ] No move, no file changes
- [ ] No output beyond the resume display

## Manual status change (supplementary — non-chain-complete engagement)
- [ ] For an engagement NOT at ESTIMATOR complete, full 4-option prompt shown:
      "Status: active  →  (B) blocked  /  (C) completed  /  (A) archived  /  skip"
- [ ] Selecting B shows correct confirmation with blocked/ destination
- [ ] Selecting A shows correct confirmation with archived/ destination
- [ ] Current status not offered as a target option

## Legacy flat folder transition (supplementary)
- [ ] A flat folder (no bucket parent) shows: "Status: active (legacy path)"
- [ ] Selecting a status change for a legacy folder moves it into the correct bucket
- [ ] After move, project.md Status field written (adds field if not present)

## Test Notes
- Simulate by presenting DecorConnect files to START as if found in the active/ bucket.
- For move steps: verify INTENDED bash commands shown in confirmation — not actual filesystem ops.
- Supplementary checks: simulate additional folder states inline to test each path.
