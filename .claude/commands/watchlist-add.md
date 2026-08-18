---
description: Add a symbol (with thesis) to memory/watchlist.md, mirrored to the Robinhood "Agentic Watchlist" for the Agentic account
---

Add a symbol to the Agentic-Trader watchlist.

Arguments: `$ARGUMENTS` — expected as `<SYMBOL> [one-line thesis]`, e.g. `/watchlist-add NVDA breaking out above 200d MA on volume`. The thesis is optional — `/watchlist-add TSLA` on its own is fine.

## Steps

1. **Parse.** First whitespace-separated token is the ticker symbol (uppercase it). Everything after it is the thesis, if any. A thesis is optional — if none was given, don't ask for one or invent a rationale; just proceed with an empty thesis (step 4 covers how that's recorded).

2. **Verify the symbol.** Use the Robinhood MCP tools (`search` and/or `get_equity_tradability`) to confirm it resolves to a real, tradable equity. If it doesn't, tell the user and stop — don't add it.

3. **Check for a duplicate.** Read `memory/watchlist.md`. If the symbol is already on it (case-insensitive), don't add a second row — show the user the existing thesis/status and ask whether they want to update the thesis instead (if so, edit that row in place rather than appending).

4. **Add the row.** Append `| SYMBOL | <thesis> | <today's date, YYYY-MM-DD> | watching |` to the table. If no thesis was given, use `_(no thesis yet)_` as the cell so the table stays well-formed and it's visibly a placeholder to fill in later — never leave the cell blank. If the file still has the placeholder empty-watchlist note (the italic line starting "_Empty — ..."), remove that note now that the table has a real entry — don't leave stale placeholder text sitting under real rows.

5. **Mirror it to Robinhood.** Call `add_to_watchlist` with `list_id` `aaf2fa51-2240-4463-8af2-d8c8b0621302` (the "Agentic Watchlist" custom list under the Agentic account) and `symbols: [SYMBOL]`. This repo's `memory/watchlist.md` and that Robinhood list are supposed to always match (per `CLAUDE.md`) — don't consider this command done until both are updated.

6. **Confirm back to the user** what was added, in both places. If the thesis was left blank, say so and suggest they add one later (e.g. by re-running this command to update the row, once that's supported, or editing `memory/watchlist.md` directly).

7. **Journal it.** Use the `journal-entry` skill to log this: Summary ("added SYMBOL to watchlist"), Proposals (the symbol + thesis, or "no thesis given yet" if blank), User decisions ("approved adding SYMBOL to watchlist"). The user invoking this command with a symbol *is* the approval required by `CLAUDE.md` — no separate confirmation round-trip needed once steps 1–3 pass, since they named the exact symbol themselves.

This command only ever touches the watchlist (`memory/watchlist.md` and its mirrored Robinhood list) — it never places an order or proposes a trade. Adding a symbol here is not a trade proposal; an actual entry still goes through the `propose-trade` skill later.
