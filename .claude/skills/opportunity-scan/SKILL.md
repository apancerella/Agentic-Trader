---
name: opportunity-scan
description: Runs the three-times-daily opportunity scan for the Agentic-Trader repo — checks the current watchlist plus a fresh screen for assets showing real, current upside potential, reports directly to the user, and can act on what it finds. Use this when the user asks "what's looking good right now," "any new opportunities," "upside scan," or when the 10am/12pm/3:30pm ET "Opportunity Scan" Routines fire. Unlike the other read-only routines, this one may add or remove watchlist symbols on its own conviction and may surface a trade proposal via propose-trade (after cross-checking earnings-watch and weekly-scan findings) — it never places an order itself; that always requires the user's explicit confirmation. This is the operationalized version of routines/opportunity-scan.md.
---

# Opportunity Scan

A lighter, more frequent pass than `weekly-scan` — three times a day, meant to catch something worth acting on before the next `daily-check` or `weekly-scan` would. Unlike the other routines in this repo, it's allowed to act: it can self-manage the watchlist and can initiate a trade proposal, though it never has to.

Full spec: `routines/opportunity-scan.md`. If this skill and that doc ever disagree, treat the playbook as the source of truth and flag it.

## Steps

1. **Build the candidate set.**
   - Every symbol on `memory/watchlist.md`.
   - A fresh scan: `get_scans` for a bullish/momentum-oriented screen, `run_scan` on it (or `create_scan` if none exists — same liquidity bar as `weekly-scan`).
   - **Filter out junk before doing any real work on it.** The scanner has no native "exclude leveraged/basket product" filter, but every leveraged ETF/ETN, inverse product, and basket/trust observed in scan output comes back with an empty `market_cap` field (`""`), while every real operating company has one populated. Drop any hit with an empty `market_cap` from the candidate set — don't spend a technicals/fundamentals call vetting a 2x/3x wrapper riding sector beta. Note the count skipped this way in the journal, don't just silently drop them.
   - **Bring forward catalyst context.** Read the most recent `weekly-scan` journal entry for any catalyst-calendar findings (FDA decisions, index rebalances, M&A chatter, conferences) — if a scan hit or watchlist mover lines up with a catalyst already flagged there, say so; it's a stronger signal than a bare price move with no known reason behind it.
   - **Bring forward the momentum-building watch list.** Same journal entry — check `weekly-scan`'s "building momentum" names (rising RSI, no single day extreme enough to hit this scan's own threshold). If any of them shows up as a scan hit or watchlist mover this run, treat it as a name that was already flagged as accumulating, not a surprise — say so explicitly rather than reporting it as a fresh, unexplained move.

2. **Check for a real signal.** For each surviving candidate: `get_equity_technical_indicators`, `get_equity_historicals`, and (for anything not already vetted) `get_equity_fundamentals`. You're looking for something that justifies flagging it *right now* — a breakout, momentum shift, or a level from an existing thesis being approached — not just "still exists and still looks fine."
   - **Trend context.** Pull SMA 50/200 from `get_equity_technical_indicators` — is the move happening with the trend (price above both, 50 above 200) or against it (a bounce inside a downtrend is a much weaker signal than a breakout in an established uptrend)?
   - **Support/resistance.** Pull the `pivot_points` indicator for candidates that clear the first screen — a move approaching a pivot resistance level is a different, weaker setup than one that's already cleared it with room to the next level; note the nearest level either way.

3. **Filter hard before reporting.** This runs three times a day. Reporting the same unchanged symbols every run trains the user to stop reading it — only surface what's actually new or changed since the signal would have last been true. It's fine, and often correct, for a run to report nothing.

