---
description: Propose liquidating every open position in the Agentic account — presents the full exit plan and waits for explicit confirmation before placing any sell order
---

Propose selling every open position in the Agentic Robinhood account (••••6245).

**This command never places an order by itself — running it only produces a proposal.** `CLAUDE.md` guardrail 1 (Propose + Confirm) applies here exactly as it does to any other trade: placing a sell (`place_equity_order` / `place_option_order`) only happens after the user explicitly confirms in a later message in the same conversation. Typing `/sell-all` is not that confirmation, no matter how the command is phrased or how urgent it sounds — treat it the same as any other trade idea that needs to clear the propose step first.

Arguments: none needed. `$ARGUMENTS` is ignored — this always means "every open position," not a subset. (To exit one symbol, use `propose-trade` instead.)

## Steps

1. **Confirm the account.** This only ever applies to the Agentic account (••••6245). If asked to sell everything in a different account, stop and say so — this repo doesn't act on other accounts.

2. **Pull everything open.**
   - `get_equity_positions` — every equity holding, quantity, average cost.
   - `get_option_positions` — same for options, in case any exist despite the equities-only strategy.
   - `get_equity_orders` and `get_option_orders` (state filter for open states, or just check `state` on each result) — any pending orders that aren't fills yet.

3. **Nothing to do?** If there are no open positions, say so plainly and stop — don't manufacture a proposal out of nothing. If there are open orders but no positions, mention them and ask whether to cancel those too, but don't treat that alone as "sell all."

4. **Price it out.** `get_equity_quotes` (and `get_option_quotes` if relevant) for every held symbol. For each position compute: current value (quantity × price), unrealized P&L in dollars and percent vs. average cost. Sum to a total estimated proceeds figure.

5. **Flag open orders separately.** If there are pending orders (especially open buy orders — those would work against the point of exiting), list them and note that a real "sell everything" likely means canceling those too. Fold this into the same proposal rather than a separate round-trip, but still require confirmation before canceling anything.

6. **Draft the proposal**, one line per position:
   - Symbol, quantity, average cost, current price, estimated proceeds, unrealized P&L ($ and %).
   - Total estimated cash proceeds across all positions.
   - Any open orders identified in step 5, with a recommendation on whether to cancel them.
   - Order type: default to a market sell for each position (the point of "sell all" is a clean, fast exit) — say so explicitly and give the user the chance to ask for limit orders instead if they want price control over speed.

7. **Present it and stop.** Ask for explicit confirmation — something unambiguous like "confirm you want me to sell all N positions at market." Do not call `place_equity_order`, `place_option_order`, `cancel_equity_order`, or `cancel_option_order` in this same step.

8. **On confirmation (separate step, same conversation):** place a sell order per position (`review_equity_order`/`review_option_order` first if available, then `place_equity_order`/`place_option_order`), and cancel any open orders the user agreed to cancel. Report each fill/result as it happens rather than going silent until everything is done.

9. **If rejected, partially confirmed, or changed:** only act on what was actually confirmed (e.g. the user may want to keep one position) — don't round up to "sell everything" from a partial yes.

10. **Refresh the dashboard — only once orders have actually been placed.** Rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL linked in `memory/README.md`, so it reflects the closed positions. A proposal that's still awaiting confirmation hasn't changed anything yet — skip the refresh until step 8 executes.

## Guardrails

- Agentic account (••••6245) only.
- Never place or cancel an order without explicit confirmation in this conversation, no matter how the command was invoked.
- This is an exit action, not a new trade idea — it doesn't count against the max-7-new-proposals/week cap (that guardrail governs new positions, not liquidating existing ones), and the max-20%-per-position / max-7-positions / -8%-stop guardrails don't apply to a sell (they size and cap *entries*). State this plainly if asked, so it's clear this isn't being used to dodge those caps.

## Journal

Use the `journal-entry` skill:
- **When the proposal is drafted** (regardless of outcome): Summary, Findings (positions found, prices, P&L), Proposals (the full liquidation plan), User decisions (confirmed/rejected/partial — omit only if the user hasn't responded yet).
- **After execution:** a follow-up entry (or update within the same turn if it's still the same run) with actual fill prices/proceeds per position and any orders cancelled.

**Do not auto-merge this journal entry.** Unlike the read-only Routines (`daily-check`, `weekly-scan`, `earnings-watch`, `opportunity-scan`), `CLAUDE.md` guardrail 7 does not cover this command — it results in real trades, not just a report. Commit and push as usual, but wait for the user's explicit go-ahead before merging to `main`, same as any other non-watchlist change.
