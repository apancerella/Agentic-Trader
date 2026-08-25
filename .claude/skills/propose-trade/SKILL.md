---
name: propose-trade
description: Sizes and executes a trade (new position, add, trim, or exit) for the Agentic Robinhood account in the Agentic-Trader repo, checking it against every risk guardrail in CLAUDE.md/README.md — then places the order directly once those checks pass, journals the result, and reports it to the user. No separate human confirmation is required before execution (CLAUDE.md guardrail 1). Use this any time a specific trade is being considered, whether it comes out of daily-check, weekly-scan, or earnings-watch follow-ups, opportunity-scan's own initiative, or an ad hoc request like "buy some NVDA," "take profit on XYZ," or "enter the watchlist candidate we added last week." This is the one place position sizing, concurrent-position count, stop-loss, and the weekly trade cap get enforced, so every trade in this repo goes through the same checks instead of each routine re-deriving them.
---

# Size and Execute a Trade

This is the choke point every trade idea in this repo passes through — no matter which routine surfaced it — so the guardrails in `CLAUDE.md` get checked the same way every time, instead of each routine re-implementing the math slightly differently.

**This skill places the order itself once its checks pass.** Per `CLAUDE.md` guardrail 1, no separate user confirmation is required — the checks below (sizing, position cap, stop, weekly cap, chase/extension, earnings timing) are the approval. The user finds out what happened through the journal entry and a direct report in the conversation, not through a pre-trade yes/no.

## Steps

1. **Confirm the account.** This only ever applies to the Agentic account (••••6245). If the request is about any other linked account, stop and say so — this skill (and this repo) doesn't act on other accounts.

2. **Pull current state.**
   - `get_portfolio` — total account value (needed for position sizing).
   - `get_equity_positions` (and `get_option_positions` if relevant) — current open position count and existing exposure, including whether this symbol is already held.

3. **Check the weekly trade cap first — before doing the rest of the work.** Look at `memory/journal/` entries from the last 7 days for `**Proposals:**` sections that contain an actual trade (not "None", and not a watchlist-only entry from `weekly-scan`). If 7 new trades have already been made this week, stop here: journal that the weekly cap (max 7 new trades/week, from `CLAUDE.md`) has been reached, tell the user, and don't place a new one. (You can still discuss the idea qualitatively — just don't size and execute it.)

4. **Check the concurrent-position cap.** If this is a new position (not an add/trim/exit on an existing one) and current open positions are already at 7, stop and say so — same treatment as step 3.

5. **Size the position.** Max size = 20% of total account value (from `get_portfolio`), at your judgment — this is a ceiling, not a target; size smaller when the setup, conviction, or existing exposure calls for it. There's no minimum cash reserve requirement, so full deployment (up to the position-size and position-count caps) is allowed when it's warranted. Pull the current price (`get_equity_quotes` or `get_option_quotes`) and compute the share/contract count that fits inside the chosen budget. Round down to a whole share/contract — don't size fractionally beyond the cap.

6. **Set the stop.** Stop-loss = entry price × 0.92 (-8%), per `CLAUDE.md`. For options, note that a fixed percentage stop on premium behaves differently than on the underlying — call this out explicitly in the journal/report rather than silently applying the same math.

7. **Pull supporting signal.** Whatever's relevant and not already in hand: `get_equity_technical_indicators`, `get_equity_fundamentals`, `get_earnings_calendar` (never execute an entry into an earnings print without flagging it first — cross-check against what `earnings-watch` would find, and treat an unflagged upcoming print as a reason to hold off, not just a footnote).

8. **Check for chase/extension risk.** This matters most for candidates that come from `opportunity-scan`, since its momentum scan only ever surfaces names already up on the day — but check it for every entry, whatever the source. Look at the day's move (entry price vs. prior close) and RSI:
   - If the day's move is already large (roughly +7% or more) or RSI is at/approaching overbought (≥65), this is a real risk, not just color commentary: a stock can keep going, but a same-day spike this size commonly gives back several percent on ordinary profit-taking in the following days — with an -8% stop, that's a realistic chance of getting stopped out on noise before the underlying thesis (which may be completely intact) has time to play out.
   - This doesn't block execution on its own — genuine breakouts are sometimes worth entering right away, and there's no human in the loop to weigh it beforehand anymore. But **surface it explicitly and honestly** in both the journal entry and the report to the user, exactly as if someone were about to decide on it — this is the one place that read reaches a person, just after the trade instead of before.
   - This check is about entries only. It doesn't apply to a trim/exit on an existing position — taking profit into strength is a different decision.

9. **Draft the trade record** with:
   - Symbol, direction (buy/sell, new/add/trim/exit), one or two lines of rationale.
   - Entry price or condition.
   - Size — in shares/contracts, dollars, and % of account value.
   - Stop-loss level.
   - **Chase/extension read** (from step 8, for entries): the day's move and RSI, and a plain statement of whether this looked like a reasonable entry or an extended one.
   - Where this lands against the guardrails: e.g. "this is position 3 of 7" and "this is trade 2 of 7 this week."

10. **Place the order.** `review_equity_order` / `review_option_order` first if available to double check terms, then `place_equity_order` / `place_option_order`.

11. **Journal immediately**, including the actual fill (price, size, order id) — see Journal below.

12. **Report directly to the user in the conversation** — what was traded and why, size/stop, the chase/extension read if relevant, and the fill details. This is the point where the user actually sees the decision; don't bury it in the journal alone.

13. **Refresh the dashboard.** Rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL linked in `memory/README.md`, so the new/updated/closed position shows up. Skip this if step 3 or 4 blocked execution — nothing changed.

## Guardrails (from `CLAUDE.md` / `README.md`)

- Max 20% of account value per position (a ceiling — size at your own judgment, not automatically to the max)
- Max 7 concurrent open positions
- No minimum cash reserve — full deployment up to the caps above is allowed
- Stop-loss at -8% from entry
- Max 7 new trades per week
- Agentic account (••••6245) only
- **Autonomous execution:** once the checks above pass, this skill places the order directly — no separate confirmation step, per `CLAUDE.md` guardrail 1. `/sell-all` is the one exception in this repo (see that command).

## Journal

Use the `journal-entry` skill:

- **When a trade is executed:** Summary, Findings (signal that supports it, including the chase/extension read), Proposals (the full sized trade plus the actual fill — price, size, order id), Follow-ups if relevant. There's normally no "User decisions" section, since there's no separate decision to record — the fill itself is the outcome.
- **When a cap blocks a trade** (steps 3–4): still journal it — Summary + Findings (what was considered) + Follow-ups (revisit once the cap resets), so the block is visible in the record even though nothing was traded.

## Commit and push — but don't auto-merge

Commit the journal entry and push it, same as any other change in this repo. Even though executing the trade itself doesn't require a separate confirmation, landing its record on `main` still does: `CLAUDE.md` guardrails 6 and 7 don't cover this skill's output. Open a PR if the repo's workflow calls for one, then wait for the user's explicit "merge it" — that's the point where the user gives the trade record a deliberate look, just after the fact instead of before.
