---
name: signal-scores
description: Descriptive 0-100 composite condition scores for fan tokens with a five-component breakdown.
---

# Signal Scores

A descriptive 0-100 composite score per fan token summarizing current
conditions across five weighted components: whale flow, macro regime, price
momentum, social momentum, and sports catalyst. Use it as a quick "what is
active right now" ranking. It describes conditions; it is not a trade
recommendation and carries no win-rate claim.

**Base URL (REST, free):** `https://web-production-ad7c4.up.railway.app`

## Commands

### list_scores
Scores for all tracked tokens.

**Endpoint:** `GET /api/v1/scores`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `min_score` | int | 0 | Only tokens with composite score >= this value |
| `direction` | string | null | Filter by direction label |
| `sort_by` | string | `score` | Sort field |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/v1/scores?min_score=30
```

**Example response (live capture, 2026-07-11, truncated -- note: a bare JSON array):**
```json
[
  {
    "token_symbol": "JUV",
    "score": 41,
    "direction": "neutral",
    "components": {
      "whale_flow": { "max": 26.8, "score": 13.4 },
      "macro_regime": { "max": 19.6, "score": 10.77 },
      "price_momentum": { "max": 16.1, "score": 8.48 },
      "social_momentum": { "max": 10.7, "score": 8.99 },
      "sports_catalyst": { "max": 26.8, "score": 0.0 }
    },
    "computed_at": "2026-07-11T20:32:33.792920+00:00"
  }
]
```

**Interpreting results:**
- Each component reports `score` out of its `max` weight. Weights: whale flow
  26.8, sports catalyst 26.8, macro regime 19.6, price momentum 16.1, social
  momentum 10.7 (sums to 100).
- `sports_catalyst: 0` simply means no imminent mapped fixture -- common outside
  match windows.
- Scores cluster in the 35-45 band in quiet markets. Treat relative ranking as
  the information, not the absolute number.
- `direction` is a coarse label derived from the components. Do not present it
  as a prediction; there is no accuracy guarantee attached.
- Check `computed_at` for staleness before presenting.

### get_token_score
Detailed breakdown for a single token.

**Endpoint:** `GET /api/v1/scores/{symbol}`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/v1/scores/PSG
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "token_symbol": "PSG",
  "score": 38,
  "direction": "neutral",
  "components": {
    "whale_flow": { "max": 26.8, "score": 13.4 },
    "macro_regime": { "max": 19.6, "score": 10.85 },
    "price_momentum": { "max": 16.1, "score": 7.59 },
    "social_momentum": { "max": 10.7, "score": 6.43 },
    "sports_catalyst": { "max": 26.8, "score": 0.0 }
  },
  "computed_at": "2026-07-11T20:38:21.407753+00:00",
  "active_signals": [],
  "upcoming_matches": [],
  "recent_whales": [
    {
      "direction": "buy",
      "volume_usd": 20.05,
      "exchange": "binance",
      "detected_at": "2026-07-11T20:38:07.507264+00:00"
    }
  ]
}
```

**Interpreting results:**
- `recent_whales` on this surface includes small trades (tens of USD) -- it is
  recent flow, not literal whales. Use the whale-intel skill for size-filtered
  flow and thresholds.
- `upcoming_matches` populates the sports component; cross-check with the
  sports-data skill for the full calendar.
- Narrate the components, not just the composite: "PSG scores 38 (neutral);
  whale flow contributes 13.4/26.8, no sports catalyst in window."

## Related surfaces

- **MCP (free):** `tokenintel_health_matrix` (A-F grades, similar intent),
  `tokenintel_briefing` at `https://mcp-production-f681.up.railway.app/mcp`.
- **x402 (paid):** `market-regime` ($0.02) and `token-context` ($0.03) at
  `https://x402.brunopessoa.com/catalog`.

## Notes

- Common symbols: `PSG`, `BAR`, `JUV`, `ACM`, `INTER`, `ATM`, `CITY`, `GAL`,
  `ASR`, `NAP`, `POR`, `ARG`, `SPAIN`, `OG`, `SANTOS`, `ALPINE`, `CHZ`.
  Uppercase before calling.
- Earlier versions of this skill documented an `on_chain/sports/social/technical`
  component set and a score-accuracy endpoint. Those do not exist on the live
  surface; the payloads above are the real contract.
