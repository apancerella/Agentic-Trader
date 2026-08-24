---
name: propose-trade
description: Drafts a trade proposal (new position, add, trim, or exit) for the Agentic Robinhood account in the Agentic-Trader repo, checking it against every risk guardrail in CLAUDE.md/README.md before presenting it — and never places the order itself. Use this any time a specific trade is being considered, whether it comes out of daily-check, weekly-scan, or earnings-watch follow-ups, or from an ad hoc request like "should I buy some NVDA calls," "let's take profit on XYZ," or "propose an entry on the watchlist candidate we added last week." This is the one place position sizing, concurrent-position count, stop-loss, and the weekly proposal cap get enforced, so every trade idea in this repo goes through the same checks instead of each routine re-deriving them.
---

# Propose a Trade

This is the choke point between "here's an idea" and "here's a proposal a human can say yes/no to." Every trade idea in this repo — no matter which routine surfaced it — should pass through here so the guardrails in `CLAUDE.md` get checked the same way every time, instead of each routine re-implementing the math slightly differently.

**This skill never places an order.** It ends at a proposal. Placing the order (`place_equity_order` / `place_option_order`) only happens after the user explicitly confirms in the same conversation — that confirmation is a separate, later step, not part of drafting the proposal.

## Steps

1. **Confirm the account.** This only ever applies to the Agentic account (••••6245). If the request is about any other linked account, stop and say so — this skill (and this repo) doesn't act on other accounts.

2. **Pull current state.**
   - `get_portfolio` — total account value (needed for position sizing).
   - `get_equity_positions` (and `get_option_positions` if relevant) — current open position count and existing exposure, including whether this symbol is already held.

3. **Check the weekly proposal cap first — before doing the rest of the work.** Look at `memory/journal/` entries from the last 7 days for `**Proposals:**` sections that contain an actual trade proposal (not "None", and not a watchlist-only proposal from `weekly-scan`). If 7 new trade proposals have already been made this week, stop here: tell the user the weekly cap (max 7 new proposals/week, from `CLAUDE.md`) has been reached, and don't draft a new one. (You can still discuss the idea qualitatively — just don't produce a formal sized proposal.)

4. **Check the concurrent-position cap.** If this proposal is a new position (not an add/trim/exit on an existing one) and current open positions are already at 7, stop and say so — same treatment as step 3.

5. **Size the position.** Max size = 20% of total account value (from `get_portfolio`), at your judgment — this is a ceiling, not a target; size smaller when the setup, conviction, or existing exposure calls for it. There's no minimum cash reserve requirement, so full deployment (up to the position-size and position-count caps) is allowed when it's warranted. Pull the current price (`get_equity_quotes` or `get_option_quotes`) and compute the share/contract count that fits inside the chosen budget. Round down to a whole share/contract — don't propose fractional sizing that exceeds the cap.

6. **Set the stop.** Stop-loss = entry price × 0.92 (-8%), per `CLAUDE.md`. For options, note that a fixed percentage stop on premium behaves differently than on the underlying — call this out explicitly rather than silently applying the same math, and ask the user how they want to define risk if it's not obvious.

7. **Pull supporting signal.** Whatever's relevant and not already in hand: `get_equity_technical_indicators`, `get_equity_fundamentals`, `get_earnings_calendar` (don't propose an entry into an earnings print without flagging it — cross-check against what `earnings-watch` would find).

8. **Check for chase/extension risk.** This matters most for candidates that come from `opportunity-scan`, since its momentum scan only ever surfaces names already up on the day — but check it for every entry proposal, whatever the source. Look at the day's move (entry price vs. prior close) and RSI:
   - If the day's move is already large (roughly +7% or more) or RSI is at/approaching overbought (≥65), this is a real risk, not just color commentary: a stock can keep going, but a same-day spike this size commonly gives back several percent on ordinary profit-taking in the following days — with an -8% stop, that's a realistic chance of getting stopped out on noise before the underlying thesis (which may be completely intact) has time to play out.
   - Don't let this silently block a proposal — genuine breakouts are sometimes worth entering right away. But **surface it explicitly** in the proposal (step 9) rather than leaving the user to notice the extension themselves, and give an honest read on whether entering now vs. waiting for consolidation looks better here.
   - This check is about entries only. It doesn't apply to a trim/exit on an existing position — taking profit into strength is a different decision.

9. **Draft the proposal** with:
   - Symbol, direction (buy/sell, new/add/trim/exit), one or two lines of rationale.
   - Entry price or condition.
   - Size — in shares/contracts, dollars, and % of account value.
   - Stop-loss level.
   - **Chase/extension read** (from step 8, for entries): the day's move and RSI, and a plain statement of whether this looks like a reasonable entry now or one worth waiting on.
   - Where this lands against the guardrails: e.g. "this would be position 3 of 7" and "this is proposal 2 of 2 allowed this week."

10. **Present it and stop.** Ask the user to confirm, reject, or adjust. Do not call any order-placing tool in this same step.

11. **On confirmation (separate step, same conversation):** place the order (`place_equity_order` or `place_option_order`, using `review_equity_order`/`review_option_order` first if available to double check terms), then move to journaling.

12. **Refresh the dashboard — only once an order has actually been placed.** Rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL linked in `memory/README.md`, so the new position (or updated/closed one) shows up. Drafting or presenting a proposal doesn't change account state, so skip this until step 11 actually executes — refreshing after a proposal that's still awaiting a yes/no would show nothing different anyway.

## Guardrails (from `CLAUDE.md` / `README.md`)

- Max 20% of account value per position (a ceiling — size at your own judgment, not automatically to the max)
- Max 7 concurrent open positions
- No minimum cash reserve — full deployment up to the caps above is allowed
- Stop-loss at -8% from entry
- Max 7 new trade proposals per week
- Agentic account (••••6245) only
- Propose + Confirm: never place an order without the user's explicit "yes" in that conversation

## Journal

Use the `journal-entry` skill in both cases:

- **When a proposal is drafted** (regardless of outcome): Summary, Findings (signal that supports it), Proposals (the full sized proposal), User decisions (approved/rejected/changed — leave this out only if the user hasn't responded yet and will in a later turn), Follow-ups if relevant.
- **When a cap blocks a proposal** (steps 3–4): still journal it — Summary + Findings (what was considered) + Follow-ups (revisit once the cap resets), so the block is visible in the record even though nothing was proposed.

## Commit and push — but don't auto-merge

Commit the journal entry and push it, same as any other change in this repo. Unlike the four read-only Routines or a watchlist-only edit, `CLAUDE.md` guardrails 6 and 7 do **not** cover this skill's output — a drafted or executed trade proposal is exactly the kind of change that still needs the user's explicit go-ahead before it lands on `main`, whether or not the order itself has been confirmed yet. Open a PR if the repo's workflow calls for one, then wait for the user to say to merge it.
