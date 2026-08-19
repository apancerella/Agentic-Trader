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
| [`daily-check.md`](daily-check.md) | Portfolio + watchlist snapshot, including performance-since-added on each watchlist pick | Weekdays, 4:30 PM ET |
| [`weekly-scan.md`](weekly-scan.md) | Screen for new swing candidates (junk-filtered), proactive catalyst-calendar research, and sector relative-strength ranking | Weekly, Sunday 6:00 PM ET |
| [`earnings-watch.md`](earnings-watch.md) | Flag earnings risk on watchlist/holdings | Weekly, Monday 7:00 AM ET |
| [`opportunity-scan.md`](opportunity-scan.md) | Surface upside (watchlist + junk-filtered fresh scan, with trend/pivot context and catalyst tie-ins) — and act on it: self-manages the watchlist, may surface a trade proposal | Weekdays, 10:00 AM, 12:00 PM & 3:30 PM ET |

All four are wired to Claude Code Routines now (see each playbook's Schedule line for the trigger ID; `opportunity-scan` has two, since a single cron expression can't mix the on-the-hour and half-past-the-hour firing times).

`opportunity-scan` is the one exception to "read-only": per `CLAUDE.md` guardrail 8, it can add/remove watchlist symbols on its own conviction and can invoke `propose-trade` when something clears the bar (after cross-checking `earnings-watch` and `weekly-scan` findings) — it still never places an order itself. They're all self-bound to the same session (`session_01WAYdETVewPXXJGZ9qTaTAa`) rather than firing into a fresh session each time — this account's org doesn't support attaching connectors (e.g. Robinhood) to fresh-session triggers, so self-binding to a session that already has the connector was the only way to guarantee tool access on every firing. The tradeoff: no proactive push notification when a routine fires (self-bind/persistent-session triggers don't support the `notifications` option), and every firing resumes that one conversation rather than starting clean.

Their cron expressions are all pinned to ET while EDT (UTC-4) is in effect — each playbook's Schedule line notes the UTC offset it'll need once EST (UTC-5) starts in November.

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
