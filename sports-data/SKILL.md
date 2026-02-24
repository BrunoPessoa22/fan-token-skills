---
name: sports-data
description: Match schedules, live scores, results, and sports-price correlation data for fan token teams.
---

# Sports Data

Access match schedules, live scores, historical results, and the correlation between sports outcomes and token price movements. This is the raw sports data layer -- use prematch-alpha for trading-specific analysis, and this skill for general sports queries and correlation research.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_upcoming_matches
Fetch upcoming matches for fan token teams.

**Endpoint:** `GET /api/v1/matches/upcoming`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol (e.g. `PSG`). Omit for all teams. |
| `hours` | int | 48 | Look-ahead window in hours. Max 168. |
| `competition` | string | null | Filter by competition slug (e.g. `champions_league`, `serie_a`) |
| `min_importance` | int | 0 | Minimum importance 0-100 |

**When to use:** When the user asks "when does PSG play next", "upcoming matches", "what's on this week", or needs match schedule context.

**Example request:**
```
GET /api/v1/matches/upcoming?hours=48&min_importance=40
```

**Example response:**
```json
{
  "matches": [
    {
      "match_id": "ucl-2026-r16-psg-bay",
      "home_team": "Paris Saint-Germain",
      "home_token": "PSG",
      "away_team": "Bayern Munich",
      "away_token": null,
      "competition": "UEFA Champions League",
      "competition_slug": "champions_league",
      "round": "Round of 16 - Leg 2",
      "kickoff_utc": "2026-02-17T20:00:00Z",
      "importance": 92,
      "venue": "Parc des Princes",
      "status": "scheduled",
      "first_leg_score": "1-1"
    },
    {
      "match_id": "seriea-2026-w25-acm-nap",
      "home_team": "AC Milan",
      "home_token": "ACM",
      "away_team": "SSC Napoli",
      "away_token": "NAP",
      "competition": "Serie A",
      "competition_slug": "serie_a",
      "round": "Matchweek 25",
      "kickoff_utc": "2026-02-18T19:45:00Z",
      "importance": 74,
      "venue": "San Siro",
      "status": "scheduled",
      "first_leg_score": null
    }
  ],
  "count": 8
}
```

**Interpreting results:**
- **away_token null:** Bayern Munich does not have a fan token. Only Chiliz-based teams have tokens.
- **importance:** Based on competition stage, rival significance, and standings implications. UCL knockout = 90+, top-flight derby = 70-85, mid-table league match = 30-50.
- **first_leg_score:** For two-legged ties, shows the aggregate context.
- **status:** `scheduled`, `live`, `halftime`, `fulltime`, `postponed`, `cancelled`.

### get_live_matches
Fetch currently live matches.

**Endpoint:** `GET /api/v1/matches/live`

**Parameters:** None required. Returns all live matches involving fan token teams.

**When to use:** When the user asks "any matches on right now", "live scores", or you need real-time match context.

**Example response:**
```json
{
  "matches": [
    {
      "match_id": "ucl-2026-r16-psg-bay",
      "home_team": "Paris Saint-Germain",
      "home_token": "PSG",
      "away_team": "Bayern Munich",
      "away_token": null,
      "score": { "home": 2, "away": 1 },
      "minute": 67,
      "status": "live",
      "events": [
        { "minute": 12, "type": "goal", "team": "home", "player": "Dembele" },
        { "minute": 34, "type": "goal", "team": "away", "player": "Musiala" },
        { "minute": 58, "type": "goal", "team": "home", "player": "Kolo Muani" },
        { "minute": 62, "type": "red_card", "team": "away", "player": "Upamecano" }
      ],
      "token_prices": {
        "PSG": { "current": 3.68, "kickoff": 3.42, "change_pct": 7.6 }
      }
    }
  ],
  "count": 1
}
```

**Interpreting results:**
- **token_prices:** Shows real-time price movement since kickoff. PSG up 7.6% with a 2-1 lead -- classic match-day correlation.
- **events:** Key match events. Goals and red cards have the strongest price impact.
- Present live matches with score, minute, and token price change together for a unified view.

### get_match_result
Fetch the result and post-match analysis for a completed match.

**Endpoint:** `GET /api/v1/matches/{match_id}`

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `match_id` | path | The match identifier |

**When to use:** When the user asks about a past match result or you need post-match context.

