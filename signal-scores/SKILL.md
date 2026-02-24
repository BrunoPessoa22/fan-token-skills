---
name: signal-scores
description: Retrieve unified 0-100 composite signal scores for fan tokens with direction and confidence breakdowns.
---

# Signal Scores

Unified composite scoring system for all fan tokens. Each token gets a 0-100 score combining on-chain flows, sports sentiment, social momentum, and technical signals. Use this as the primary entry point for "which tokens look interesting right now."

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### list_scores
Fetch scores for all tokens, optionally filtered by minimum score or direction.

**Endpoint:** `GET /api/v1/scores`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `min_score` | int | 0 | Only return tokens with composite score >= this value |
| `direction` | string | null | Filter by `bullish` or `bearish` |
| `sort_by` | string | `score` | Sort field: `score`, `symbol`, `change_1h` |

**When to use:** When the user asks "what tokens are hot", "any strong signals", "show me bullish tokens", or anything about overall market sentiment across fan tokens.

**Example request:**
```
GET /api/v1/scores?min_score=60&direction=bullish&sort_by=score
```

**Example response:**
```json
{
  "scores": [
    {
      "symbol": "PSG",
      "score": 82,
      "direction": "bullish",
      "confidence": 0.78,
      "components": {
        "on_chain": 75,
        "sports": 90,
        "social": 68,
        "technical": 85
      },
      "price_usd": 3.42,
      "change_1h": 2.1,
      "change_24h": 5.8,
      "updated_at": "2026-02-17T14:30:00Z"
    }
  ],
  "count": 12,
  "timestamp": "2026-02-17T14:30:00Z"
}
```

**Interpreting results:**
- **score 70-100:** Strong signal, worth highlighting to the user with specific component breakdown.
- **score 40-69:** Moderate activity, mention if relevant to the user's query.
- **score 0-39:** Low activity, only mention if user specifically asks about that token.
- **direction:** `bullish` means net positive momentum, `bearish` means net negative.
- **confidence:** 0-1 float indicating how reliable the composite signal is. Below 0.4, caveat the signal as "low confidence."
- **components:** Break these down when the user wants to understand *why* a token is scoring high. For example: "PSG is at 82 largely driven by sports sentiment (90) ahead of their Champions League match."

### get_token_score
Fetch the detailed score breakdown for a single token.

**Endpoint:** `GET /api/v1/scores/{symbol}`

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `symbol` | path | Token symbol, e.g. `PSG`, `BAR`, `JUV` (uppercase) |

**When to use:** When the user asks about a specific token's signal, e.g. "how does PSG look", "what's the score on BAR".

**Example request:**
```
GET /api/v1/scores/PSG
```

**Example response:**
```json
{
  "symbol": "PSG",
  "score": 82,
  "direction": "bullish",
  "confidence": 0.78,
  "components": {
    "on_chain": 75,
    "sports": 90,
    "social": 68,
    "technical": 85
  },
  "signals_active": [
    {
      "type": "MATCH_ALPHA",
      "detail": "Champions League vs Bayern Munich in 4h",
      "weight": 0.35
    },
    {
      "type": "WHALE_ACCUMULATION",
      "detail": "3 wallets accumulated 45k PSG in last 2h",
      "weight": 0.25
    }
  ],
  "price_usd": 3.42,
  "change_1h": 2.1,
  "change_24h": 5.8,
  "volume_24h": 892000,
  "updated_at": "2026-02-17T14:30:00Z"
}
```

**Interpreting results:**
- The `signals_active` array tells you exactly what is driving the score. Always surface these to the user.
- `weight` shows how much each active signal contributes to the composite. Higher weight = more influential.
- Combine score + active signals for a narrative: "PSG scores 82 (bullish) -- Champions League match in 4 hours is the main driver (weight 0.35), plus whale accumulation detected."

## Common Token Symbols

Standard fan token symbols used across all endpoints:
`PSG`, `BAR`, `JUV`, `ACM`, `ASR`, `ATM`, `MCI`, `POR`, `GAL`, `INTER`, `NAP`, `LAZ`, `OG`, `CHZ`, `SANTOS`, `CITY`, `AFC`, `LEG`, `ARG`, `TRA`, `ALPINE`

Always convert user input to uppercase before calling the API. If the user says "barcelona" use `BAR`, "paris saint-germain" use `PSG`, etc.
