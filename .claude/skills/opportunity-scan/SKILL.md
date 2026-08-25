---
name: opportunity-scan
description: Runs the opportunity scan for the Agentic-Trader repo (every 2 hours, 9:30am-3:30pm ET on weekdays) — checks the current watchlist plus a reactive breakout screen and two leading-indicator screens (unusual options activity, stealth volume accumulation) for assets showing real, current or emerging upside potential, reports directly to the user, and can act on what it finds. Use this when the user asks "what's looking good right now," "any new opportunities," "upside scan," or when the "Opportunity Scan" Routine fires. Unlike the other read-only routines, this one may add or remove watchlist symbols on its own conviction and may trigger propose-trade (after cross-checking earnings-watch and weekly-scan findings), which sizes and places the order directly — no separate user confirmation, per CLAUDE.md guardrail 1. This is the operationalized version of routines/opportunity-scan.md.
---

# Opportunity Scan

A lighter, more frequent pass than `weekly-scan` — every 2 hours during market hours (9:30am, 11:30am, 1:30pm, 3:30pm ET, weekdays), meant to catch something worth acting on before the next `daily-check` or `weekly-scan` would. Unlike the other routines in this repo, it's allowed to act: it can self-manage the watchlist and can initiate a trade proposal, though it never has to.

Full spec: `routines/opportunity-scan.md`. If this skill and that doc ever disagree, treat the playbook as the source of truth and flag it.

## Steps

0. **Confirm the market is actually open.** The Routine's cron only restricts firing to weekdays — it has no concept of US market holidays, and no holiday-calendar tool exists to check in advance. Pull `get_equity_quotes` for one or two watchlist symbols and check `state` and trade-timestamp freshness before doing anything else. If the market isn't actually open (closed today, most likely a holiday), stop here, skip the rest of the steps, and journal a one-line "market closed today, skipping" note instead of running the full scan against dead data.

1. **Build the reactive candidate set.**
   - Every symbol on `memory/watchlist.md`.
   - A fresh scan: `get_scans` for a bullish/momentum-oriented screen, `run_scan` on it (or `create_scan` if none exists — same liquidity bar as `weekly-scan`).
   - **Filter out junk before doing any real work on it.** The scanner has no native "exclude leveraged/basket product" filter, but every leveraged ETF/ETN, inverse product, and basket/trust observed in scan output comes back with an empty `market_cap` field (`""`), while every real operating company has one populated. Drop any hit with an empty `market_cap` from the candidate set — don't spend a technicals/fundamentals call vetting a 2x/3x wrapper riding sector beta. Note the count skipped this way in the journal, don't just silently drop them.
   - **Bring forward catalyst context.** Read the most recent `weekly-scan` journal entry for any catalyst-calendar findings (FDA decisions, index rebalances, M&A chatter, conferences) — if a scan hit or watchlist mover lines up with a catalyst already flagged there, say so; it's a stronger signal than a bare price move with no known reason behind it.
   - **Bring forward the momentum-building watch list.** Same journal entry — check `weekly-scan`'s "building momentum" names (rising RSI, no single day extreme enough to hit this scan's own threshold). If any of them shows up as a scan hit or watchlist mover this run, treat it as a name that was already flagged as accumulating, not a surprise — say so explicitly rather than reporting it as a fresh, unexplained move.

2. **Check for leading signals — before price has confirmed anything.** Everything in step 1 is reactive: it only ever surfaces a name once it's already moved. This step looks for positioning that tends to run *ahead* of a move. Run both saved scans below every cycle and apply the same empty-`market_cap` junk filter as step 1:
   - **"Predictive Scan — Unusual Options Activity"** (`get_scans` by title, `run_scan`) — relative options volume > 4x normal on a $1B+ name that *hasn't* already made a big move (%change < 3%). Pull the `Total call volume` / `Total put volume` columns from the result and note the skew — a lopsided call/put ratio (e.g. 10:1 or higher) on real volume is informed positioning ahead of an anticipated move, not noise.
   - **"Predictive Scan — Stealth Accumulation"** (`get_scans` by title, `run_scan`) — relative volume > 1.5x normal with price still flat (between -1% and +2% on the day) — real buying pressure showing up in volume before it's shown up in price.
   - **Treat a survivor here as "leading, unconfirmed" throughout the rest of this run** — carry that label into steps 3-6. A leading signal alone (no other corroboration) is a **watch-only flag, journaled but not added to the watchlist and not a trade candidate** — same treatment `weekly-scan`'s momentum-building alert gets, and for the same reason: the whole point is it hasn't been proven right yet, so treating it as a full add would just be lowering the conviction bar on a bare mechanical signal. It only clears the watchlist-add bar in step 4 if something else corroborates it — a real catalyst, clean fundamentals, or a trend that's already favorable.

