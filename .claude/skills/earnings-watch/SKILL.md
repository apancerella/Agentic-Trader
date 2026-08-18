---
name: earnings-watch
description: Runs the earnings-calendar risk check for the Agentic-Trader repo — checks current holdings and every memory/watchlist.md symbol against the upcoming earnings calendar so an earnings print never blindsides an open or proposed position. Use this when the user asks about "earnings risk," "what's reporting soon," "earnings this week," or wants to sanity-check timing before entering a position, or when the weekly earnings-watch Routine fires. This is the operationalized version of routines/earnings-watch.md.
---

# Earnings Calendar Watch

Earnings volatility is a distinct risk from the technical/fundamental thesis behind a position — a good setup can still get blown up by a surprise print. This skill is purely about flagging that risk on a schedule; it never proposes or places a trade itself.

Full spec: `routines/earnings-watch.md`. If this skill and that doc ever disagree, treat the playbook as the source of truth and flag it.

## Steps

1. **Build the symbol set.** Current holdings (`get_equity_positions`) plus every symbol in `memory/watchlist.md`.

2. **Check the calendar.** `get_earnings_calendar` for the coming week, plus a short lookahead, for each symbol in the set.

3. **Flag anything in the window.** For every symbol with earnings coming up:
   - **Open position** → note the position size and how close earnings is to today.
   - **Watchlist candidate (not yet a position)** → note that any new entry proposal on this symbol needs to account for earnings timing before it goes to `propose-trade`.

## Guardrails

- Read-only: never calls `place_equity_order`, `place_option_order`, or modifies the watchlist. It flags risk for a human — or a later `propose-trade` run — to account for.

## Journal

Always finish with the `journal-entry` skill:
- **Summary:** earnings watch, date range covered.
- **Findings:** symbols with upcoming earnings, whether each is a holding or a watchlist candidate, and the specific risk flagged for each.
- **Follow-ups:** anything that needs a decision before its earnings date (e.g., "consider trimming before print" or "hold off proposing entry until after earnings").
- No Proposals or User decisions sections — this routine doesn't produce either.

## Commit, push, and merge

Commit the journal entry, push, and merge it to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrail 7 pre-authorizes this specifically for this Routine's output, since it never places a trade.
