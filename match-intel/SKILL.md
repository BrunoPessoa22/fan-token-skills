---
name: match-intel
description: Matchday setups and event-impact intelligence for fan tokens - goal/red-card reaction profiles with sample sizes and CIs, paid via x402.
---

# Match Intel

The flagship Fan Token Intel surface: how fan tokens ACTUALLY reprice around
football events. Match setups for upcoming fixtures, and the event-impact
family -- goal and red-card reaction profiles, market-adjusted vs CHZ, with
honest statistics (sample size, confidence intervals, hit rates). Premium data
is paid per call over the x402 protocol; the live calendar is free.

Everything here is descriptive history, not financial advice.

## Surfaces

| Surface | URL | Cost |
|---------|-----|------|
| x402 gateway (premium SKUs) | `https://x402.brunopessoa.com` | $0.01-$0.30 per call, USDT0 on OKX X Layer |
| x402 catalog (browse SKUs) | `https://x402.brunopessoa.com/catalog` | Free |
| REST live calendar | `https://web-production-ad7c4.up.railway.app/api/matches/upcoming` | Free |
| MCP equivalents | `https://mcp-production-f681.up.railway.app/mcp` | Free (rate-limited) |

## The x402 payment flow

x402 SKUs are plain HTTPS GETs that return `402 Payment Required` until paid:

1. `GET https://x402.brunopessoa.com/catalog` (free) -- lists every SKU with
   `path`, `price`, and `required_param`.
2. `GET` the SKU path (e.g. `https://x402.brunopessoa.com/v1/match-setup`).
   Unpaid, you get HTTP **402** with a `payment-required` response header:
   base64-encoded JSON (`x402Version: 2`) whose `accepts[]` specifies scheme
   `exact`, network `eip155:196` (OKX X Layer), the USDT0 asset contract
   (`0x779ded0c9e1022225f8e0630b35a9b54be713736`), the amount in 6-decimal
   units, and the `payTo` address.
3. Pay with any x402-capable client (x402 SDKs, OKX agentic wallet) -- the
   client signs the USDT0 transfer, retries the request with the
   `X-PAYMENT` header, and receives the JSON body.
4. A **402 status is the expected unpaid response, not an error.** Payments are
   non-refundable; each call is billed at the catalog price.

## Free: live match calendar

**Endpoint:** `GET /api/matches/upcoming` on
`https://web-production-ad7c4.up.railway.app` (params `token`, `days`, `limit`)
and `GET /api/matches/live` for in-play fixtures. See the sports-data skill for
payload details.

```
https://web-production-ad7c4.up.railway.app/api/matches/upcoming?days=7
```

Use the free calendar to find token-mapped fixtures, then spend on premium SKUs
only for the tokens that matter.

## Paid: matchday setup SKUs

### match-setup ($0.06)
```
https://x402.brunopessoa.com/v1/match-setup
```
Each upcoming fan-token fixture with market-implied win/draw/loss probabilities
crossed with the token's historical abnormal-return profile, producing a
probability-weighted expected 24h move, downside scenario, and confidence.
The single best pre-match view per fixture.

### match-predictions ($0.03)
```
https://x402.brunopessoa.com/v1/match-predictions
```
The matchday watchlist: upcoming fixtures mapped to tokens with each token's
historical if-win / if-loss / if-draw price impact.

### match-sensitivity ($0.04)
```
https://x402.brunopessoa.com/v1/match-sensitivity
```
Ranking of which fan tokens swing most on match results (average volatility,
win/loss/draw impact, W-L-D record). Use it to pick which tokens are worth
watching at all.

### match-impact ($0.04)
```
https://x402.brunopessoa.com/v1/match-impact
```
Average post-match price change after wins vs losses vs draws, with a
high-impact (derby/major) amplification breakdown.

### sports-calendar ($0.01)
```
https://x402.brunopessoa.com/v1/sports-calendar
```
Upcoming matches mapped to fan tokens (paid mirror of the free calendar, for
agents already inside the x402 flow).

## Paid: the event-impact family (the moat)

Only computable with the token-to-team map joined to minute-level match events
and tick-level prices. All returns are market-adjusted vs CHZ at +15/+30/+60
minutes.

### event-impact-asymmetry ($0.30)
```
https://x402.brunopessoa.com/v1/event-impact-asymmetry
```
How a token reacts when its team SCORES vs CONCEDES. The headline finding:
scoring is largely priced in; conceding moves price. Ships `n` and hit-rate per
cell.

### event-impact-profile ($0.25)
```
https://x402.brunopessoa.com/v1/event-impact-profile
```
Full event-conditioned profiles: goal/red_card x for/against x minute bucket x
scoreline state x importance. Each cell carries match-clustered t-stat,
bootstrap 95% CI, hit rate, decay, and FDR correction. Omit a dimension to pool.

### event-impact-redcard ($0.30)
```
https://x402.brunopessoa.com/v1/event-impact-redcard
```
Red-card reaction profile. Rare, high-impact events with honest wide CIs; cells
with n<15 are explicitly flagged.

### event-impact-replay ($0.20)
```
https://x402.brunopessoa.com/v1/event-impact-replay
```
Event-by-event reaction tape for one match: each goal/red card with minute,
running score, scoreline state, and the token's market-adjusted reaction.
Provide `match_id`, or `token` (+ optional `date`).

## Reading n and CI honestly (required)

- **Never quote a cell effect without its `n`.** A +1.8% mean reaction on n=6
  is noise-compatible; on n=140 it is a finding.
- **Use the CI, not just the mean.** If the bootstrap 95% CI spans zero, say
  "no reliable effect", whatever the point estimate is.
- **Hit rate near 50% means coin-flip** even if the mean is nonzero (a few
  outliers dragging it).
- **FDR matters:** with many cells, some will look significant by chance; the
  profile SKU's FDR field exists to keep you honest. Prefer FDR-surviving
  cells.
- Red-card cells are structurally thin -- present them with their flagged-n
  caveats.

## Free MCP equivalents

Rate-limited free versions of the family exist on the MCP server
(`claude mcp add --transport http fan-token-intel https://mcp-production-f681.up.railway.app/mcp`):
`tokenintel_goal_direction_asymmetry`, `tokenintel_event_reaction_profile`,
`tokenintel_late_game_redcard_profile`, `tokenintel_match_event_replay`,
`tokenintel_match_impact_history`. The x402 SKUs are the commercial,
per-call-settled surface for autonomous agents with wallets.

## Terms

Data is provided as-is, may be incomplete or stale, and is not financial
advice. Payments are non-refundable. Full terms:
`https://x402.brunopessoa.com/terms`.
