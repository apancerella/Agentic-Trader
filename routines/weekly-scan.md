# Weekly Scan for New Candidates

**Purpose.** Surface new swing-trade candidates that fit the strategy, so the watchlist doesn't go stale.

**Schedule.** Weekly, Sunday 6:00 PM ET (ahead of Monday's earnings-watch pass and the trading week). Linked Routine: "Weekly Scan (Sun 6pm ET)" (`trig_013fd83SVdupRpgnAjPwAPCr`), self-bound to the session at `session_01WAYdETVewPXXJGZ9qTaTAa`. Cron `0 22 * * 0` (UTC) — correct for 6 PM ET while EDT is in effect; needs updating to `0 23 * * 0` when clocks fall back to EST in November.

**Steps.**
1. `get_scans` to see existing saved scans; `run_scan` on one that fits swing-trade criteria (liquid, trending/breakout, reasonable volume), or `create_scan` if none exists yet. Drop any hit with an empty `market_cap` field first — the mechanical signature of leveraged/inverse ETFs, ETNs, and basket/trust products; note the count skipped.
2. **Proactive catalyst research**, independent of the scan: check for upcoming catalysts (FDA decisions, index rebalances, M&A chatter, major conferences, relevant macro releases) for current watchlist symbols and their sectors via `get_earnings_calendar`/`search`/`get_equity_fundamentals`, even if nothing has moved yet — this is what `opportunity-scan` reads back to connect a later price move to a known reason.
3. **Sector relative-strength check:** pull performance over the last week and month for a fixed sector-ETF basket (XLK, XLF, XLE, XLV, XLY, XLP, XLI, XLB, XLU, XLRE, XLC), rank them, and journal the ranking to catch rotations proactively.
4. For each surviving scan hit, pull fundamentals (`get_equity_fundamentals`) and technicals (`get_equity_technical_indicators`, `get_equity_historicals`, including SMA 50/200 trend context and the nearest `pivot_points` level) to sanity-check the setup.
5. Cross-check against `memory/watchlist.md` to avoid duplicates and against open positions to respect the max-5-concurrent-positions guardrail.
6. Propose watchlist additions with a one-line thesis each — this routine proposes watchlist changes, not trades. Adding to the watchlist still requires user confirmation before `memory/watchlist.md` is updated (per `CLAUDE.md`); on approval, capture the current price via `get_equity_quotes` for the new "Price at Add" column.

**Guardrails.** Never places a trade. Respect the max-2-new-proposals/week guardrail when the *next* step (an actual trade proposal) is made from a watchlist entry.

**Journal output.** Append to `memory/journal/<date>.md`: scan used + criteria, hits skipped as junk, upcoming catalysts found, the sector relative-strength ranking, candidates found (with trend/pivot context), proposed watchlist additions and thesis, user's decisions on each.

**Dashboard output.** After journaling (and merging any watchlist changes), rebuild and republish the "Agentic Trading Desk" Artifact (see `.claude/skills/trading-dashboard/`) so it reflects this run.
