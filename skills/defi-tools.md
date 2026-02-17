---
name: defi-tools
description: DeFi tools for fan token staking, liquidity pools, governance, and DEX analytics on Chiliz Chain.
---

# DeFi Tools

Access DeFi infrastructure on Chiliz Chain: staking vaults, liquidity pools, governance proposals, and DEX analytics. Use this when the user wants to earn yield on fan tokens, provide liquidity, participate in governance, or analyze on-chain DeFi activity.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_staking_vaults
Fetch available staking vaults and current APYs.

**Endpoint:** `GET /api/v1/defi/staking`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol. Omit for all vaults. |
| `min_apy` | float | 0 | Minimum APY percentage |
| `sort_by` | string | `apy` | Sort: `apy`, `tvl`, `token` |

**When to use:** When the user asks "where can I stake PSG", "best yields", "staking options", "APY for fan tokens", or anything about earning passive income on fan tokens.

**Example request:**
```
GET /api/v1/defi/staking?sort_by=apy
```

**Example response:**
```json
{
  "vaults": [
    {
      "vault_id": "v-psg-single",
      "token": "PSG",
      "type": "single_stake",
      "apy_pct": 8.2,
      "tvl_usd": 1250000,
      "lock_period_days": 0,
      "min_stake": 10,
      "rewards_token": "CHZ",
      "platform": "Socios Staking",
      "risk_level": "low",
      "details": "Single-sided PSG staking. Rewards in CHZ. No lock, withdraw anytime."
    },
    {
      "vault_id": "v-psg-chz-lp",
      "token": "PSG",
      "type": "lp_farm",
      "pair": "PSG/CHZ",
      "apy_pct": 22.5,
      "tvl_usd": 890000,
      "lock_period_days": 14,
      "min_stake": 50,
      "rewards_token": "CHZ",
      "platform": "Chiliz DEX",
      "risk_level": "medium",
      "details": "LP farm for PSG/CHZ pair. 14-day lock. Higher yield but exposed to impermanent loss."
    }
  ],
  "count": 12
}
```

**Interpreting results:**
- **type:** `single_stake` is simpler and lower risk (no impermanent loss). `lp_farm` provides liquidity and has impermanent loss risk.
- **apy_pct:** Annualized yield. For LP farms, this includes both trading fees and reward emissions.
- **lock_period_days:** 0 means flexible withdrawal. Always mention lock periods clearly.
- **risk_level:** `low` (single-stake, no lock), `medium` (LP or short lock), `high` (leveraged or long lock).
- **tvl_usd:** Higher TVL generally means more stability but lower APY. Low TVL vaults may have inflated APY that is unsustainable.
- Always warn about impermanent loss for LP farms. Explain it simply: "If PSG price moves significantly relative to CHZ, you may end up with less value than just holding both tokens."

### get_liquidity_pools
Fetch DEX liquidity pool data.

**Endpoint:** `GET /api/v1/defi/pools`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol |
| `sort_by` | string | `volume_24h` | Sort: `volume_24h`, `tvl`, `fee_apy`, `token` |
| `min_tvl` | float | 0 | Minimum TVL in USD |

**When to use:** When the user asks "liquidity on PSG", "DEX pools", "trading volume on chain", or when assessing on-chain depth for a token.

**Example request:**
```
GET /api/v1/defi/pools?token=PSG
```

**Example response:**
```json
{
  "pools": [
    {
      "pool_id": "pool-psg-chz",
      "pair": "PSG/CHZ",
      "dex": "Chiliz DEX",
      "tvl_usd": 2150000,
      "volume_24h_usd": 485000,
      "volume_7d_usd": 2890000,
      "fee_tier_pct": 0.30,
      "fee_apy_pct": 12.4,
      "price_psg_chz": 28.5,
      "price_psg_usd": 3.42,
      "liquidity_depth": {
        "bid_2pct_usd": 42000,
        "ask_2pct_usd": 38000
      },
      "volume_change_24h_pct": 35.2
    },
    {
      "pool_id": "pool-psg-usdt",
      "pair": "PSG/USDT",
      "dex": "Chiliz DEX",
      "tvl_usd": 980000,
      "volume_24h_usd": 210000,
      "volume_7d_usd": 1250000,
      "fee_tier_pct": 0.30,
      "fee_apy_pct": 7.8,
      "price_psg_usd": 3.42,
      "liquidity_depth": {
        "bid_2pct_usd": 19000,
        "ask_2pct_usd": 17500
      },
      "volume_change_24h_pct": 12.8
    }
  ],
  "count": 2
}
```

