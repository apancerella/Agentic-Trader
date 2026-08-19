---
description: Remove a symbol from memory/watchlist.md, mirrored to the Robinhood "Agentic Watchlist" for the Agentic account
---

Remove a symbol from the Agentic-Trader watchlist.

Arguments: `$ARGUMENTS` — a single ticker symbol, e.g. `/watchlist-remove TSLA`.

## Steps

1. **Parse.** Uppercase the symbol. If nothing was given, ask which symbol before doing anything else.

2. **Find it.** Read `memory/watchlist.md`. If the symbol (case-insensitive) isn't on it, tell the user it's not there and stop — nothing to remove, and don't touch the Robinhood side either (removing something not on the repo list risks removing a symbol someone added by hand directly in the Robinhood app, which this command has no visibility into).

3. **Remove the row.** Delete that line from the table in `memory/watchlist.md`. If this was the last row, restore a placeholder note under the header (matching the original empty-watchlist style: `_Empty — no symbols currently on the watchlist._`) so the file doesn't end with a header and nothing else.

4. **Mirror it to Robinhood.** Call `remove_from_watchlist` with `list_id` `aaf2fa51-2240-4463-8af2-d8c8b0621302` (the "Agentic Watchlist" custom list under the Agentic account) and `symbols: [SYMBOL]`. Don't consider this command done until both the repo file and the Robinhood list are updated — same rule as `/watchlist-add`.

5. **Confirm back to the user** what was removed, in both places.

6. **Journal it.** Use the `journal-entry` skill to log this: Summary ("removed SYMBOL from watchlist"), Findings (the thesis/status it had before removal, and why it's coming off if the user said — record "no reason given" rather than inventing one), User decisions ("approved removing SYMBOL from watchlist"). The user invoking this command with a symbol *is* the approval required by `CLAUDE.md` — no separate confirmation round-trip needed.

7. **Commit, push, and merge to `main` in the same turn.** Commit the `memory/watchlist.md` and journal changes, push, and merge to `main` (open a PR first if the repo's workflow requires one) without waiting for the user to separately say "merge it." `CLAUDE.md` guardrail 6 pre-authorizes auto-merging watchlist-only changes specifically, so the repo file and the Robinhood list never sit out of sync waiting on a merge. This authorization is scoped to changes that only touch the watchlist (plus its journal entry) — don't extend it to anything else you happen to be touching in the same session.

This command only ever touches the watchlist (`memory/watchlist.md` and its mirrored Robinhood list) — it never places an order, closes a position, or modifies a trade. Removing a symbol from the watchlist has no bearing on an existing position in that symbol, if one exists — that's a separate decision through `propose-trade`.
