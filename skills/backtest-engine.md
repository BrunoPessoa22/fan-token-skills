---
name: backtest-engine
description: Backtest trading strategies against historical fan token data with configurable entry/exit timing and filters.
---

# Backtest Engine

Run historical backtests on fan token trading strategies. Supports custom entry/exit timing, token filtering, strategy presets, and detailed performance analytics. Use this when the user wants to validate a strategy idea or compare approaches.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### run_backtest
Submit a new backtest job. Backtests run asynchronously -- submit, then poll for results.

**Endpoint:** `POST /api/v1/backtest`

**Body (JSON):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entry_timing` | string | yes | When to enter. Options: `pre_match_4h`, `pre_match_2h`, `pre_match_1h`, `kickoff`, `halftime`, `fulltime`, `signal_fired` |
| `exit_timing` | string | yes | When to exit. Options: `pre_match_1h`, `kickoff`, `halftime`, `fulltime`, `post_match_1h`, `post_match_4h`, `post_match_24h`, `trailing_stop` |
| `token_filter` | array | no | List of token symbols to include. Omit for all tokens. |
| `date_range` | object | no | `{ "from": "2025-01-01", "to": "2026-02-17" }`. Default is last 12 months. |
| `min_importance` | int | no | Minimum match importance (0-100). Default 0. |
| `competition_filter` | array | no | Filter by competition: `champions_league`, `europa_league`, `la_liga`, `premier_league`, `serie_a`, `ligue_1`, etc. |
| `stop_loss_pct` | float | no | Stop loss percentage (e.g. 5.0 for -5%). Default: none. |
| `take_profit_pct` | float | no | Take profit percentage (e.g. 10.0 for +10%). Default: none. |
| `position_size_usdt` | float | no | Simulated position size. Default: 100 USDT. |

**When to use:** When the user says "backtest this strategy", "how would X have performed", "test buying before matches", or anything about historical performance of a trading approach.

**Example request:**
```json
POST /api/v1/backtest
{
  "entry_timing": "pre_match_4h",
  "exit_timing": "kickoff",
  "token_filter": ["PSG", "BAR", "JUV"],
  "min_importance": 60,
  "stop_loss_pct": 3.0,
  "position_size_usdt": 100
}
```

**Example response (job submitted):**
```json
{
  "job_id": "bt-a1b2c3d4",
  "status": "queued",
  "estimated_seconds": 15,
  "params": {
    "entry_timing": "pre_match_4h",
    "exit_timing": "kickoff",
    "token_filter": ["PSG", "BAR", "JUV"],
    "min_importance": 60,
    "stop_loss_pct": 3.0,
    "position_size_usdt": 100
  }
}
```

After receiving the job_id, poll `GET /api/v1/backtest/{job_id}` until status is `completed`.

### get_backtest_results
Fetch results for a submitted backtest job.

**Endpoint:** `GET /api/v1/backtest/{job_id}`

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `job_id` | path | The job ID returned from POST /api/v1/backtest |

**When to use:** After submitting a backtest, poll this every few seconds until `status` is `completed`.

**Example response (completed):**
```json
{
  "job_id": "bt-a1b2c3d4",
  "status": "completed",
  "results": {
    "total_trades": 87,
    "winning_trades": 58,
    "losing_trades": 29,
    "win_rate": 66.7,
    "avg_return_pct": 2.8,
    "total_return_pct": 243.6,
    "max_drawdown_pct": -8.2,
    "sharpe_ratio": 1.45,
    "profit_factor": 2.1,
    "avg_hold_duration": "3h 42m",
    "best_trade": { "symbol": "PSG", "return_pct": 12.3, "match": "PSG vs Real Madrid UCL" },
    "worst_trade": { "symbol": "JUV", "return_pct": -3.0, "match": "JUV vs Napoli Serie A" },
    "by_token": [
      { "symbol": "PSG", "trades": 31, "win_rate": 71.0, "avg_return": 3.5 },
      { "symbol": "BAR", "trades": 29, "win_rate": 65.5, "avg_return": 2.6 },
      { "symbol": "JUV", "trades": 27, "win_rate": 63.0, "avg_return": 2.2 }
    ],
    "by_month": [
      { "month": "2025-09", "trades": 12, "return_pct": 18.5 },
      { "month": "2025-10", "trades": 14, "return_pct": 22.1 }
    ]
  }
}
```

**Interpreting results:**
- **win_rate above 60%:** Historically solid for fan token strategies. Present positively.
- **win_rate 50-60%:** Marginal. Note that slippage and fees could erode this.
- **win_rate below 50%:** The strategy underperforms. Suggest adjustments.
- **sharpe_ratio above 1.0:** Good risk-adjusted return. Above 2.0 is excellent.
- **profit_factor above 1.5:** More profit than loss. Below 1.0 means net negative.
- **max_drawdown_pct:** Always mention this. Users need to know worst-case.
- **by_token:** Highlight which tokens the strategy works best/worst for. Some tokens have stronger match-day patterns.
- **by_month:** Show seasonality. European football season (Aug-May) typically has more signal than off-season.

### list_presets
List available pre-built strategy presets.

**Endpoint:** `GET /api/v1/backtest/presets`

**When to use:** When the user wants to see what strategies are available out of the box, or says "what strategies can I test" or "show me presets".

**Example response:**
```json
{
  "presets": [
    {
      "id": "pre_match_pump",
      "name": "Pre-Match Pump",
      "description": "Buy 4h before kickoff, sell at kickoff. Classic match-day momentum.",
      "entry_timing": "pre_match_4h",
      "exit_timing": "kickoff",
      "min_importance": 50
    },
    {
      "id": "ucl_alpha",
      "name": "Champions League Alpha",
      "description": "Buy 2h pre-kickoff for Champions League matches only. High importance filter.",
      "entry_timing": "pre_match_2h",
      "exit_timing": "post_match_1h",
      "competition_filter": ["champions_league"],
      "min_importance": 70
    },
    {
      "id": "win_hold",
      "name": "Win & Hold",
      "description": "Enter at kickoff, hold until 24h post-match. Captures post-win momentum.",
      "entry_timing": "kickoff",
      "exit_timing": "post_match_24h",
      "min_importance": 40
    },
    {
      "id": "trailing_stop_v1",
      "name": "Trailing Stop Momentum",
      "description": "Enter on signal fire with 3% trailing stop. Rides momentum without fixed exit.",
      "entry_timing": "signal_fired",
      "exit_timing": "trailing_stop",
      "stop_loss_pct": 3.0
    }
  ]
}
```

**Interpreting presets:**
- Presets are starting points. Users can modify any parameter before running.
- When presenting presets, give a one-line summary and the key idea behind each.
- If a user describes a strategy in plain English, try to map it to a preset first, then customize if needed.

## Workflow

1. User describes a strategy or asks to backtest something.
2. Map their description to `entry_timing`, `exit_timing`, and filters.
3. Check presets with `GET /api/v1/backtest/presets` if unsure.
4. Submit with `POST /api/v1/backtest`.
5. Poll `GET /api/v1/backtest/{job_id}` until complete (typically 5-20 seconds).
6. Present results with focus on win rate, average return, max drawdown, and per-token breakdown.
7. Suggest modifications if performance is weak (e.g., "try tighter stop loss" or "filter to Champions League only").
