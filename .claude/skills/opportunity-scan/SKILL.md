---
name: opportunity-scan
description: Runs the twice-daily opportunity scan for the Agentic-Trader repo — checks the current watchlist plus a fresh screen for assets showing real, current upside potential, and reports directly to the user. Use this when the user asks "what's looking good right now," "any new opportunities," "upside scan," or when the 9am/1pm ET "Opportunity Scan" Routine fires. Purely a report — never modifies the watchlist or proposes a trade; that's a separate, deliberate step via /watchlist-add or the propose-trade skill. This is the operationalized version of routines/opportunity-scan.md.
---

# Opportunity Scan

A lighter, more frequent pass than `weekly-scan` — twice a day, meant to catch something worth a look before the next `daily-check` or `weekly-scan` would. Its only job is to tell the user what looks interesting right now; it never acts on it.

Full spec: `routines/opportunity-scan.md`. If this skill and that doc ever disagree, treat the playbook as the source of truth and flag it.

## Steps

1. **Build the candidate set.**
   - Every symbol on `memory/watchlist.md`.
   - A fresh scan: `get_scans` for a bullish/momentum-oriented screen, `run_scan` on it (or `create_scan` if none exists — same liquidity bar as `weekly-scan`).

2. **Check for a real signal.** For each candidate: `get_equity_technical_indicators`, `get_equity_historicals`, and (for anything not already vetted) `get_equity_fundamentals`. You're looking for something that justifies flagging it *right now* — a breakout, momentum shift, or a level from an existing thesis being approached — not just "still exists and still looks fine."

3. **Filter hard before reporting.** This runs twice a day. Reporting the same unchanged symbols every run trains the user to stop reading it — only surface what's actually new or changed since the signal would have last been true. It's fine, and often correct, for a run to report nothing.

4. **Add guardrail context for anything actionable.** If a candidate is already held or on the watchlist, note where it sits against the caps in `CLAUDE.md` (room under the 5-position limit, 10% sizing, etc.) so the user has what they need to act immediately if they want to.

5. **Report directly to the user** — symbol, the specific signal, current price/level, one line on why it matters now. Point to `/watchlist-add` (to track it) or the `propose-trade` skill (to size an actual entry) as the next step if something looks compelling — don't take either action yourself.

## Guardrails

- Read-only: never calls `place_equity_order`/`place_option_order`, and never edits `memory/watchlist.md` or the Robinhood "Agentic Watchlist" — that's out of scope here even though `/watchlist-add` and watchlist-only changes are otherwise pre-authorized to auto-merge (`CLAUDE.md` guardrail 6 covers *committed edits*, not this skill taking the action itself).
- At twice a day, auto-acting on findings would blow past the max-2-new-proposals/week cap — this skill only ever reports.

## Journal

Always finish with the `journal-entry` skill:
- **Summary:** "Opportunity scan, <morning/midday>."
- **Findings:** what was checked (watchlist + scan used) and what stood out, with the specific signal for each — or that nothing new stood out.
- **Follow-ups:** anything worth a closer look next run, if relevant.
- No Proposals or User decisions sections — this routine doesn't produce either.

## Commit, push, and merge

Commit the journal entry (and, if applicable, a new/updated saved scan reference), push, and merge it to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrail 7 pre-authorizes this specifically for this Routine's output, since it never places a trade. This covers the journal/report side only — it does not extend to actually acting on anything the scan found (that's still `/watchlist-add` or `propose-trade`, each gated the normal way).

## Refresh the dashboard

After journaling, rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL already linked in `memory/README.md` rather than creating a duplicate. This is read-only — it doesn't touch the repo or Robinhood, so nothing above gates it. If the `trading-dashboard` skill or the Artifact tool isn't available, say so rather than silently skipping it.
