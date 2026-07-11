---
name: whale-intel
description: CEX and on-chain DEX whale flow description for fan tokens - buy/sell tilt, exchange inflows, large swaps.
---

# Whale Intel

Describes large-holder activity across fan tokens: CEX order-flow tilt
(Binance-primary, some HTX/OKX), on-chain exchange inflows, and large DEX swaps
on FanX (Chiliz Chain). Descriptive history and current tilt -- not signals,
not advice.

**Base URL (REST, free):** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_cex_flow
Aggregate CEX buy/sell flow per token.

**Endpoint:** `GET /api/whales/cex/flow`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `hours` | int | 24 | Look-back window |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/whales/cex/flow?hours=24
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "period_hours": 24,
  "tokens": [
    {
      "symbol": "JUV",
      "buy_volume": 2033217.61,
      "sell_volume": 2129120.0,
      "net_flow": -95902.39,
      "buy_count": 36744,
      "sell_count": 39778,
      "signal": "bearish"
    }
  ]
}
```

**Interpreting results:**
- `net_flow` is buy volume minus sell volume (token units). Negative = net
  selling pressure on CEXs over the window.
- The `signal` label is a simple net-flow classification, not a prediction.
  Present the underlying numbers, not just the label.
- Coverage is Binance-primary. Counts include mid-size flow, not only whales.

### get_cex_trades
Individual large CEX trades above a USD threshold.

**Endpoint:** `GET /api/whales/cex/trades`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 50 | Max trades returned |
| `symbol` | string | null | Filter by token |
| `exchange` | string | null | Filter by exchange |
| `min_value` | float | 50000 | Minimum trade value (USD) |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/whales/cex/trades?limit=10&min_value=5000
```

An empty `trades` array is normal at the default $50k threshold -- fan-token
trades are small (see `get_stats`: average CEX trade ~$57). Lower `min_value`
to see real flow, and say what threshold you used.

### get_exchange_inflows
On-chain deposits into exchange wallets (a leading indicator of intent to sell).

**Endpoint:** `GET /api/exchange-inflows`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token |
| `hours` | int | 24 | Look-back window |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/exchange-inflows
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "summary": [
    {
      "token": "CHZ",
      "inflow_1h": 1051506.49,
      "inflow_4h": 8044899.30,
      "inflow_24h": 14936669.24,
      "inflow_1h_usd": 18564.98,
      "inflow_4h_usd": 142146.68,
      "inflow_24h_usd": 262021.28,
      "unique_depositors_4h": 17,
      "top_exchange_4h": "binance"
    }
  ]
}
```

- Rising inflows with multiple `unique_depositors_4h` describes broad-based
  movement toward exchanges; a single depositor may be treasury ops. Report the
  depositor count alongside the volume.

### get_dex_swaps
Large on-chain swaps on FanX DEX with tx hashes.

**Endpoint:** `GET /api/whales/dex/swaps`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 50 | Max swaps returned |
| `symbol` | string | null | Filter by token |
| `min_value` | float | 50000 | Minimum swap value (USD) |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/whales/dex/swaps?limit=5&min_value=1000
```

Each swap carries `tx_hash`, `block_number`, `pool_address`, `token_in`/`token_out`,
amounts, `value_usd`, and `side` -- verifiable on-chain.

### get_dex_volume
Per-token DEX buy/sell volume aggregates.

**Endpoint:** `GET /api/whales/dex/volume`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `hours` | int | 24 | Look-back window |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/whales/dex/volume?hours=24
```

### get_stats
One-call CEX + DEX whale activity summary.

**Endpoint:** `GET /api/whales/stats`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/whales/stats
```

**Example response (live capture, 2026-07-11):**
```json
{
  "period": "24h",
  "cex": {
    "total_trades": 274268,
    "total_volume": 15549629.48,
    "avg_trade_size": 56.70,
    "largest_trade": 19586.76,
    "exchanges_active": 3,
    "tokens_active": 16
  },
  "dex": {
    "total_swaps": 2182,
    "total_volume": 75524.70,
    "avg_swap_size": 34.61,
    "largest_swap": 18398.18
  },
  "threshold_usd": 50000
}
```

- Use this first to calibrate expectations: average trade sizes are tens of
  dollars, so "whale" thresholds must be set relative to this market, not to
  BTC-scale markets.

## Related surfaces

- **MCP (free):** `tokenintel_whale_flows`, `tokenintel_capital_rotation` at
  `https://mcp-production-f681.up.railway.app/mcp`.
- **x402 (paid):** `whale-flows` ($0.05, all tokens) and `whale-flows-token`
  ($0.05, one token with per-exchange breakdown) at
  `https://x402.brunopessoa.com/catalog`.

## Honest limitations

- CEX coverage is Binance-primary with some HTX/OKX; it is not all 10+ venues.
- There is no per-wallet labeling or "historical accuracy" score on this
  surface. Do not invent wallet reputations.
- Buy/sell classification is taker-side heuristic; treat tilt as descriptive.
