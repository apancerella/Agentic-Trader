# Daily Portfolio + Watchlist Check

**Purpose.** A same-day pulse on the account and the watchlist — did anything move enough to matter before tomorrow's session.

**Schedule.** Weekdays, 4:30 PM ET (after market close). Linked Routine: "Daily Robinhood Portfolio Summary" (`trig_0159qNqU6AWs6kepqu4FSqBq`).

**Steps.**
1. `get_portfolio` and `get_equity_positions` on the Agentic account (••••6245) — total value, day's gain/loss, open positions.
2. `get_equity_orders` — any open orders still pending.
3. Read `memory/watchlist.md`. For each symbol, `get_equity_quotes` (and `get_equity_technical_indicators` if something looks notable) to check for a meaningful move (e.g. beyond typical daily noise) or a level relevant to its thesis.
4. Note anything that would need a proposal (approaching a stop, thesis invalidated, thesis playing out) — but do not propose or place a trade as part of this routine; flag it for follow-up.

**Guardrails.** Read-only — this routine never places or modifies an order. Position-sizing/stop-loss rules apply only when a separate proposal is made elsewhere.

**Journal output.** Append to `memory/journal/<date>.md`: portfolio snapshot (value, day P&L), open orders, watchlist symbols that moved and why it matters, follow-ups.
