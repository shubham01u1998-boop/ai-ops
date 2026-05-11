@"
# ai-ops

AI operations repository for the Odoo ticket pipeline.
Contains all Claude skill files, context, CI scripts, and templates.

## Structure

skills/core     — global rules, always loaded
skills/layers   — skill layers loaded on demand
context         — TEAM_CONTEXT.md, PARKING_LOT.md, session state
templates       — generic project template for replication
ci              — GitLab CI scripts
docs            — onboarding guides per role
CLAUDE.md       — Claude Code instructions for IDE usage
"@ | Out-File -FilePath README.md -Encoding utf8