3. **Check for a real signal.** For each surviving candidate — both reactive (step 1) and leading (step 2): `get_equity_technical_indicators`, `get_equity_historicals`, and (for anything not already vetted) `get_equity_fundamentals`. You're looking for something that justifies flagging it *right now* — a breakout, momentum shift, unusual positioning, or a level from an existing thesis being approached — not just "still exists and still looks fine."
   - **Trend context.** Pull SMA 50/200 from `get_equity_technical_indicators` — is the move happening with the trend (price above both, 50 above 200) or against it (a bounce inside a downtrend is a much weaker signal than a breakout in an established uptrend)?
   - **Support/resistance.** Pull the `pivot_points` indicator for candidates that clear the first screen — a move approaching a pivot resistance level is a different, weaker setup than one that's already cleared it with room to the next level; note the nearest level either way.

4. **Filter hard before reporting.** This runs every 2 hours during market hours. Reporting the same unchanged symbols every run trains the user to stop reading it — only surface what's actually new or changed since the signal would have last been true. It's fine, and often correct, for a run to report nothing.

5. **Manage the watchlist directly.** This is the one routine in this repo authorized to do this without a confirmation round-trip (`CLAUDE.md` guardrail 8):
   - **Add** a new candidate that clears the repo's usual conviction bar — real, current signal; no bad news or major concerns (the same bar applied in every watchlist research pass so far) — by invoking the `watchlist-add` skill. A step-2 leading signal only qualifies here if something else corroborates it (see step 2) — a bare options-flow or volume spike with nothing else behind it stays a watch-only journal note, not an add.
   - **Remove** an existing watchlist symbol whose thesis has clearly broken — real bad news landed, the technical setup failed, the original catalyst is gone — by invoking the `watchlist-remove` skill.
   - Always record *why* in the journal entry. Don't add or remove on a marginal or ambiguous read — when in doubt, leave it and just report the observation.

6. **Consider a trade proposal — optional.** For anything, new or already on the watchlist, that looks compelling enough to actually trade:
   - **Cross-check earnings timing first.** Read the most recent `earnings-watch` journal entry; if it's stale (older than about a week) or never covered this symbol, run `get_earnings_calendar` for it directly. Never let a proposal walk into an earnings print unflagged.
   - **Cross-check deeper vetting.** Read the most recent `weekly-scan` journal entry for prior fundamentals/technicals work on this symbol; if none exists, pull `get_equity_fundamentals` directly — the same sanity-check `weekly-scan` itself would do.
   - Only once both checks come back clean, invoke the `propose-trade` skill for that symbol. It runs its own guardrail checks (sizing, position cap, weekly trade cap) and, once they pass, places the order directly — no separate confirmation step.
   - This step is genuinely optional. A run with nothing that clears both cross-checks should say so plainly rather than stretch a marginal setup into a trade. A bare step-2 leading signal (no corroboration) shouldn't reach this step at all — it isn't a trade candidate on its own.

7. **Report directly to the user** — symbol, the specific signal (reactive or leading — say which), current price/level, one line on why it matters now, plus anything added to or removed from the watchlist this run, plus whether a trade was executed (and why, if not).

## Guardrails

- Never calls `place_equity_order`/`place_option_order` directly itself — trade execution flows only through `propose-trade`, which places the order once its own guardrail checks pass. No separate confirmation step, per `CLAUDE.md` guardrail 1.
- Watchlist adds/removes are self-authorized (`CLAUDE.md` guardrail 8) but must be journaled with reasoning and mirrored to the Robinhood "Agentic Watchlist" — same mechanics as `watchlist-add`/`watchlist-remove`.
- Never trigger a trade without first checking `earnings-watch` and `weekly-scan` findings for that symbol (step 6).
- `propose-trade`'s own guardrails (20% max position size, 7 max concurrent positions, no fixed cash reserve, -8% stop-loss, 7 max new trades/week) apply once invoked — this skill doesn't re-implement them, just triggers the check.

## Journal

Always finish with the `journal-entry` skill:
- **Summary:** "Opportunity scan, <morning/midday/afternoon>."
- **Findings:** what was checked (watchlist + reactive scan + the two leading-signal scans) and what stood out, with the specific signal for each (including trend/SMA context and nearest pivot level where checked, and — for a leading signal — the options call/put skew or relative-volume read that triggered it) — or that nothing new stood out. Note how many hits were skipped as junk (empty market_cap) across all three scans, and label any leading-signal finding as such so it's not confused with a confirmed move.
- **Proposals:** any watchlist add/remove made this run (with reasoning), and the trade if one was executed via `propose-trade` (with the earnings-watch/weekly-scan cross-check noted, and the fill) — omit if neither happened.
- **Follow-ups:** anything worth a closer look next run, if relevant.
- If step 0 found the market closed, a one-line entry noting that is enough — skip the rest of the template.

## Commit, push, and merge

Commit the journal entry (and any watchlist add/remove, and any new/updated saved scan reference), push, and merge it to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrails 6 and 7 pre-authorize this for the journal and watchlist output specifically. This also covers a trade that actually executed — `propose-trade` auto-merges its own journal entry in the same turn, per `CLAUDE.md` guardrail 7.

## Refresh the dashboard

After journaling, rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL already linked in `memory/README.md` rather than creating a duplicate. This is read-only — it doesn't touch the repo or Robinhood, so nothing above gates it. If the `trading-dashboard` skill or the Artifact tool isn't available, say so rather than silently skipping it.
