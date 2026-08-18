# Opportunity Scan

**Purpose.** A more frequent, lighter-weight pass than `weekly-scan.md`: surface assets — from the current watchlist and from a fresh scan — that look like they have real upside potential right now, and report it directly to the user. This routine never modifies the watchlist or proposes a trade itself; it's purely a heads-up so the user can decide whether to act (via `/watchlist-add` or the `propose-trade` skill) themselves.

**Schedule.** Weekdays, 9:00 AM ET and 1:00 PM ET. Linked Routine: "Opportunity Scan (9am & 1pm ET)".

**Steps.**
1. Build the candidate set:
   - Every symbol currently on `memory/watchlist.md`.
   - A fresh scan for new candidates: `get_scans` for an existing bullish/momentum-oriented scan, `run_scan` on it (or `create_scan` if nothing suitable exists), same liquidity bar as `weekly-scan.md`.
2. For each candidate, pull `get_equity_technical_indicators` and `get_equity_historicals` (and `get_equity_fundamentals` for anything not already vetted by `weekly-scan.md`) to assess whether there's a real, current upside signal — not just "still on the list."
3. Filter hard. This runs twice a day — report only what actually changed or stands out since the last pass (new breakout, momentum turning up, approaching a level from the thesis, a new scan hit). A routine that reports the same unchanged symbols every run trains the user to ignore it.
4. For anything already held or on the watchlist, note where it sits against the guardrails in `CLAUDE.md` (room under the 5-position cap, 10% sizing, etc.) so the user has enough context to act immediately if they want to.
5. Report directly to the user in the conversation — symbol, the specific signal, current price/level, and one line on why it's worth a look now. If there's genuinely nothing new, say that plainly and keep it short; don't manufacture a report out of noise.

**Guardrails.** Strictly read-only — never calls `place_equity_order`/`place_option_order`, never edits `memory/watchlist.md` or the Robinhood "Agentic Watchlist" itself. At twice a day this would blow through the max-2-new-proposals/week and watchlist-change guardrails if it acted on its own, so it only ever reports; the user (or a follow-up `/watchlist-add` / `propose-trade`) does the acting.

**Journal output.** Append to `memory/journal/<date>.md`: which scan/watchlist symbols were checked, what stood out and why (or that nothing new stood out). No Proposals or User decisions sections — this routine doesn't produce either.
