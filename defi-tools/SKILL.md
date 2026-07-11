---
name: defi-tools
description: Read-only Chiliz Chain DeFi data - FanX DEX pools and TVL, validators, tokenomics, token registry.
---

# DeFi Tools

Read-only DeFi data for Chiliz Chain: FanX DEX pool TVL and volume, TVL history,
Chiliz governance validators, CHZ tokenomics, and the fan-token contract
registry. This skill does not stake, swap, or move funds -- execution tooling is
walled off the public Fan Token Intel surface by design.

**Base URL (REST, free):** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_dex_summary
Ecosystem-level FanX DEX snapshot.

**Endpoint:** `GET /api/dex/summary`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/dex/summary
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "tvl": {
    "total_usd": 2926319.16,
    "change_1h": -1.64,
    "change_24h": 2.39,
    "change_7d": -5.68,
    "last_update": "2026-07-11T20:26:22.467950+00:00"
  },
  "pools": {
    "count": 93,
    "tracked_tvl": 2940030.61,
    "stale_pools_excluded": 81
  },
  "top_pools": [
    { "pair": "SPAIN/wCHZ", "tvl_usd": 439464.05 }
  ]
}
```

- `stale_pools_excluded` is an honesty feature: pools with stale reserves are
  excluded from the headline TVL instead of inflating it. Mention it when the
  user compares TVL numbers across sources.

### get_pools
FanX pools with reserves and 24h volume.

**Endpoint:** `GET /api/dex/pools`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 50 | Max pools returned |
| `min_tvl` | float | 100 | Minimum TVL (USD) |
| `token` | string | null | Filter by token symbol |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/dex/pools?limit=10
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "pools": [
    {
      "pool_address": "0x8572e4364de72134ab1d65260a50052595093075",
      "token0_symbol": "SPAIN",
      "token1_symbol": "wCHZ",
      "pair": "SPAIN/wCHZ",
      "dex_name": "FanX",
      "tvl_usd": 439464.05,
      "reserve0": 391329.77,
      "reserve1": 12467128.88,
      "volume_24h_usd": 8031.18,
      "last_update": "2026-07-11T20:26:22.467950+00:00"
    }
  ]
}
```

- Most pairs are quoted against wCHZ, not USDT. Volume in the low thousands per
  day is normal for this market -- warn users about depth before they think in
  CEX-scale sizes.
- `pool_address` feeds the paid `dex-pool-history` SKU and on-chain lookups.

### get_tvl_history
DEX TVL time series.

**Endpoint:** `GET /api/dex/tvl/history`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `hours` | int | 168 | Look-back window |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/dex/tvl/history?hours=24
```

Returns `{ "history": [ { "time": ..., "tvl_usd": ..., "change_24h": ... } ] }`.

### get_validators
Chiliz Chain governance validators.

**Endpoint:** `GET /api/governance/validators`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/governance/validators
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "validators": [
    {
      "name": "Meria",
      "address": "0xf84aeD72066e635FD9b0f2FbdDBdb77a8761d028",
      "type": "Active/Main",
      "apr_pct": 18.1,
      "delegator_apr_pct": 16.65,
      "commission_pct": 8.0,
      "total_delegated_chz": 167499473.0,
      "is_main": true
    }
  ]
}
```

- `delegator_apr_pct` (APR after commission) is the number a delegator actually
  earns -- quote that one, not the headline `apr_pct`.
- This surface lists validators; it does not stake. Staking is done by the user
  in their own wallet (e.g. Chiliz staking UI).

### get_tokenomics
CHZ supply, distribution, burn, and live staking aggregates.

**Endpoint:** `GET /api/governance/tokenomics`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/governance/tokenomics
```

Returns total supply (8,888,888,888 CHZ), distribution percentages, the
EIP-1559-style burn mechanism, and live validator/staking stats.

### get_token_registry
Fan-token contract addresses on Chiliz Chain.

**Endpoint:** `GET /api/governance/tokens/registry`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/governance/tokens/registry
```

Returns `{ "tokens": [ { "symbol": "ACM", "address": "0xF9C0...", "is_active": true, ... } ] }`.
Use it to resolve symbols to contracts before any on-chain lookup.

## Related surfaces

- **MCP (free):** `tokenintel_dex_liquidity`, `tokenintel_dex_depth`
  (constant-product slippage curves), `tokenintel_governance_validators` at
  `https://mcp-production-f681.up.railway.app/mcp`.
- **x402 (paid):** `dex-summary` ($0.01), `dex-pools` ($0.02),
  `dex-tvl-history` ($0.03), `dex-pool-history` ($0.03, needs `pool_address`),
  `token-chains` ($0.03, multi-chain identity for one symbol) at
  `https://x402.brunopessoa.com/catalog`.

## What this skill no longer claims

Earlier versions documented staking vaults with APYs, LP farms, governance
proposals with vote tallies, and swap execution. Those endpoints do not exist on
the live surface and the execution tools are capability-walled. If a user asks
for them, say so plainly instead of fabricating data.
