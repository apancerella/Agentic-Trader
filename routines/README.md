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
| [`opportunity-scan.md`](opportunity-scan.md) | Report assets with potential upside right now (watchlist + fresh scan) | Weekdays, 9:00 AM & 1:00 PM ET |

Adding a new routine: write a playbook here first, then wire it to a Claude Code Routine (`create_trigger`) once the playbook is settled.

## Skills

Each playbook above is operationalized as a Claude Code Skill in `.claude/skills/`, so it runs the same way every time instead of being re-derived from prose on each invocation:

| Skill | Wraps |
|---|---|
| `daily-check` | `daily-check.md` |
| `weekly-scan` | `weekly-scan.md` |
| `earnings-watch` | `earnings-watch.md` |
| `opportunity-scan` | `opportunity-scan.md` |
| `propose-trade` | the trade-proposal step referenced by all three playbooks — enforces the guardrails in `CLAUDE.md` (position size, concurrent positions, stop-loss, weekly proposal cap) in one place |

There's also a `journal-entry` skill that every routine skill uses to append to `memory/journal/<date>.md` in the format from `memory/README.md`.

If a playbook doc and its skill ever disagree, the playbook doc is the source of truth — update the skill to match, don't silently follow the skill.
