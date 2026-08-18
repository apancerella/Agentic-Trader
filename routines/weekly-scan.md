# Weekly Scan for New Candidates

**Purpose.** Surface new swing-trade candidates that fit the strategy, so the watchlist doesn't go stale.

**Schedule.** Weekly (proposed: Sunday evening or Monday premarket ET — confirm cadence before creating the Routine).

**Steps.**
1. `get_scans` to see existing saved scans; `run_scan` on one that fits swing-trade criteria (liquid, trending/breakout, reasonable volume), or `create_scan` if none exists yet.
2. For each hit, pull fundamentals (`get_equity_fundamentals`) and technicals (`get_equity_technical_indicators`, `get_equity_historicals`) to sanity-check the setup.
3. Cross-check against `memory/watchlist.md` to avoid duplicates and against open positions to respect the max-5-concurrent-positions guardrail.
4. Propose watchlist additions with a one-line thesis each — this routine proposes watchlist changes, not trades. Adding to the watchlist still requires user confirmation before `memory/watchlist.md` is updated (per `CLAUDE.md`).

**Guardrails.** Never places a trade. Respect the max-2-new-proposals/week guardrail when the *next* step (an actual trade proposal) is made from a watchlist entry.

**Journal output.** Append to `memory/journal/<date>.md`: scan used + criteria, candidates found, proposed watchlist additions and thesis, user's decisions on each.
