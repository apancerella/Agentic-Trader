# Agentic-Trader

An agentic swing-trading practice built on Claude Code Routines and the Robinhood MCP server. This repo is the operating home for the strategy: what gets traded, how decisions get made, and a running journal of everything the routines find and do.

## Strategy

**Objective & account.** Equities-only swing / position trading — holding periods of days to weeks, not intraday. All trading activity runs through the **Agentic** Robinhood account (••••6245), the only account enabled for this agent to act on.

**Autonomy model: Propose + Confirm.** Routines research, analyze, and propose trades with a clear rationale — they never place an order on their own. Every trade requires explicit user confirmation in conversation before execution.

**Signal sources.**
| Source | Robinhood MCP tools |
|---|---|
| Technical indicators | `get_equity_technical_indicators`, `get_equity_historicals` |
| Fundamentals & earnings | `get_equity_fundamentals`, `get_earnings_calendar`, `get_earnings_results` |
| Reactive scans / screeners (price has already moved) | `get_scans`, `run_scan`, `create_scan` — "Opportunity Scan — Momentum Breakout" and "Weekly Scan — Early Momentum" |
| Leading-indicator scans (positioning ahead of a move, price hasn't confirmed yet) | Same tools — "Predictive Scan — Unusual Options Activity" (relative options volume > 4x normal, call/put skew, on a name that hasn't moved yet) and "Predictive Scan — Stealth Accumulation" (relative stock volume > 1.5x normal while price is still flat). Run by `opportunity-scan` and `weekly-scan`; a survivor is a watch-only flag until something else corroborates it — see those playbooks. |
| Watchlist | curated in [`memory/watchlist.md`](memory/watchlist.md) |

**Risk guardrails** (every proposal must respect these):
- Max **20%** of account value per position — a ceiling, not a target; `propose-trade` sizes at its own judgment within it
- Max **7** concurrent open positions
- No fixed minimum cash reserve — up to full deployment is allowed within the caps above
- Stop-loss at **-8%** from entry
- No more than **7** new trade proposals per week

**Process flow.**
```
Scan / Watch → Analyze (technicals + fundamentals + earnings) → Propose
   (rationale, entry, size, stop) → Confirm (human) → Execute → Journal
```

This flow is operationalized in [`routines/`](routines/) (the playbooks each recurring Routine follows) and [`memory/`](memory/) (the watchlist and daily journal). See [`CLAUDE.md`](CLAUDE.md) for the standing guardrails every session and scheduled Routine in this repo must follow.
