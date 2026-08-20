---
name: weekly-scan
description: Runs the weekly screen for new swing-trade candidates in the Agentic-Trader repo — scans for liquid, trending/breakout setups, sanity-checks hits against fundamentals and technicals, cross-checks against the existing watchlist and open positions, and proposes watchlist additions (never trades) for the user to confirm. Use this when the user asks to "scan for new candidates," "find new swing trades," "what's new to watch," or wants to refresh the watchlist, or when a weekly scan Routine fires. This is the operationalized version of routines/weekly-scan.md.
---

# Weekly Scan for New Candidates

Keeps the watchlist from going stale by surfacing new candidates that fit the swing-trade strategy. This skill only ever proposes **watchlist** additions — it never proposes or places a trade. Actually opening a position on a watchlist symbol later is a separate step, handled by the `propose-trade` skill.

Full spec: `routines/weekly-scan.md`. If this skill and that doc ever disagree, treat the playbook as the source of truth and flag it.

## Steps

1. **Find or build a scan.** `get_scans` to see what exists. Run one that fits the strategy (liquid, trending/breakout, reasonable volume) with `run_scan`, or `create_scan` if nothing suitable exists yet.
   - **Junk filter.** Drop any hit with an empty `market_cap` field before doing any further work on it — that's the mechanical signature of leveraged/inverse ETFs, ETNs, and basket/trust products (the scanner has no native filter for this; every real operating company has a populated market cap, every leveraged/basket product observed so far doesn't). Note the count skipped.

2. **Proactive catalyst research — independent of the scan.** The reactive momentum scan only ever surfaces names that have *already* moved. Before or alongside it, look forward: check for upcoming catalysts on the current watchlist symbols and the broader market/sector — FDA decision dates, index rebalance windows, M&A chatter, major industry conferences, macro data releases relevant to the watchlist's sectors. Use `get_earnings_calendar` plus `search`/`get_equity_fundamentals` for anything watchlist-adjacent, and note any dated catalyst worth watching even if nothing has moved yet — this is what `opportunity-scan` reads back later to connect a future price move to a known reason.

3. **Momentum-building alert.** Even the reactive scan in step 1 has a blind spot: it only fires once a name is already up 3%+ in a single day, so a name quietly accumulating strength over a week or two — steady RSI climb, rising volume, no single day dramatic enough to trip that filter — sails through undetected until the move is already extended (this is precisely the pattern WPM showed in early August before it ever reached this repo's watchlist). Catch that earlier:
   - `get_scans` for the saved **"Weekly Scan — Early Momentum"** scan (looser than the breakout scan: same $10 / 500k-average-volume liquidity floor, but %-change > 1.5% and relative volume > 1.15 instead of 3% / 1.3x) and `run_scan` it. Apply the same empty-`market_cap` junk filter as step 1 — this looser bar is noisy by design (a 1.5% day catches leveraged/inverse ETFs and crypto wrappers almost every session; expect the large majority of hits to be junk) and the filter is what makes it usable.
   - For every survivor, **and every current `memory/watchlist.md` symbol regardless of today's move**, pull RSI over the trailing 2–3 weeks via `get_equity_technical_indicators` (daily interval). Flag a name as **building momentum** only when all three hold: RSI has been climbing across at least 3 consecutive sessions (a trend, not a single spike), RSI now sits in the 55–65 band (past neutral, short of the ≥65 line this repo treats as extended/overbought), and the cumulative price return over the trailing 5–10 sessions is meaningfully positive (roughly ≥5%) despite no single day having cleared the 3% breakout-scan bar.
   - Cross-check `get_earnings_calendar` for each flagged name — a rising-RSI trend running into a print within the next 7 days is a different, riskier read (anticipation risk) than one with a clear runway; call it out explicitly rather than folding it into the same flag.
   - This step never adds to the watchlist or triggers a proposal on its own — it's a "watch closer" observation. Journal it even when nothing else about the name is actionable, so `opportunity-scan` (step 1's catalyst-context read already covers this file) can recognize a later breakout as something already on watch rather than something out of nowhere.

5. **Sector/theme relative-strength check.** Pull `get_equity_historicals` (or quotes) for a fixed basket of major sector ETFs — XLK (tech), XLF (financials), XLE (energy), XLV (health care), XLY (consumer discretionary), XLP (consumer staples), XLI (industrials), XLB (materials), XLU (utilities), XLRE (real estate), XLC (communication services) — over the last 1-week and 1-month windows, rank them by relative performance, and journal the ranking. The point is to catch a sector rotation proactively (e.g. money rotating into energy/materials, out of tech) rather than only noticing it after several watchlist names in the same sector have already moved.

6. **Sanity-check each surviving scan hit.** For every candidate the scan returns (post junk-filter): `get_equity_fundamentals` and (`get_equity_technical_indicators`, `get_equity_historicals`) to confirm the setup actually looks like a swing-trade candidate, not just a scanner artifact. Include trend context (SMA 50/200 — is this with or against the prevailing trend) and the nearest support/resistance level from the `pivot_points` indicator.

7. **Cross-check.**
   - Against `memory/watchlist.md` — skip anything already on it.
   - Against current open positions (`get_equity_positions`) — respect the max-7-concurrent-positions guardrail from `CLAUDE.md` when thinking about whether a candidate is realistic to ever act on soon.

8. **Propose, don't add.** For each surviving candidate, present a one-line thesis (why it fits: setup, catalyst, level, trend). Present this to the user and ask for confirmation on which (if any) to add.

9. **Update the watchlist only after confirmation.** Once the user approves a symbol, add a row to `memory/watchlist.md` (`| Symbol | Thesis | Added | Price at Add | Status |`, pulling the current price via `get_equity_quotes`, status `watching`) in the same turn — per `CLAUDE.md`, the watchlist only changes on explicit user approval.

## Guardrails

- Never calls `place_equity_order` / `place_option_order` — this skill proposes watchlist entries, not trades.
- Max 2 new **trade** proposals/week applies later, when a watchlist symbol becomes an actual trade proposal via `propose-trade` — not to watchlist additions themselves.
- Don't add a symbol already on the watchlist or duplicate an open position's thesis without noting it's already held.

## Journal

Always finish with the `journal-entry` skill:
- **Summary:** scan run + criteria used.
- **Findings:** candidates found, with the fundamentals/technicals (including trend/SMA and nearest pivot level) that made each pass or fail the sanity check; how many hits were skipped as junk; upcoming catalysts found for watchlist/sector symbols even if nothing was actionable yet; names flagged as building momentum (RSI trend, cumulative move, earnings-proximity read); the sector relative-strength ranking.
- **Proposals:** candidates proposed for the watchlist, with one-line thesis each.
- **User decisions:** which were approved, rejected, or changed.
- **Follow-ups:** anything worth a closer look next time (e.g., a candidate that was close but not quite there).

## Commit, push, and merge

Commit the journal entry (and any approved watchlist additions — already covered by guardrail 6), push, and merge to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrail 7 pre-authorizes this for this Routine's output, since it never places a trade. This doesn't change steps 8/9 above: still get the user's confirmation before writing anything to `memory/watchlist.md` in the first place — the auto-merge is about not needing a *second* approval to land the change once it's made, not a license to add symbols unconfirmed.

## Refresh the dashboard

After journaling (and merging any watchlist changes), rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL already linked in `memory/README.md` rather than creating a duplicate. This is read-only — it doesn't touch the repo or Robinhood, so nothing above gates it. If the `trading-dashboard` skill or the Artifact tool isn't available, say so rather than silently skipping it.
