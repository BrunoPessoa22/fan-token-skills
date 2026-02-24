---
name: whale-intel
description: Real-time whale wallet tracking with large flow detection and token distribution analysis for fan tokens.
---

# Whale Intel

Tracks large wallet activity across fan tokens. Detects whale accumulation, distribution, and unusual flow patterns. Provides token-level holder distribution analysis. Use this when the user wants to understand what "smart money" is doing.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_whale_flows
Fetch recent whale transaction flows.

**Endpoint:** `GET /api/v1/whales/flows`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol. Omit for all tokens. |
| `min_usd` | float | 5000 | Minimum transaction value in USD |
| `hours` | int | 24 | Look-back window in hours. Max 168 (7 days). |
| `direction` | string | null | Filter: `inflow` (buying/accumulating), `outflow` (selling/distributing), or null for both |
| `limit` | int | 50 | Max results to return |

**When to use:** When the user asks "what are whales doing", "any big buys", "whale activity on PSG", "large transactions", or "smart money flows".

**Example request:**
```
GET /api/v1/whales/flows?token=PSG&min_usd=10000&hours=12
```

**Example response:**
```json
{
  "flows": [
    {
      "id": "wf-001",
      "token": "PSG",
      "wallet": "0x1a2b...3c4d",
      "wallet_label": "known_accumulator_47",
      "direction": "inflow",
      "amount_tokens": 45200,
      "amount_usd": 154534,
      "price_at_tx": 3.418,
      "source": "binance_withdrawal",
      "destination": "chiliz_wallet",
      "timestamp": "2026-02-17T12:15:00Z",
      "whale_tier": "mega",
      "historical_accuracy": 0.72
    },
    {
      "id": "wf-002",
      "token": "PSG",
      "wallet": "0x5e6f...7g8h",
      "wallet_label": null,
      "direction": "inflow",
      "amount_tokens": 18000,
      "amount_usd": 61524,
      "price_at_tx": 3.418,
      "source": "chiliz_dex_swap",
      "destination": "chiliz_wallet",
      "timestamp": "2026-02-17T11:42:00Z",
      "whale_tier": "large",
      "historical_accuracy": null
    }
  ],
  "summary": {
    "net_flow_usd": 216058,
    "net_direction": "inflow",
    "unique_whales": 3,
    "total_transactions": 5,
    "inflow_usd": 256058,
    "outflow_usd": 40000
  },
  "count": 2,
  "filters_applied": {
    "token": "PSG",
    "min_usd": 10000,
    "hours": 12
  }
}
```

**Interpreting results:**
- **whale_tier:** `mega` (>100k USD), `large` (50-100k USD), `medium` (10-50k USD). Focus on mega and large for signal quality.
- **wallet_label:** If present, this wallet has been identified and tracked. `known_accumulator_*` wallets have historically preceded price moves.
- **historical_accuracy:** For labeled wallets, this is their track record (0-1). A wallet with 0.72 accuracy has been "right" 72% of the time historically. Only available for tracked wallets.
- **source/destination:** `binance_withdrawal` to `chiliz_wallet` = moving off exchange to hold (bullish signal). `chiliz_wallet` to `binance_deposit` = moving to exchange to sell (bearish signal).
- **summary.net_direction:** The aggregate direction. Strong `inflow` across multiple whales is a high-conviction bullish signal.
- When presenting: "3 whale wallets moved a net $216k of PSG on-chain in the last 12 hours (net inflow). The largest was a known accumulator wallet with 72% historical accuracy."

### get_distribution
Fetch holder distribution analysis for a token.

**Endpoint:** `GET /api/v1/whales/distribution`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | required | Token symbol (uppercase) |
| `include_history` | bool | false | Include 30-day distribution change history |

**When to use:** When the user asks "who holds PSG", "token distribution", "concentration risk", "how distributed is BAR", or when assessing a token's health.

**Example request:**
```
GET /api/v1/whales/distribution?token=PSG&include_history=true
```

**Example response:**
```json
{
  "token": "PSG",
  "total_holders": 48250,
  "circulating_supply": 20000000,
  "distribution": {
    "top_10_pct": 42.5,
    "top_50_pct": 61.3,
    "top_100_pct": 72.8,
    "retail_pct": 27.2
  },
  "top_holders": [
    {
      "rank": 1,
      "wallet": "0xabc...def",
      "label": "Socios.com Treasury",
      "balance_tokens": 3200000,
      "balance_pct": 16.0,
      "change_30d_pct": 0.0
    },
    {
      "rank": 2,
      "wallet": "0x123...456",
      "label": "Binance Hot Wallet",
      "balance_tokens": 1800000,
      "balance_pct": 9.0,
      "change_30d_pct": -2.1
    },
    {
      "rank": 3,
      "wallet": "0x789...abc",
      "label": null,
      "balance_tokens": 950000,
      "balance_pct": 4.75,
      "change_30d_pct": 8.5
    }
  ],
  "history": [
    {
      "date": "2026-01-18",
      "total_holders": 46100,
      "top_10_pct": 44.2
    },
    {
      "date": "2026-02-17",
      "total_holders": 48250,
      "top_10_pct": 42.5
    }
  ],
  "health_score": 68,
  "concentration_risk": "moderate"
}
```

**Interpreting results:**
- **top_10_pct:** If above 50%, the token is highly concentrated -- whale moves will have outsized price impact. Warn the user.
- **top_10_pct decreasing over time:** Positive sign, distribution is improving.
- **retail_pct:** Higher is better for organic price discovery. Below 20% means whales dominate the market.
- **change_30d_pct per holder:** If unlabeled top holders are increasing their position, that is a bullish whale signal.
- **health_score:** 0-100 composite of distribution quality. Above 70 is healthy, below 40 is concerning.
- **concentration_risk:** `low`, `moderate`, `high`, `extreme`. Always mention this when discussing a token's fundamentals.
- **Binance Hot Wallet decreasing:** Could mean users are withdrawing to hold (bullish) or reduced trading interest (neutral). Cross-reference with flow data.

## Whale Signal Patterns

When interpreting whale data, look for these high-value patterns:

| Pattern | What It Means | Confidence |
|---------|--------------|------------|
| Multiple whales accumulating same token | Coordinated buying, often pre-catalyst | High |
| CEX to wallet transfers clustered | Moving off exchange to hold | Medium-High |
| Known accumulator wallet active | Historically profitable whale is positioning | High (check `historical_accuracy`) |
| Large outflows to exchange | Potential sell pressure incoming | Medium |
| Distribution improving (top_10 decreasing) | Healthier token, less manipulation risk | Medium |
| Single mega whale buying | Could be meaningful or could be treasury movement -- check label | Low-Medium |

Always cross-reference whale signals with the signal-scores skill for a complete picture.
