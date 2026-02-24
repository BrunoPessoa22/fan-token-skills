---
name: order-router
description: Smart order routing across CEX and DEX venues for fan token execution with best-price quotes.
---

# Order Router

Smart order routing for fan token trades. Compares prices across multiple venues (Chiliz DEX, Binance, Bitget, Gate.io, etc.), calculates slippage, and routes to the best execution. Use this when the user wants to actually trade or check execution quality.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_quote
Get a real-time price quote across all available venues before executing.

**Endpoint:** `GET /api/v1/execute/quote/{symbol}`

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `symbol` | path | yes | Token symbol, e.g. `PSG` (uppercase) |
| `side` | query | yes | `buy` or `sell` |
| `amount_usdt` | query | yes | Trade size in USDT |

**When to use:** When the user asks "what's the best price for PSG", "where should I buy BAR", "get me a quote", or before presenting an execution recommendation.

**Example request:**
```
GET /api/v1/execute/quote/PSG?side=buy&amount_usdt=500
```

**Example response:**
```json
{
  "symbol": "PSG",
  "side": "buy",
  "amount_usdt": 500,
  "best_venue": {
    "venue": "chiliz_dex",
    "price": 3.415,
    "estimated_tokens": 146.41,
    "slippage_pct": 0.12,
    "fee_pct": 0.30,
    "total_cost_usdt": 501.47
  },
  "alternatives": [
    {
      "venue": "binance",
      "price": 3.428,
      "estimated_tokens": 145.85,
      "slippage_pct": 0.05,
      "fee_pct": 0.10,
      "total_cost_usdt": 500.72
    },
    {
      "venue": "bitget",
      "price": 3.432,
      "estimated_tokens": 145.68,
      "slippage_pct": 0.08,
      "fee_pct": 0.20,
      "total_cost_usdt": 501.39
    }
  ],
  "spread_analysis": {
    "best_to_worst_spread_pct": 0.50,
    "arb_opportunity": false
  },
  "quote_valid_seconds": 10,
  "timestamp": "2026-02-17T14:30:00Z"
}
```

**Interpreting results:**
- **best_venue:** The venue with the lowest `total_cost_usdt` (for buys) or highest proceeds (for sells). This accounts for price, slippage, AND fees.
- **slippage_pct:** Expected price impact from the trade size. Above 1% is high -- warn the user. Above 3% suggest splitting the order.
- **total_cost_usdt:** The all-in cost including fees and slippage. Always present this, not just the price.
- **alternatives:** Show the top 2-3 alternatives so the user can see the comparison.
- **arb_opportunity:** If true, there is a cross-venue price discrepancy worth mentioning.
- **quote_valid_seconds:** Quotes expire quickly. If the user deliberates, re-fetch before executing.

### execute_order
Execute a trade through the router.

**Endpoint:** `POST /api/v1/execute`

**Body (JSON):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `symbol` | string | yes | Token symbol (uppercase) |
| `side` | string | yes | `buy` or `sell` |
| `amount_usdt` | float | yes | Trade size in USDT |
| `mode` | string | no | `best_price` (default), `lowest_slippage`, `split` |
| `venue_override` | string | no | Force a specific venue (e.g. `binance`, `chiliz_dex`) |
| `max_slippage_pct` | float | no | Maximum acceptable slippage. Order rejected if exceeded. Default: 2.0 |

**When to use:** When the user explicitly confirms they want to execute a trade. NEVER call this without user confirmation. Always show a quote first.

**CRITICAL: Always get explicit user confirmation before calling this endpoint. Present the quote, ask "do you want to proceed?", then execute only after a clear yes.**

**Example request:**
```json
POST /api/v1/execute
{
  "symbol": "PSG",
  "side": "buy",
  "amount_usdt": 500,
  "mode": "best_price",
  "max_slippage_pct": 1.0
}
```

**Example response:**
```json
{
  "order_id": "ord-x7y8z9",
  "status": "filled",
  "symbol": "PSG",
  "side": "buy",
  "venue": "chiliz_dex",
  "amount_usdt": 500,
  "tokens_received": 146.23,
  "avg_price": 3.419,
  "slippage_pct": 0.14,
  "fee_usdt": 1.50,
  "total_cost_usdt": 501.50,
  "executed_at": "2026-02-17T14:30:05Z"
}
```

**Possible statuses:**
- `filled` -- Order fully executed.
- `partial` -- Only part of the order filled (low liquidity). Present what filled and remaining amount.
- `rejected` -- Slippage exceeded `max_slippage_pct` or venue unavailable. Inform user and suggest retrying with higher tolerance or smaller size.
- `pending` -- Rare, order submitted but not yet confirmed on-chain (DEX). Poll `GET /api/v1/execute/order/{order_id}` for updates.

### list_venues
List all available trading venues and their current status.

**Endpoint:** `GET /api/v1/execute/venues`

**When to use:** When the user asks "where can I trade fan tokens", "which exchanges are supported", or when debugging a failed execution.

**Example response:**
```json
{
  "venues": [
    {
      "id": "chiliz_dex",
      "name": "Chiliz DEX",
      "type": "dex",
      "chain": "chiliz",
      "status": "online",
      "supported_tokens": ["PSG", "BAR", "JUV", "ACM", "ASR", "ATM", "GAL", "INTER", "POR", "NAP", "LAZ", "OG", "SANTOS", "CITY"],
      "fee_pct": 0.30,
      "avg_slippage_pct": 0.15
    },
    {
      "id": "binance",
      "name": "Binance",
      "type": "cex",
      "status": "online",
      "supported_tokens": ["PSG", "BAR", "JUV", "ATM", "ACM", "SANTOS", "CITY", "POR", "LAZ", "CHZ"],
      "fee_pct": 0.10,
      "avg_slippage_pct": 0.05
    },
    {
      "id": "bitget",
      "name": "Bitget",
      "type": "cex",
      "status": "online",
      "supported_tokens": ["PSG", "BAR", "JUV", "CHZ"],
      "fee_pct": 0.20,
      "avg_slippage_pct": 0.08
    }
  ]
}
```

**Interpreting results:**
- **status:** `online` means active, `degraded` means slow but functional, `offline` means unavailable.
- **type:** `dex` venues require on-chain transactions (slower, potentially higher slippage). `cex` is faster but requires exchange accounts.
- **supported_tokens:** Not all tokens are on all venues. Always check this before quoting.
- **fee_pct / avg_slippage_pct:** Useful for quick mental math on expected costs.

## Execution Modes

| Mode | Behavior |
|------|----------|
| `best_price` | Routes to the single venue with the best all-in price (default) |
| `lowest_slippage` | Routes to the venue with the least price impact, even if slightly more expensive |
| `split` | Splits the order across multiple venues to minimize total slippage. Best for large orders (>1000 USDT) |

## Safety Rules

1. **Always quote before executing.** Never skip the quote step.
2. **Always confirm with the user.** Never auto-execute.
3. **Respect max_slippage_pct.** Default is 2%. For small orders (<200 USDT), 1% is usually safe. For large orders (>2000 USDT), suggest `split` mode.
4. **Re-quote if more than 10 seconds have passed** since the last quote. Prices move fast.
