---
name: daily-check
description: Runs the daily portfolio + watchlist pulse check for the Agentic-Trader repo — a same-day, read-only snapshot of the Agentic Robinhood account (••••6245) and every symbol on memory/watchlist.md, flagging anything that moved enough to matter before the next session. Use this whenever the user asks for a "daily check," "portfolio check," "how's my portfolio/account doing," a status update on the watchlist, or when the "Daily Robinhood Portfolio Summary" Routine fires. This is the operationalized version of routines/daily-check.md — always run it end to end (including the journal entry) rather than improvising the steps.
---

# Daily Portfolio + Watchlist Check

Read-only pulse check, meant to run after market close (routine schedule: weekdays 4:30 PM ET). It answers one question: did anything move enough today that it needs a follow-up, without taking any action itself.

Full spec: `routines/daily-check.md`. This skill operationalizes it — if the two ever disagree, treat the playbook doc as the source of truth and flag the mismatch rather than silently picking one.

## Steps

1. **Account snapshot.** Call `get_portfolio` and `get_equity_positions` for the Agentic account (••••6245) only — never any other linked account. Note total account value, today's gain/loss, and each open position.

2. **Open orders.** Call `get_equity_orders` to check for anything still pending.

3. **Watchlist scan.** Read `memory/watchlist.md`. For each symbol listed:
   - `get_equity_quotes` for current price.
   - If the move looks notable (beyond typical daily noise, or near a level the thesis cares about), also pull `get_equity_technical_indicators` for more context.

4. **Flag, don't act.** For anything that stands out — approaching a stop-loss level, thesis looking invalidated, thesis playing out and worth acting on — note it as a follow-up. **Do not propose or place a trade as part of this skill.** If a follow-up looks like it warrants an actual trade proposal, say so explicitly and point to the `propose-trade` skill as the next step (in a later turn, once the user wants to act on it).

## Guardrails

- Strictly read-only: no calls to `place_equity_order`, `place_option_order`, or anything that modifies an order or the watchlist.
- Agentic account (••••6245) only.

## Journal

Always finish by using the `journal-entry` skill to log this run. Include:
- **Summary:** one-line — daily check, date.
- **Findings:** portfolio value, day P&L, open positions, open orders, watchlist symbols that moved and why it matters.
- **Follow-ups:** anything flagged for later (approaching stop, thesis shift, etc.) — omit if genuinely nothing stood out.
- No Proposals or User decisions sections — this routine doesn't produce either.

## Commit, push, and merge

Commit the journal entry, push, and merge it to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrail 7 pre-authorizes this specifically for this Routine's output, since it never places a trade. If a follow-up here leads to an actual trade proposal later, that's a separate step through `propose-trade`, gated the normal way.
