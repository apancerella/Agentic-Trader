# CLAUDE.md

Standing instructions for any Claude Code session — interactive or a scheduled Routine — working in this repo. See `README.md` for the full strategy.

## Guardrails

1. **Propose + Confirm only.** Never place an order (equity or option) without the user explicitly confirming it in that conversation. Research, analyze, and propose — execution requires a human "yes."
2. **Agentic account only.** Trade exclusively through the Agentic Robinhood account (••••6245). Do not act on any other linked account.
3. **Respect the risk guardrails** in `README.md` on every proposal: max 10% of account value per position, max 5 concurrent open positions, -8% stop-loss, max 2 new proposals/week.
4. **Journal every routine run.** After any routine (scheduled or manual) touches the portfolio, watchlist, or a scan, append an entry to `memory/journal/<YYYY-MM-DD>.md` (create the file if today's doesn't exist yet) using the format in `memory/README.md`.
5. **Keep the watchlist current — in both places.** When the user approves adding, removing, or updating the thesis on a symbol, reflect it in `memory/watchlist.md` **and** mirror the add/remove to the Robinhood "Agentic Watchlist" (list_id `aaf2fa51-2240-4463-8af2-d8c8b0621302`, via `add_to_watchlist`/`remove_from_watchlist`) under the Agentic account, in the same turn. The repo file is the richer record (thesis, added date, status); the Robinhood list is the flat symbol list that should always match its rows.
6. **Auto-merge watchlist-only changes.** For a commit that touches only `memory/watchlist.md` (plus its journal entry) — e.g. from `/watchlist-add` or any other watchlist edit — commit, push, and merge it to `main` in the same turn without waiting for a separate "please merge" instruction. The user has pre-authorized this specifically so the repo file and the Robinhood watchlist never sit out of sync waiting on a merge. This does **not** extend to any other change (skills, routines, commands, code, docs) — those still wait for explicit confirmation before merging, per the repo's normal git workflow.

## Layout

- `routines/` — playbooks each recurring Routine follows (purpose, schedule, steps, guardrails, journal output)
- `memory/watchlist.md` — current watchlist and thesis per symbol
- `memory/journal/` — one markdown file per day, journaling routine activity and decisions
