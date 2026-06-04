# Expected Behaviors — synthetic-new (New Project Creation path)
# Tests START skill against a clean new project setup.
# Simulated consultant answers: Client="FreshMart", Engagement="RetailEdge", Type="PWA"

## Pre-flight
- [ ] START runs without being asked (no command needed beyond "run START")
- [ ] Does NOT create folders before checking current directory for project.md
- [ ] No project.md found in current dir → proceeds to Main Menu / new project flow
- [ ] No engagement folders found → skips menu, goes directly to new project questions

## New Project Questions
- [ ] Asks for client name first (before engagement name)
- [ ] Asks for engagement name second
- [ ] Asks for project type third
- [ ] Questions asked one at a time — not all at once
- [ ] Project type prompt lists options: web / mobile / PWA / enterprise / hybrid

## Confirmation Gate
- [ ] Shows full folder paths before creating anything:
      ~/fiftyfive-engagements/retailedge/
      ~/fiftyfive-engagements/retailedge/input/
      ~/fiftyfive-engagements/retailedge/project.md
      ~/fiftyfive-engagements/retailedge/session_state.md
- [ ] Asks "Confirm? (yes / no)"
- [ ] Does NOT create any files before confirmation
- [ ] On "no": outputs "Cancelled — no folders created." and stops

## Files Created (on confirm)
- [ ] Folder slug is lowercase, hyphens for spaces, no special chars: "RetailEdge" → retailedge | "Vector Seven" → vector-seven
- [ ] project.md written with correct content:
      # Project — RetailEdge
      Client: FreshMart
      Type: PWA
      Started: <today: YYYY-MM-DD>
      Stage: not started
      Last session: <today: YYYY-MM-DD>
- [ ] session_state.md written with correct content:
      # Session State — RetailEdge
      # Updated: <today: YYYY-MM-DD>
      ## Current Stage
      Last completed: none
      Status: not started
      ## Next Step
      Run: DISCOVERY
      From: ~/fiftyfive-engagements/retailedge/
      ## Open Items
      (none)
      ## Notes
- [ ] input/ subfolder created

## Handoff Output
- [ ] Outputs path of created folder
- [ ] Tells consultant to drop docs in input/ subfolder
- [ ] Tells consultant to open Claude Code in retailedge/ and say "run DISCOVERY"

## Test Notes
- This fixture simulates running START from a parent directory with no existing engagements.
- The skill uses Bash mkdir to create folders — in a test simulation, the subagent
  should verify the INTENDED behavior (questions, confirmation, file content) rather than
  actually creating folders on the test machine.
- Subagent should write what project.md and session_state.md WOULD contain if created.