4. **Manage the watchlist directly.** This is the one routine in this repo authorized to do this without a confirmation round-trip (`CLAUDE.md` guardrail 8):
   - **Add** a new candidate that clears the repo's usual conviction bar — real, current signal; no bad news or major concerns (the same bar applied in every watchlist research pass so far) — by invoking the `watchlist-add` skill.
   - **Remove** an existing watchlist symbol whose thesis has clearly broken — real bad news landed, the technical setup failed, the original catalyst is gone — by invoking the `watchlist-remove` skill.
   - Always record *why* in the journal entry. Don't add or remove on a marginal or ambiguous read — when in doubt, leave it and just report the observation.

5. **Consider a trade proposal — optional.** For anything, new or already on the watchlist, that looks compelling enough to actually trade:
   - **Cross-check earnings timing first.** Read the most recent `earnings-watch` journal entry; if it's stale (older than about a week) or never covered this symbol, run `get_earnings_calendar` for it directly. Never let a proposal walk into an earnings print unflagged.
   - **Cross-check deeper vetting.** Read the most recent `weekly-scan` journal entry for prior fundamentals/technicals work on this symbol; if none exists, pull `get_equity_fundamentals` directly — the same sanity-check `weekly-scan` itself would do.
   - Only once both checks come back clean, invoke the `propose-trade` skill for that symbol. It runs its own guardrail checks (sizing, position cap, weekly proposal cap) and presents to the user — it still never places an order without their explicit confirmation.
   - This step is genuinely optional. A run with nothing that clears both cross-checks should say so plainly rather than stretch a marginal setup into a proposal.

6. **Report directly to the user** — symbol, the specific signal, current price/level, one line on why it matters now, plus anything added to or removed from the watchlist this run, plus whether a trade proposal is attached (and why, if not).

## Guardrails

- Never calls `place_equity_order`/`place_option_order` itself. `propose-trade` still requires the user's explicit confirmation before any order executes, per `CLAUDE.md` guardrail 1 — surfacing a proposal is not that confirmation.
- Watchlist adds/removes are self-authorized (`CLAUDE.md` guardrail 8) but must be journaled with reasoning and mirrored to the Robinhood "Agentic Watchlist" — same mechanics as `watchlist-add`/`watchlist-remove`.
- Never propose a trade without first checking `earnings-watch` and `weekly-scan` findings for that symbol (step 5).
- `propose-trade`'s own guardrails (20% max position size, 7 max concurrent positions, no fixed cash reserve, -8% stop-loss, 2 max new proposals/week) apply once invoked — this skill doesn't re-implement them, just triggers the check.

## Journal

Always finish with the `journal-entry` skill:
- **Summary:** "Opportunity scan, <morning/midday/afternoon>."
- **Findings:** what was checked (watchlist + scan used) and what stood out, with the specific signal for each (including trend/SMA context and nearest pivot level where checked) — or that nothing new stood out. Note how many scan hits were skipped as junk (empty market_cap).
- **Proposals:** any watchlist add/remove made this run (with reasoning), and the trade proposal if one was drafted (with the earnings-watch/weekly-scan cross-check noted) — omit if neither happened.
- **User decisions:** the user's response to a trade proposal, if resolved in the same turn — omit if there was no proposal, or if it's still awaiting a reply.
- **Follow-ups:** anything worth a closer look next run, if relevant.

## Commit, push, and merge

Commit the journal entry (and any watchlist add/remove, and any new/updated saved scan reference), push, and merge it to `main` in the same turn — don't leave it sitting on a branch waiting for a "please merge." `CLAUDE.md` guardrails 6 and 7 pre-authorize this for the journal and watchlist output specifically. This does **not** cover a trade that actually executed after user confirmation — that journal entry follows `propose-trade`'s own (non-auto-merge) rule, since it documents a real trade, not just a report.

## Refresh the dashboard

After journaling, rebuild and republish the "Agentic Trading Desk" Artifact via the `trading-dashboard` skill, redeploying to the URL already linked in `memory/README.md` rather than creating a duplicate. This is read-only — it doesn't touch the repo or Robinhood, so nothing above gates it. If the `trading-dashboard` skill or the Artifact tool isn't available, say so rather than silently skipping it.
