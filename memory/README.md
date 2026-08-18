# Memory

Persistent state for the trading practice, read and written by routines and by any Claude Code session working in this repo.

## `watchlist.md`

Current watchlist: symbol, thesis, when it was added, and status (watching / proposed / position). Updated only when the user confirms a change (see `CLAUDE.md`).

**Mirrored to Robinhood.** This file's symbol list should always match the Robinhood custom watchlist named **"Agentic Watchlist"** (list_id `aaf2fa51-2240-4463-8af2-d8c8b0621302`) under the Agentic account — that's the source of truth on the Robinhood side (what shows up in the app), while this file carries the extra context (thesis, added date, status) Robinhood's watchlist doesn't. Any add/remove here must be mirrored there via `add_to_watchlist`/`remove_from_watchlist` in the same turn, per `CLAUDE.md`.

## `journal/`

One markdown file per day: `journal/YYYY-MM-DD.md`. Every routine run (or manual trading activity in this repo) appends an entry — create the day's file if it doesn't exist yet, never overwrite a prior entry.

**Entry template:**

```markdown
## <HH:MM ET> — <routine or activity name>

**Summary:** one or two lines on what this was.

**Findings:** what was observed (prices, indicators, scan hits, earnings dates, etc.)

**Proposals:** any trade or watchlist proposal made, with rationale, size, and stop if applicable.

**User decisions:** what the user approved, rejected, or changed.

**Follow-ups:** anything to revisit next time.
```

Omit a section if it doesn't apply (e.g. a read-only daily check has no Proposals).
