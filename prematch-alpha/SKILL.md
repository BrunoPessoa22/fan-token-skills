---
name: prematch-alpha
description: Pre-match alpha packets with price predictions, historical patterns, and optimal entry/exit timing for upcoming matches.
---

# Pre-Match Alpha

Generates alpha packets for upcoming sports matches. Each packet contains the match context, historical price patterns for that matchup type, predicted price impact, and suggested entry/exit windows. This is the core sports-driven trading intelligence.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### upcoming_alpha
List alpha packets for all upcoming matches within a time window.

**Endpoint:** `GET /api/v1/matches/alpha/upcoming`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol (e.g. `PSG`). Omit for all tokens. |
| `hours_ahead` | int | 24 | Look-ahead window in hours. Max 72. |
| `min_importance` | int | 0 | Minimum match importance 0-100 (Champions League knockout = 95, friendly = 15) |

**When to use:** When the user asks "any matches coming up", "what's the alpha for tonight", "pre-match signals", or anything about upcoming sports events and their trading implications.

**Example request:**
```
GET /api/v1/matches/alpha/upcoming?hours_ahead=12&min_importance=50
```

**Example response:**
```json
{
  "packets": [
    {
      "match_id": "ucl-2026-r16-psg-bay",
      "home_team": "Paris Saint-Germain",
      "away_team": "Bayern Munich",
      "competition": "UEFA Champions League",
      "round": "Round of 16 - Leg 2",
      "kickoff_utc": "2026-02-17T20:00:00Z",
      "importance": 92,
      "tokens_affected": [
        {
          "symbol": "PSG",
          "side": "home",
          "current_price": 3.42,
          "prediction": {
            "pre_match_pump": { "pct": 3.5, "confidence": 0.72, "window": "T-4h to T-1h" },
            "kickoff_impact": { "pct": 1.2, "confidence": 0.65 },
            "win_scenario": { "pct": 8.0, "confidence": 0.55 },
            "loss_scenario": { "pct": -5.0, "confidence": 0.60 }
          },
          "historical_pattern": {
            "avg_pre_match_move": 2.8,
            "avg_post_win_move": 6.5,
            "avg_post_loss_move": -4.2,
            "sample_size": 14
          },
          "suggested_entry": "2026-02-17T16:00:00Z",
          "suggested_exit": "2026-02-17T19:30:00Z"
        }
      ],
      "risk_factors": [
        "First leg ended 1-1, high uncertainty",
        "Mbappe injury rumor circulating (unconfirmed)"
      ]
    }
  ],
  "count": 3,
  "window_hours": 12
}
```

**Interpreting results:**
- **importance:** 80+ is a major match (knockout stages, derby, title decider). 50-79 is notable league play. Below 50 is lower-tier.
- **prediction.pre_match_pump:** The expected price rise before kickoff. This is the primary alpha -- buy before this window, sell at the end.
- **prediction.win_scenario / loss_scenario:** Conditional outcomes. Always present both to the user.
- **confidence:** Below 0.5, treat predictions as speculative. 0.5-0.7 is moderate. Above 0.7 is a historically reliable pattern.
- **historical_pattern.sample_size:** If below 5, warn the user that the pattern has limited data.
- **suggested_entry / suggested_exit:** Optimal timing based on historical analysis. Frame as "historically, the best window has been..."
- **risk_factors:** Always surface these. They are human-curated or system-detected caveats.

### get_match_alpha
Fetch the full alpha packet for a specific match.

**Endpoint:** `GET /api/v1/matches/alpha/{match_id}`

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `match_id` | path | The match identifier from the upcoming list |

**When to use:** When the user wants to drill into a specific match after seeing the upcoming list, or when you already know the match_id from context.

**Example request:**
```
GET /api/v1/matches/alpha/ucl-2026-r16-psg-bay
```

**Example response:**
Returns the same packet structure as above for the single match, but with additional fields:

```json
{
  "match_id": "ucl-2026-r16-psg-bay",
  "...same fields as above...",
  "deep_analysis": {
    "similar_matches": [
      {
        "date": "2025-03-08",
        "matchup": "PSG vs Real Madrid (UCL R16)",
        "pre_match_move": 4.1,
        "post_match_move": 9.2,
        "result": "PSG won 3-1"
      }
    ],
    "volume_profile": {
      "avg_volume_match_day": 1250000,
      "current_volume_trend": "above_average",
      "volume_ratio": 1.35
    },
    "social_buzz": {
      "mentions_24h": 4200,
      "sentiment": 0.68,
      "trending_topics": ["Mbappe", "Champions League", "PSG vs Bayern"]
    }
  }
}
```

**Interpreting deep analysis:**
- **similar_matches:** Show the user 2-3 most relevant historical parallels. Frame as "in similar situations, X happened."
- **volume_profile:** If `volume_ratio` > 1.5, there is unusual volume building up -- worth mentioning.
- **social_buzz:** High mentions with positive sentiment reinforces the bullish thesis. Negative sentiment could mean the pre-match pump is muted.

## Important Notes

- All times are UTC. Convert to the user's timezone if known.
- Predictions are probabilistic, not guarantees. Always frame them as historical tendencies.
- The pre-match pump is the most reliable pattern (highest sample sizes). Win/loss scenarios are more speculative.
- For tokens with low liquidity, the suggested entry/exit windows may have higher slippage -- cross-reference with the order-router skill if the user wants to execute.
