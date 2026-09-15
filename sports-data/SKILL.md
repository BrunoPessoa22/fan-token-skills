---
name: sports-data
description: Live match schedules, results, and historical match-to-price impact for fan token teams on Chiliz.
---

# Sports Data

Match schedules, live scores, and the historical price impact of match results for
fan-token teams. This is the free descriptive sports layer. For paid matchday
setups and event-conditioned reaction profiles (goal/red-card impact with
confidence intervals), use the match-intel skill.

**Base URL (REST, free):** `https://web-production-ad7c4.up.railway.app`
(the API behind `https://www.fantokenintel.com`)

All data is descriptive history, not financial advice.

## Commands

### get_upcoming_matches
Upcoming matches for fan-token teams.

**Endpoint:** `GET /api/matches/upcoming`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol (e.g. `PSG`). Omit for all teams. |
| `days` | int | 14 | Look-ahead window in days |
| `limit` | int | 50 | Max matches returned |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/matches/upcoming?days=14&limit=50
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "count": 50,
  "days": 14,
  "token_filter": null,
  "matches": [
    {
      "match_id": "apifb_1582681",
      "home_team": "Argentina",
      "away_team": "Switzerland",
      "match_date": "2026-07-12T01:00:00+00:00",
      "competition": "World Cup",
      "status": "scheduled",
      "importance_score": 0.65,
      "fixture_id": 1582681,
      "home_token": "ARG",
      "away_token": null
    }
  ]
}
```

**Interpreting results:**
- `home_token` / `away_token` are null when that team has no fan token. Only
  token-mapped fixtures matter for price analysis.
- `importance_score` is 0-1 (competition stage, rivalry, standings). World Cup
  knockouts and derbies score highest.
- `match_id` (`apifb_<fixture_id>`) is the key for the impact endpoint below.

### get_calendar
Matches grouped by date (past and future).

**Endpoint:** `GET /api/matches/calendar`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `start_date` | string | today | ISO date (YYYY-MM-DD) |
| `end_date` | string | +7d | ISO date |
| `token` | string | null | Filter by token symbol |

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/matches/calendar
```

Returns an object keyed by date, each value a list of matches in the same shape
as `get_upcoming_matches`, plus `home_score` / `away_score` for finished games.

### get_live_matches
Currently live matches involving fan-token teams.

**Endpoint:** `GET /api/matches/live`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/matches/live
```

**Example response (live capture, 2026-07-11):**
```json
{
  "count": 1,
  "matches": [
    {
      "match_id": "apifb_1562912",
      "home_team": "Sevilla",
      "away_team": "Juventud Torremolinos",
      "match_date": "2026-07-11T19:00:00+00:00",
      "competition": "Friendlies Clubs",
      "status": "live",
      "home_score": 1,
      "away_score": 0,
      "importance_score": 0.4,
      "home_token": "SEVILLA",
      "away_token": null,
      "price_at_kickoff": 0.050869
    }
  ]
}
```

- `price_at_kickoff` is the token price snapshot at kickoff (USD). Compare with a
  realtime price (MCP `tokenintel_realtime_prices`) to see the in-match move.

### get_match_impact
Match record plus price impact detail for one match.

**Endpoint:** `GET /api/matches/{match_id}/impact`

Take a finished `match_id` from `get_calendar` or `get_upcoming_matches`, e.g.
`GET /api/matches/apifb_1554860/impact`. Returns the match record (teams,
`token_symbol`, `token_result` win/loss/draw, `importance`, `is_derby`, scores)
plus price snapshots around the match where available.

### get_impact_stats
Aggregate match-impact profiles across the whole corpus.

**Endpoint:** `GET /api/matches/stats`

**Example request:**
```
https://web-production-ad7c4.up.railway.app/api/matches/stats
```

**Example response (live capture, 2026-07-11, truncated):**
```json
{
  "total_matches": 5113,
  "matches_with_price_data": 2434,
  "impact_profiles": [
    {
      "token_symbol": "GAL",
      "event_type": "knockout_advance",
      "avg_price_impact_pct": 3.65,
      "median_price_impact_pct": 2.22,
      "sample_count": 3
    }
  ]
}
```

**Interpreting results:**
- Always report `sample_count` next to any average. A +9.8% average on n=5 is an
  anecdote, not a pattern. Downrank small-n cells.
- Median and average diverging means outliers drive the number -- say so.

## Related surfaces

- **MCP (free, streamable HTTP):** `tokenintel_match_impact_history`,
  `tokenintel_match_correlation`, `tokenintel_match_event_replay` at
  `https://mcp-production-f681.up.railway.app/mcp`
  (`claude mcp add --transport http fan-token-intel https://mcp-production-f681.up.railway.app/mcp`).
- **x402 (paid, per-call):** `sports-calendar` ($0.01), `match-impact` ($0.04)
  and the event-impact family -- see the match-intel skill and
  `https://x402.brunopessoa.com/catalog`.

## Notes

- All times UTC.
- Match importance is system-computed, not user-tunable.
- Coverage: 5,000+ matches ingested, ~2,400 with joined price data (counts grow;
  check `/api/matches/stats` for current numbers).
