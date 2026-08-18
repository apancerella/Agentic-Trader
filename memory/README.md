# Memory

Persistent state for the trading practice, read and written by routines and by any Claude Code session working in this repo.

## `watchlist.md`

Current watchlist: symbol, thesis, when it was added, and status (watching / proposed / position). Updated only when the user confirms a change (see `CLAUDE.md`).

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
