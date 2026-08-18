# Earnings Calendar Watch

**Purpose.** Make sure an earnings print never blindsides an open or proposed swing position — earnings volatility is a distinct risk from the technical/fundamental thesis.

**Schedule.** Weekly (proposed: Monday 7:00 AM ET, ahead of the trading week — confirm cadence before creating the Routine).

**Steps.**
1. Build the symbol set: current holdings (`get_equity_positions`) plus everything in `memory/watchlist.md`.
2. `get_earnings_calendar` for the coming week (and a short lookahead) for each symbol.
3. For any symbol with earnings in the window, flag it:
   - Open position → note the position and how close earnings is to now
   - Watchlist candidate → note that any new entry should account for earnings timing before it's proposed

**Guardrails.** Read-only — never places, modifies, or proposes a trade itself. It only flags risk for a human (or another routine's proposal) to account for.

**Journal output.** Append to `memory/journal/<date>.md`: symbols with upcoming earnings, whether each is a holding or watchlist candidate, and the flagged risk.
