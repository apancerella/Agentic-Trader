# Routines

Each file in this folder is a **playbook**: a short spec for one recurring task. A playbook is executed either by a Claude Code Routine (a scheduled trigger that fires this prompt into a session) or manually, on request. Every playbook follows the same shape:

- **Purpose** — what question this routine answers
- **Schedule** — how often it runs (and the linked Routine, once one exists)
- **Steps** — which MCP tools to call, in what order
- **Guardrails** — the rules from `README.md` / `CLAUDE.md` that apply
- **Journal output** — what gets written to `memory/journal/<date>.md`

## Current playbooks

| Playbook | Purpose | Schedule |
|---|---|---|
| [`daily-check.md`](daily-check.md) | Portfolio + watchlist snapshot | Weekdays, 4:30 PM ET |
| [`weekly-scan.md`](weekly-scan.md) | Screen for new swing candidates | Weekly |
| [`earnings-watch.md`](earnings-watch.md) | Flag earnings risk on watchlist/holdings | Weekly |

Adding a new routine: write a playbook here first, then wire it to a Claude Code Routine (`create_trigger`) once the playbook is settled.
