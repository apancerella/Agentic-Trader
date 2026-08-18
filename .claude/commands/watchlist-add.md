---
description: Add a symbol (with thesis) to memory/watchlist.md for the Agentic-Trader account
---

Add a symbol to the Agentic-Trader watchlist.

Arguments: `$ARGUMENTS` — expected as `<SYMBOL> <one-line thesis>`, e.g. `/watchlist-add NVDA breaking out above 200d MA on volume`.

## Steps

1. **Parse.** First whitespace-separated token is the ticker symbol (uppercase it). Everything after it is the thesis. If no thesis was given, ask the user for one before doing anything else — don't invent a rationale, and don't add a row with a blank thesis.

2. **Verify the symbol.** Use the Robinhood MCP tools (`search` and/or `get_equity_tradability`) to confirm it resolves to a real, tradable equity. If it doesn't, tell the user and stop — don't add it.

3. **Check for a duplicate.** Read `memory/watchlist.md`. If the symbol is already on it (case-insensitive), don't add a second row — show the user the existing thesis/status and ask whether they want to update the thesis instead (if so, edit that row in place rather than appending).

4. **Add the row.** Append `| SYMBOL | <thesis> | <today's date, YYYY-MM-DD> | watching |` to the table. If the file still has the placeholder empty-watchlist note (the italic line starting "_Empty — ..."), remove that note now that the table has a real entry — don't leave stale placeholder text sitting under real rows.

5. **Confirm back to the user** what was added.

6. **Journal it.** Use the `journal-entry` skill to log this: Summary ("added SYMBOL to watchlist"), Proposals (the symbol + thesis), User decisions ("approved adding SYMBOL to watchlist"). The user invoking this command with a symbol *is* the approval required by `CLAUDE.md` — no separate confirmation round-trip needed once steps 1–3 pass, since they named the exact symbol and thesis themselves.

This command only ever touches `memory/watchlist.md` — it never places an order or proposes a trade. Adding a symbol here is not a trade proposal; an actual entry still goes through the `propose-trade` skill later.
