---
name: weekly-scan
description: Runs the weekly screen for new swing-trade candidates in the Agentic-Trader repo — scans for liquid, trending/breakout setups, sanity-checks hits against fundamentals and technicals, cross-checks against the existing watchlist and open positions, and proposes watchlist additions (never trades) for the user to confirm. Use this when the user asks to "scan for new candidates," "find new swing trades," "what's new to watch," or wants to refresh the watchlist, or when a weekly scan Routine fires. This is the operationalized version of routines/weekly-scan.md.
---

# Weekly Scan for New Candidates

Keeps the watchlist from going stale by surfacing new candidates that fit the swing-trade strategy. This skill only ever proposes **watchlist** additions — it never proposes or places a trade. Actually opening a position on a watchlist symbol later is a separate step, handled by the `propose-trade` skill.

Full spec: `routines/weekly-scan.md`. If this skill and that doc ever disagree, treat the playbook as the source of truth and flag it.

## Steps

1. **Find or build a scan.** `get_scans` to see what exists. Run one that fits the strategy (liquid, trending/breakout, reasonable volume) with `run_scan`, or `create_scan` if nothing suitable exists yet.

2. **Sanity-check each hit.** For every candidate the scan returns: `get_equity_fundamentals` and (`get_equity_technical_indicators`, `get_equity_historicals`) to confirm the setup actually looks like a swing-trade candidate, not just a scanner artifact.

3. **Cross-check.**
   - Against `memory/watchlist.md` — skip anything already on it.
   - Against current open positions (`get_equity_positions`) — respect the max-5-concurrent-positions guardrail from `CLAUDE.md` when thinking about whether a candidate is realistic to ever act on soon.

4. **Propose, don't add.** For each surviving candidate, present a one-line thesis (why it fits: setup, catalyst, level). Present this to the user and ask for confirmation on which (if any) to add.

5. **Update the watchlist only after confirmation.** Once the user approves a symbol, add a row to `memory/watchlist.md` (`| Symbol | Thesis | Added | Status |`, status `watching`) in the same turn — per `CLAUDE.md`, the watchlist only changes on explicit user approval.

## Guardrails

- Never calls `place_equity_order` / `place_option_order` — this skill proposes watchlist entries, not trades.
- Max 2 new **trade** proposals/week applies later, when a watchlist symbol becomes an actual trade proposal via `propose-trade` — not to watchlist additions themselves.
- Don't add a symbol already on the watchlist or duplicate an open position's thesis without noting it's already held.

## Journal

Always finish with the `journal-entry` skill:
- **Summary:** scan run + criteria used.
- **Findings:** candidates found, with the fundamentals/technicals that made each pass or fail the sanity check.
- **Proposals:** candidates proposed for the watchlist, with one-line thesis each.
- **User decisions:** which were approved, rejected, or changed.
- **Follow-ups:** anything worth a closer look next time (e.g., a candidate that was close but not quite there).

## Commit, push, and merge

Commit the journal entry (and any approved watchlist additions — already covered by guardrail 6), push, and merge to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrail 7 pre-authorizes this for this Routine's output, since it never places a trade. This doesn't change step 4/5 above: still get the user's confirmation before writing anything to `memory/watchlist.md` in the first place — the auto-merge is about not needing a *second* approval to land the change once it's made, not a license to add symbols unconfirmed.