**Interpreting results:**
- **liquidity_depth:** Shows how much USD can be traded within 2% of mid-price. This is critical for execution quality. If `bid_2pct_usd` is low, large sells will cause significant slippage.
- **fee_apy_pct:** The annualized return from trading fees alone (no farming rewards). This is more sustainable than farming APY.
- **volume_change_24h_pct:** Spikes in volume often correlate with upcoming catalysts (matches, announcements). Worth mentioning.
- **PSG/CHZ vs PSG/USDT:** CHZ pair typically has deeper liquidity. USDT pair is simpler for users who think in dollar terms.

### get_governance
Fetch active and recent governance proposals.

**Endpoint:** `GET /api/v1/defi/governance`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token |
| `status` | string | null | `active`, `passed`, `rejected`, `pending` |
| `limit` | int | 20 | Max results |

**When to use:** When the user asks "any governance votes", "what can I vote on with PSG", "fan token governance", "proposals", or anything about token holder rights and participation.

**Example request:**
```
GET /api/v1/defi/governance?status=active
```

**Example response:**
```json
{
  "proposals": [
    {
      "proposal_id": "prop-psg-2026-04",
      "token": "PSG",
      "title": "Choose PSG 2026-27 Third Kit Color Scheme",
      "description": "PSG token holders vote on the primary color palette for next season's third kit. Options: A) Black & Gold, B) Navy & White, C) All Black.",
      "status": "active",
      "voting_start": "2026-02-15T00:00:00Z",
      "voting_end": "2026-02-22T00:00:00Z",
      "min_tokens_to_vote": 1,
      "options": [
        { "id": "A", "label": "Black & Gold", "votes": 12450, "pct": 45.2 },
        { "id": "B", "label": "Navy & White", "votes": 9800, "pct": 35.6 },
        { "id": "C", "label": "All Black", "votes": 5300, "pct": 19.2 }
      ],
      "total_votes": 27550,
      "participation_pct": 8.2,
      "platform": "Socios.com"
    }
  ],
  "count": 3
}
```

**Interpreting results:**
- **min_tokens_to_vote:** Usually 1 token. Confirm the user holds enough.
- **participation_pct:** Fan token governance typically has 5-15% participation. Above 15% is high engagement.
- **voting_end:** Make sure to note when voting closes so the user doesn't miss it.
- Governance is a key utility of fan tokens. Even if the proposals seem light (kit colors, walkout songs), they drive holder engagement and are part of the value proposition.

### get_dex_analytics
Fetch aggregate DEX analytics for Chiliz Chain.

**Endpoint:** `GET /api/v1/defi/analytics`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `period` | string | `24h` | Time period: `1h`, `24h`, `7d`, `30d` |

**When to use:** When the user asks "how's the Chiliz DEX doing", "on-chain volume", "DeFi overview", or wants a macro view of Chiliz Chain DeFi activity.

**Example request:**
```
GET /api/v1/defi/analytics?period=24h
```

**Example response:**
```json
{
  "period": "24h",
  "total_volume_usd": 4250000,
  "total_tvl_usd": 28500000,
  "unique_traders": 3420,
  "total_transactions": 18500,
  "volume_change_pct": 22.5,
  "tvl_change_pct": 1.8,
  "top_pairs_by_volume": [
    { "pair": "PSG/CHZ", "volume_usd": 485000 },
    { "pair": "BAR/CHZ", "volume_usd": 412000 },
    { "pair": "CHZ/USDT", "volume_usd": 380000 },
    { "pair": "JUV/CHZ", "volume_usd": 295000 },
    { "pair": "ACM/CHZ", "volume_usd": 218000 }
  ],
  "gas_avg_gwei": 2.5,
  "gas_trend": "stable"
}
```

**Interpreting results:**
- **total_volume_usd:** Daily volume across all Chiliz DEX pairs. Useful as a health metric.
- **unique_traders:** Active user count. Growing trader count is bullish for the ecosystem.
- **volume_change_pct:** Positive = growing activity, often match-day driven.
- **top_pairs_by_volume:** Shows which tokens are most actively traded on-chain.
- **gas_avg_gwei:** Chiliz Chain has low gas fees. This is rarely a concern but worth noting if there is a spike.

## DeFi Concepts for Agent Context

When discussing DeFi with users, keep these fan-token-specific nuances in mind:

1. **Impermanent loss** is especially relevant because fan token prices are volatile around matches. LPs in a PSG/CHZ pool during a match day may experience significant IL.
2. **Staking single-sided** is the safest yield option and should be the default recommendation for users who are not DeFi-savvy.
3. **Governance participation** is a unique value proposition of fan tokens. Encourage users to vote -- it is the primary non-financial utility.
4. **TVL on Chiliz Chain** is smaller than major L1s. Liquidity is concentrated in a few pools. Always check depth before recommending large trades.
5. **CHZ is the base asset** on Chiliz Chain. Most fan token pairs are quoted against CHZ, not USDT. Users may need to acquire CHZ first.
