# Fan Token Skills

Open-source agent skills for fan token intelligence and trading on Chiliz.

**8 skills. 72+ commands. Zero API keys. Zero signup.**

```bash
npx skills add fantokenintel/fan-token-skills
```

---

## Skills

| Skill | Domain | Commands | Data Sources |
|-------|--------|----------|-------------|
| `signal-scores` | Token scoring | 3 | CEX whale data, LunarCrush, match calendar, price feeds, BTC macro |
| `prematch-alpha` | Match alpha | 4 | match_price_correlation (847+ matchdays), whale flows, LunarCrush |
| `backtest-engine` | Backtesting | 3 | Historical price snapshots at -24h/-2h/kickoff/FT/+1h/+24h |
| `order-router` | Execution | 3 | Gate.io, OKX, HTX, MEXC, Bitget, Chiliz DEX + 7 more |
| `whale-intel` | Whale tracking | 6 | 10 CEX exchange APIs, 4h rolling windows |
| `oracle-signals` | Bearish alerts | 5 | Bearish HCA engine, signal accuracy database |
| `sports-data` | Match data | 8 | Football API, 27 clubs across 10 leagues |
| `defi-tools` | DeFi | 7 | Chiliz Chain RPC, Kayen DEX, validator APIs |

---

## What it covers

**27 fan tokens** across 10 football leagues, **13 exchanges** (CEX + DEX), and **847+ matchdays** of historical price-match correlation data.

Your agent gets:
- Unified 0-100 signal scores with 5-component breakdown (whale, social, sports, price, macro)
- Pre-match alpha packets at -24h, -2h, and -15m before kickoff
- Backtesting across real historical matchday data
- Best-price routing across all exchanges
- Real-time whale distribution tracking
- Bearish distribution alerts with verified accuracy
- Match schedules, results, derby detection, importance scoring
- Governance staking, LP provision, and DEX swaps on Chiliz Chain

---

## Quick start

### Install
```bash
npx skills add fantokenintel/fan-token-skills
```

### Ask your agent
```
What are the top tokens to trade right now?
```
```
Any pre-match alpha for upcoming matches?
```
```
Backtest buying 2h before kickoff, sell at fulltime
```
```
Route a $500 BAR buy across all exchanges
```

---

## Skill reference

### signal-scores

Unified 0-100 composite scores per fan token. 5 weighted components: whale flow (25 pts), social momentum (20 pts), sports catalyst (25 pts), price momentum (15 pts), macro regime (15 pts). Pre-computed every 5 minutes.

| Command | Description |
|---------|-------------|
| `get_signal_scores` | All tokens ranked by score with component breakdown |
| `get_score_detail` | Deep-dive on a single token with active signals, matches, whale data |
| `get_score_accuracy` | Historical accuracy stats for a score bucket and direction |

### prematch-alpha

Alpha packets dispatched at -24h, -2h, and -15m before every match. Includes historical stats, live whale signals, galaxy score, suggested entry/exit, and composite confidence.

| Command | Description |
|---------|-------------|
| `get_upcoming_alpha` | All recent alpha packets, filterable by token and hours ahead |
| `get_match_alpha` | All alpha windows for a specific match |
| `get_alpha_history` | Historical alpha dispatches with outcomes |
| `get_alpha_accuracy` | Win rate by window type and confidence level |

### backtest-engine

Simulate any match-based strategy across 847+ real matchdays. Returns win rate, Sharpe ratio, equity curve, max drawdown, profit factor, and per-token breakdown.

| Command | Description |
|---------|-------------|
| `run_backtest` | Submit strategy params (entry/exit timing, filters) and get results |
| `get_presets` | Pre-built strategy templates (pre-kickoff long, fade post-loss, etc.) |
| `get_job_result` | Poll results for async backtest jobs |

### order-router

Best-price execution across 13 exchanges. Quote without executing, simulate with paper trade, or execute live DEX swap. Calculates fees, slippage, and savings vs worst venue.

| Command | Description |
|---------|-------------|
| `get_quote` | Price quote across all venues without execution |
| `execute_trade` | Submit trade (mode: simulate or execute) |
| `get_venues` | List all exchanges with fees and supported tokens |

### whale-intel

Real-time CEX whale distribution tracking. Monitors sell ratios, net inflows, accumulation patterns, and fires distribution alerts when thresholds are crossed.

| Command | Description |
|---------|-------------|
| `get_whale_flows` | Whale activity for a token over a time window |
| `get_distribution_alerts` | Active whale distribution alerts |
| `get_whale_history` | Historical whale flow data |
| `get_sell_ratio` | Current sell ratio for a token |
| `get_net_inflow` | Net exchange inflow/outflow |
| `get_whale_summary` | Aggregated whale activity across all tokens |

### oracle-signals

Proprietary bearish alerts from whale distribution patterns. Outcome-aware cooldown, per-token auto-suppression, trailing stop system. Verified accuracy over 90 days.

| Command | Description |
|---------|-------------|
| `get_active_signals` | Currently active oracle signals |
| `get_signal_history` | Historical signals with outcomes |
| `get_signal_accuracy` | Win rate and average return by signal type |
| `get_trailing_stops` | Active trailing stop positions |
| `get_signal_config` | Current signal thresholds and parameters |

### sports-data

Match schedules, results, price-match correlation, derby detection, importance scoring, and team form analysis. Covers 27 clubs across 10 leagues with 847+ matchdays of data.

| Command | Description |
|---------|-------------|
| `get_fixtures` | Upcoming match schedule |
| `get_match_results` | Recent match results with token impact |
| `get_price_correlation` | Historical price movement around matches |
| `get_derby_schedule` | Upcoming derby matches with rivalry premiums |
| `get_importance_scores` | Match importance ratings |
| `get_team_form` | Recent team performance |
| `get_match_impact` | Average price impact by match type |
| `get_competition_stats` | Stats aggregated by league/competition |

### defi-tools

Governance staking, LP provision, and DEX swaps on Chiliz Chain. Compare validators, scan pools, add liquidity, execute swaps with safety gates.

| Command | Description |
|---------|-------------|
| `get_validators` | List validators with APR, commission, uptime |
| `stake_chz` | Delegate CHZ to a validator |
| `get_pools` | Scan Kayen DEX pools with TVL, volume, APY |
| `add_liquidity` | Provide liquidity to a pool |
| `execute_swap` | Swap tokens on Kayen DEX |
| `get_wallet_balance` | Check wallet balances |
| `get_gas_estimate` | Estimate gas for a transaction |

---

## Compatible agents

Works across 35+ AI coding agents:

- Claude Code
- Cursor
- GitHub Copilot
- Gemini CLI
- Windsurf
- Codex
- Goose
- Kilo Code
- Roo Code
- And 26 more

---

## API access

For production agents that need REST API access with authentication, webhooks, and rate limiting:

1. Register at [fantokenintel.vercel.app/agent-platform](https://fantokenintel.vercel.app/agent-platform)
2. Get your API key (200 req/min on free tier)
3. Base URL: `https://web-production-ad7c4.up.railway.app/api/v1/`

---

## Data sources

| Source | Access | API |
|--------|--------|-----|
| CEX exchanges (13) | Public orderbook/trade data | REST |
| LunarCrush | Social metrics | REST (key required) |
| Football API | Match data | REST (key required) |
| Chiliz Chain | On-chain data | RPC |
| Kayen DEX | DEX data | Smart contract calls |

> All data accessed through the Fan Token Intel backend API. Skills wrap the API calls into agent-friendly instructions.

---

## License

MIT
