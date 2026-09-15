---
name: backtest-engine
description: Replay match-timing strategies against real historical fan-token matchday prices - honest, descriptive results.
---

# Backtest Engine

Replays simple match-timing rules (enter at -24h/-2h/kickoff/fulltime, exit at
fulltime/+1h/+24h or target/stop) against the real historical matchday price
corpus. It is a descriptive research tool: it tells you what WOULD have
happened, and the honest answer is usually that naive matchday strategies lose
money. Present results as history, never as a promise of future returns.

**Base URL (REST, free):** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_presets
Pre-built strategy templates.

**Endpoint:** `GET /api/v1/backtest/presets`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/v1/backtest/presets
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "presets": [
    {
      "id": "buy_2h_sell_fulltime",
      "name": "Buy 2h Pre-Match, Sell at Fulltime",
      "params": { "entry_timing": "-2h", "exit_timing": "fulltime" }
    },
    {
      "id": "fade_post_loss",
      "name": "Fade Post-Loss Dumps",
      "params": { "entry_timing": "fulltime", "exit_timing": "+24h", "result_filter": "loss" }
    }
  ]
}
```

### get_featured
Pre-computed results for the preset strategies across the full corpus.

**Endpoint:** `GET /api/v1/backtest/featured`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/v1/backtest/featured
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "strategies": [
    {
      "id": "buy_2h_sell_fulltime",
      "total_trades": 2361,
      "win_rate": 41.21,
      "avg_return_pct": -1.2364,
      "sharpe_ratio": -2.0608,
      "profit_factor": 0.301,
      "best_trade_pct": 49.19,
      "worst_trade_pct": -93.15
    }
  ]
}
```

**Interpreting results (important):**
- These are real numbers and they are NEGATIVE: the classic "buy before
  kickoff" trade lost an average -1.24% per trade over 2,361 matchdays. Lead
  with that honesty -- it is the whole value of the tool.
- `profit_factor` below 1 means gross losses exceed gross wins.
- Use featured results to debunk folk strategies before a user spends money
  testing them live.

### run_backtest
Run a custom backtest. Synchronous: the response contains the results.

**Endpoint:** `POST /api/v1/backtest` (Content-Type: application/json; no API
key required)

Request body (`BacktestRequest`):

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `entry_timing` | string | required | One of `-24h`, `-2h`, `kickoff`, `fulltime` |
| `exit_timing` | string | required | One of `fulltime`, `+1h`, `+24h`, `target_pct`, `stop_pct` |
| `target_pct` | number | 3.0 | Target profit % (for `target_pct` exit), 0-100 |
| `stop_pct` | number | -2.0 | Stop loss % (for `stop_pct` exit), -50-0 |
| `token_filter` | string[] | null | e.g. `["BAR","PSG"]` |
| `date_range_start` / `date_range_end` | string | null | ISO dates |

**Example (verified live, 2026-07-11):**
```bash
curl -s -X POST https://web-production-ad7c4.up.railway.app/api/v1/backtest \
  -H "Content-Type: application/json" \
  -d '{"entry_timing":"-2h","exit_timing":"fulltime","token_filter":["PSG"]}'
```

**Response (truncated):**
```json
{
  "status": "completed",
  "results": {
    "total_trades": 163,
    "win_count": 59,
    "loss_count": 98,
    "win_rate": 36.2,
    "avg_return_pct": -0.425,
    "max_drawdown_pct": 74.07,
    "sharpe_ratio": -3.6009,
    "profit_factor": 0.4296,
    "equity_curve": [0.471, 0.1845, "..."]
  }
}
```

- The response is synchronous (`status: completed` with inline `results`).
  `GET /api/v1/backtest/{job_id}` exists for job lookup but you will normally
  not need it.

**Interpreting results:**
- Always report `total_trades` (sample size), `win_rate`, `avg_return_pct`, and
  `max_drawdown_pct` together. A win rate without drawdown is marketing, not
  analysis.
- Backtests here ignore fees, spread, and slippage -- real results would be
  worse. Say so.
- Past performance does not predict future results. This engine exists to test
  and usually reject hypotheses cheaply.

## Related surfaces

- **MCP (free):** `tokenintel_match_correlation` and
  `tokenintel_match_impact_history` cover the same corpus per token at
  `https://mcp-production-f681.up.railway.app/mcp`.
- **x402 (paid):** for event-conditioned reaction profiles (goal/red-card level
  rather than match level) see the match-intel skill.
