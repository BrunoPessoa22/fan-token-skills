---
name: oracle-signals
description: Oracle bearish alert system with active and historical signals for fan token risk detection.
---

# Oracle Signals

The Oracle is the bearish alert system. It monitors fan tokens for negative catalysts: unusual sell pressure, negative sports news, holder concentration spikes, social sentiment drops, and on-chain red flags. Use this when the user wants to check risk, see warnings, or understand bearish conditions.

**Base URL:** `https://web-production-ad7c4.up.railway.app`

## Commands

### get_active_signals
Fetch all currently active bearish signals.

**Endpoint:** `GET /api/v1/oracle/signals/active`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol. Omit for all tokens. |
| `min_severity` | string | null | Minimum severity: `low`, `medium`, `high`, `critical` |
| `signal_type` | string | null | Filter by type (see Signal Types below) |

**When to use:** When the user asks "any warnings", "what's the risk on PSG", "bearish signals", "oracle alerts", "anything I should worry about", or when you want to add risk context to a bullish thesis.

**Example request:**
```
GET /api/v1/oracle/signals/active?min_severity=medium
```

**Example response:**
```json
{
  "signals": [
    {
      "id": "sig-4821",
      "token": "JUV",
      "signal_type": "WHALE_DISTRIBUTION",
      "severity": "high",
      "title": "Large holder distributing JUV",
      "detail": "Top-5 wallet (0x3f2a...8b1c) has moved 280k JUV to Binance in 3 transactions over 6 hours. Historically, this wallet's exchange deposits preceded 4-8% drops within 48 hours.",
      "triggered_at": "2026-02-17T08:30:00Z",
      "expires_at": "2026-02-18T08:30:00Z",
      "confidence": 0.74,
      "price_at_trigger": 2.85,
      "current_price": 2.78,
      "price_change_since": -2.46,
      "suggested_action": "Avoid new longs on JUV. Consider reducing exposure if holding.",
      "related_signals": ["sig-4819", "sig-4815"]
    },
    {
      "id": "sig-4818",
      "token": "ACM",
      "signal_type": "NEGATIVE_SPORTS_NEWS",
      "severity": "medium",
      "title": "Key player injury confirmed",
      "detail": "ACM starting striker confirmed out 4-6 weeks. Historical pattern: key player injuries cause 2-5% token decline over 48h, especially with upcoming Serie A match.",
      "triggered_at": "2026-02-17T10:15:00Z",
      "expires_at": "2026-02-19T10:15:00Z",
      "confidence": 0.62,
      "price_at_trigger": 1.92,
      "current_price": 1.89,
      "price_change_since": -1.56,
      "suggested_action": "Reduce pre-match alpha expectations for ACM's next match.",
      "related_signals": []
    }
  ],
  "count": 2,
  "summary": {
    "total_active": 5,
    "critical": 0,
    "high": 1,
    "medium": 3,
    "low": 1,
    "most_affected_token": "JUV"
  }
}
```

**Interpreting results:**
- **severity levels:**
  - `critical`: Immediate risk. Significant price impact likely. Always surface urgently.
  - `high`: Strong bearish signal. Worth proactive mention even if the user didn't ask about this token.
  - `medium`: Notable risk factor. Mention when discussing the affected token.
  - `low`: Minor concern. Only mention if the user specifically asks.
- **confidence:** Same 0-1 scale. Below 0.5, caveat heavily. Above 0.7, treat as high-conviction.
- **price_change_since:** Shows if the signal has already played out. If a high-severity signal triggered 6 hours ago and price is already down 5%, the move may be mostly done.
- **suggested_action:** Actionable recommendation from the Oracle. Present these to the user.
- **related_signals:** If multiple signals cluster on the same token, the bearish thesis is stronger.
- **expires_at:** Signals have a TTL. An expired signal has lost relevance. The API filters these out but may return signals close to expiry.

### get_signal_history
Fetch historical Oracle signals for analysis and pattern review.

**Endpoint:** `GET /api/v1/oracle/signals/history`

**Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | string | null | Filter by token symbol |
| `days` | int | 30 | Look-back period in days. Max 90. |
| `signal_type` | string | null | Filter by type |
| `outcome` | string | null | Filter by outcome: `correct`, `incorrect`, `neutral` |
| `min_severity` | string | null | Minimum severity |
| `limit` | int | 50 | Max results |

**When to use:** When the user asks "how accurate is the Oracle", "past bearish signals on PSG", "oracle track record", or when validating whether to trust an active signal.

**Example request:**
```
GET /api/v1/oracle/signals/history?token=JUV&days=60&outcome=correct
```

**Example response:**
```json
{
  "signals": [
    {
      "id": "sig-4102",
      "token": "JUV",
      "signal_type": "WHALE_DISTRIBUTION",
      "severity": "high",
      "title": "Whale distribution detected",
      "triggered_at": "2026-01-15T14:00:00Z",
      "expired_at": "2026-01-16T14:00:00Z",
      "confidence": 0.71,
      "outcome": "correct",
      "price_at_trigger": 3.10,
      "price_at_expiry": 2.88,
      "actual_move_pct": -7.1,
      "time_to_bottom": "18h 30m"
    }
  ],
  "stats": {
    "total_signals": 28,
    "correct": 18,
    "incorrect": 6,
    "neutral": 4,
    "accuracy_pct": 64.3,
    "avg_move_when_correct": -4.8,
    "avg_move_when_incorrect": 1.2,
    "by_type": {
      "WHALE_DISTRIBUTION": { "count": 8, "accuracy": 75.0 },
      "NEGATIVE_SPORTS_NEWS": { "count": 7, "accuracy": 57.1 },
      "SENTIMENT_DROP": { "count": 6, "accuracy": 66.7 },
      "CONCENTRATION_SPIKE": { "count": 4, "accuracy": 50.0 },
      "TECHNICAL_BREAKDOWN": { "count": 3, "accuracy": 66.7 }
    }
  },
  "count": 28
}
```

**Interpreting results:**
- **accuracy_pct above 60%:** The Oracle has a meaningful edge. Present signals with conviction.
- **accuracy_pct 50-60%:** Marginal. Caveat signals as "the Oracle flags this as a risk, but its track record on this type is mixed."
- **by_type accuracy:** Different signal types have different reliability. `WHALE_DISTRIBUTION` tends to be the most accurate. Use this to weight how seriously to take each signal.
- **avg_move_when_correct:** Shows the typical downside when the Oracle is right. Useful for risk sizing.
- **time_to_bottom:** How long until the low point. Helps with timing if the user wants to buy the dip after a bearish signal plays out.

## Signal Types

| Type | Description | Typical Accuracy |
|------|-------------|-----------------|
| `WHALE_DISTRIBUTION` | Large holders moving tokens to exchanges (sell pressure) | High (70%+) |
| `NEGATIVE_SPORTS_NEWS` | Injuries, suspensions, manager sacking, relegation risk | Medium (55-65%) |
| `SENTIMENT_DROP` | Social media sentiment turning sharply negative | Medium (60%) |
| `CONCENTRATION_SPIKE` | Sudden increase in holder concentration (whale accumulation for dump) | Medium (50-60%) |
| `TECHNICAL_BREAKDOWN` | Price broke key support level or bearish chart pattern | Medium (60-65%) |
| `VOLUME_ANOMALY` | Unusual volume pattern inconsistent with known catalysts | Low-Medium (50%) |
| `CORRELATION_BREAK` | Token diverging from its historical correlation with CHZ or peer tokens | Low-Medium (50%) |

## Best Practices

1. **Always check active Oracle signals before presenting bullish recommendations.** A token with a high-severity bearish signal active should have that mentioned alongside any bullish case.
2. **Cross-reference with whale-intel** for more context on distribution signals.
3. **Outcome-aware cooldown:** If a signal recently fired and was correct, the system suppresses duplicate signals on the same token for a cooldown period. Don't expect the same alert to repeat immediately.
4. **Per-token auto-suppression:** Tokens that triggered 3+ signals in a week are auto-suppressed to avoid alert fatigue. Check history if you suspect a token has been noisy.