**Example response:**
```json
{
  "match_id": "ucl-2026-r16-psg-bay",
  "home_team": "Paris Saint-Germain",
  "away_team": "Bayern Munich",
  "score": { "home": 3, "away": 1 },
  "status": "fulltime",
  "competition": "UEFA Champions League",
  "result": "home_win",
  "token_impact": {
    "PSG": {
      "price_at_kickoff": 3.42,
      "price_at_fulltime": 3.78,
      "price_1h_after": 3.85,
      "price_24h_after": 3.71,
      "max_price_post_match": 3.92,
      "kickoff_to_fulltime_pct": 10.5,
      "kickoff_to_1h_pct": 12.6,
      "kickoff_to_24h_pct": 8.5
    }
  },
  "events": [
    { "minute": 12, "type": "goal", "team": "home", "player": "Dembele" },
    { "minute": 34, "type": "goal", "team": "away", "player": "Musiala" },
    { "minute": 58, "type": "goal", "team": "home", "player": "Kolo Muani" },
    { "minute": 81, "type": "goal", "team": "home", "player": "Dembele" }
  ]
}
```

### get_price_correlation
Fetch historical sports-price correlation data.

**Endpoint:** `GET /api/v1/matches/correlation`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token. Omit for aggregate stats. |
| `competition` | string | null | Filter by competition slug |
| `result_type` | string | null | `win`, `loss`, `draw` |
| `days` | int | 90 | Look-back period |

**When to use:** When the user asks "how does PSG price react to wins", "does BAR go up after matches", "sports-price correlation", or for research/education about match-day patterns.

**Example request:**
```
GET /api/v1/matches/correlation?token=PSG&days=180
```

**Example response:**
```json
{
  "token": "PSG",
  "period_days": 180,
  "total_matches": 32,
  "correlations": {
    "win": {
      "count": 22,
      "avg_kickoff_to_fulltime_pct": 4.8,
      "avg_kickoff_to_1h_pct": 5.6,
      "avg_kickoff_to_24h_pct": 3.2,
      "median_move_pct": 4.1
    },
    "loss": {
      "count": 6,
      "avg_kickoff_to_fulltime_pct": -3.5,
      "avg_kickoff_to_1h_pct": -4.1,
      "avg_kickoff_to_24h_pct": -2.8,
      "median_move_pct": -3.2
    },
    "draw": {
      "count": 4,
      "avg_kickoff_to_fulltime_pct": -0.5,
      "avg_kickoff_to_1h_pct": -0.8,
      "avg_kickoff_to_24h_pct": -0.3,
      "median_move_pct": -0.4
    }
  },
  "pre_match_pattern": {
    "avg_4h_pre_to_kickoff_pct": 2.1,
    "avg_2h_pre_to_kickoff_pct": 1.4,
    "reliability": 0.78
  },
  "best_exit_window": "1h_post_match",
  "correlation_strength": 0.72
}
```

**Interpreting results:**
- **pre_match_pattern:** The pre-match pump is often the most consistent pattern. A reliability of 0.78 means it happened in ~78% of matches.
- **best_exit_window:** Data-driven suggestion for when the price impact peaks.
- **correlation_strength:** 0-1 overall. Above 0.6 means the token reliably reacts to match results. Below 0.4 means the connection is weak.
- **24h after:** Often shows mean reversion -- the initial spike fades. Present this honestly.
- Draws are typically slightly negative because the pre-match pump unwinds without a catalyst.

## Competition Slugs

| Slug | Competition |
|------|-------------|
| `champions_league` | UEFA Champions League |
| `europa_league` | UEFA Europa League |
| `conference_league` | UEFA Conference League |
| `premier_league` | English Premier League |
| `la_liga` | Spanish La Liga |
| `serie_a` | Italian Serie A |
| `ligue_1` | French Ligue 1 |
| `bundesliga` | German Bundesliga |
| `primeira_liga` | Portuguese Primeira Liga |
| `super_lig` | Turkish Super Lig |
| `copa_libertadores` | Copa Libertadores |
| `friendly` | International / Club Friendlies |

## Notes

- All times are UTC. Convert for the user if their timezone is known.
- **away_token** is null when the opponent does not have a fan token. Many opponents will not.
- Match importance is calculated by the system based on competition stage, standings, and rivalry history. It is not a user-tunable heuristic.
- For trading-specific analysis (predictions, entry/exit timing), use the prematch-alpha skill instead. This skill provides the raw data.
