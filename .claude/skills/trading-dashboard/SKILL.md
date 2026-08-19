---
name: trading-dashboard
description: Rebuilds the "Agentic Trading Desk" snapshot dashboard — a published Claude Artifact showing the Agentic account's value/positions, the current watchlist with theses, and today's journal activity as a timeline. Use this when the user asks to see/refresh/update "the dashboard," "the trading desk," "the interface," or wants a visual view of what the trading bot has been doing, instead of reading memory/journal/*.md directly. This is a read-only view — it never places or proposes a trade, and rebuilding it never changes anything in memory/watchlist.md.
---

# Trading Desk Dashboard

A static HTML snapshot, published as a Claude Artifact, that gives a scannable view over data that already lives in the repo — `memory/journal/`, `memory/watchlist.md`, and live Robinhood account state. It does not store anything new; it's a rendering, regenerated on request (or as an added step in a routine, if the user asks for that later).

**Why an Artifact and not a live page:** Artifacts run sandboxed with no access to the Robinhood MCP tools or credentials, so this can't poll the account from the browser. Every refresh is built fresh, with data pulled at build time, then published.

## Steps to rebuild

1. **Pull current state:**
   - `get_portfolio` and `get_equity_positions` for the Agentic account (••••6245).
   - `get_equity_orders` for any open orders.
   - Read `memory/watchlist.md` for the current symbol list + theses.
   - `get_equity_quotes` for every watchlist symbol, for a current price and day's move (state whether the figure is a live intraday move or an end-of-close figure — don't imply a live tick if the market's closed; label it "as of &lt;time&gt;" or "close" accordingly).
   - Read today's `memory/journal/<date>.md` (and recent prior days if the user wants more history) for the activity timeline.

2. **Load `artifact-design` before writing any HTML** — this dashboard is a UI/tool (scanned and operated, not read top-to-bottom), so: surface the summary before the detail, encode state in form as well as number (pills/chips for direction and category), and keep it a polished-but-utilitarian treatment — no oversized hero, no decorative flourishes.

3. **Load `dataviz`** for the stat-tile contract, the validated palette (`references/palette.md`), and the delta/status color rules if any charts or meters are added later. Currently there's no time-series chart (only one data point exists per session — a chart with one point isn't a chart), so the dashboard is stat tiles + tables/cards + a timeline, not a chart page.

4. **Reuse the existing design tokens** rather than re-deriving them each time (keeps every rebuild visually consistent):
   - Neutrals: dataviz's chrome & ink table (page/surface/ink/secondary/muted/gridline/border, light + dark).
   - Accent: blue (`#2a78d6` light / `#3987e5` dark) — the desk's operating color.
   - Delta up/down: dataviz's delta-good green (`#006300`/`#0ca30c`) and categorical red (`#e34948`/`#e66767`).
   - Category tags for timeline entries, in fixed assignment (not re-ordered): Setup → violet, Watchlist → magenta, Routines → aqua, Opportunity Scan → orange, Research → yellow, Daily Check → blue/accent. Pick new hues from the *next unused* categorical slot if a new entry type shows up (e.g. earnings-watch entries, once they exist, → the remaining green/red slots are reserved for delta semantics, so skip those; use whichever named hue hasn't been assigned yet, or fold into an existing close category).
   - Type: IBM Plex Sans (UI/headings/prose) + IBM Plex Mono (tickers, prices, timestamps, deltas — anything tabular). Loaded from Google Fonts (the one external host the Artifact CSP allows).

5. **Structure** (top to bottom): header (desk name, account chip, generated-at timestamp) → account snapshot stat tiles (value, cash, open positions, day P&L, watchlist count) → open positions panel (table when positions exist; a designed empty state when not — don't just leave a blank table) → watchlist as a card grid (symbol, company name, price, day-change pill, full thesis text, added date, status) → activity timeline, newest first, one collapsible `<details>` per journal entry (time, category tag, title, one-line summary always visible; full Summary/Findings/Proposals/User decisions/Follow-ups inside). Keep the most recent entry expanded by default.

6. **Build the HTML in the scratchpad directory**, then publish with the `Artifact` tool. Always pass the **same `url`** as the previous publish (ask the user for it, or use `action: "list"` to find "Agentic Trading Desk" if it's not in context) so this redeploys the existing dashboard instead of creating a duplicate — keep the same favicon (📈) and title across redeploys.

7. **Send the user the file** too (`SendUserFile`, `display: "render"`) so it's visible immediately in addition to the published Artifact link.

## Guardrails

- Read-only. Never calls `place_equity_order`, `place_option_order`, `add_to_watchlist`, or `remove_from_watchlist` — this skill only reads and renders.
- Don't fabricate data. If a figure isn't available (e.g. no historical account-value series exists yet for a real chart), say so or omit it — don't estimate.
- This isn't one of the four read-only Routines `CLAUDE.md` guardrail 7 covers, so committing this skill file itself (or any change to it) follows the repo's normal confirm-before-merge workflow — but *running* the dashboard build itself doesn't touch the repo at all (it only reads), so there's nothing to journal or merge from a rebuild alone.
