# Fan Token Skills

Agent skills for fan-token intelligence on Chiliz Chain, backed by the live
Fan Token Intel surfaces: free REST API, free MCP server (streamable HTTP), and
the x402 pay-per-call gateway.

**6 skills. Descriptive data with honest statistics. Zero signup for the free
surfaces.**

```bash
npx skills add BrunoPessoa22/fan-token-skills
```

---

## Skills

| Skill | Domain | Backing surface |
|-------|--------|-----------------|
| `match-intel` | Matchday setups + goal/red-card event-impact profiles (n, CIs, hit rates) | x402 gateway ($0.01-$0.30/call) + free calendar |
| `sports-data` | Match schedules, live scores, match-to-price impact history | Free REST |
| `whale-intel` | CEX order-flow tilt, exchange inflows, large DEX swaps | Free REST + x402 |
| `defi-tools` | FanX DEX pools/TVL, validators, tokenomics, token registry | Free REST + x402 |
| `signal-scores` | Descriptive 0-100 composite condition scores | Free REST |
| `backtest-engine` | Historical matchday strategy replay (honestly negative) | Free REST |

---

## Live surfaces

| Surface | URL | Auth |
|---------|-----|------|
| REST API | `https://web-production-ad7c4.up.railway.app` (behind `https://www.fantokenintel.com`) | None for the endpoints these skills document |
| MCP server | `https://mcp-production-f681.up.railway.app/mcp` (streamable HTTP, 22 curated tools) | Optional Bearer key for higher limits |
| x402 gateway | `https://x402.brunopessoa.com/catalog` | Pay per call: USDT0 on OKX X Layer (`eip155:196`), HTTP 402 flow |

Connect the MCP server:

```bash
claude mcp add --transport http fan-token-intel https://mcp-production-f681.up.railway.app/mcp
```

---

## What your agent gets

- Upcoming/live match calendar mapped to fan tokens (World Cup fixtures included)
- Event-impact intelligence: how tokens reprice when their team scores,
  concedes, or takes a red card -- market-adjusted vs CHZ, with sample sizes,
  bootstrap CIs, hit rates, and FDR correction
- CEX buy/sell flow tilt and on-chain exchange inflows
- FanX DEX pool TVL, reserves, and slippage-relevant depth
- Chiliz validators with post-commission APRs, CHZ tokenomics, contract registry
- A backtest engine whose honest answer is usually "that strategy lost money"

## What this repo deliberately does NOT teach

- No trade execution, order routing, swaps, staking transactions, or anything
  that moves funds. Execution tooling is capability-walled off the public
  Fan Token Intel surface.
- No "signals" with advertised win rates, no alpha packets, no copy-trading.
  Those pre-pivot skills were deleted in July 2026 after verification against
  the live surface.

All data is descriptive history, not financial advice.

---

## Quick start

### Install
```bash
npx skills add BrunoPessoa22/fan-token-skills
```

### Ask your agent
```
Which fan-token teams play in the next 48 hours?
```
```
How does the ARG token historically react when Argentina concedes?
```
```
What is the CEX buy/sell tilt on JUV over the last 24 hours?
```
```
Backtest buying 2h before kickoff and selling at fulltime for PSG
```

---

## The x402 payment flow (paid SKUs)

Paid endpoints return HTTP `402 Payment Required` with a `payment-required`
header (base64 JSON, x402 version 2) specifying an `exact`-scheme USDT0 payment
on OKX X Layer. Any x402-capable client (x402 SDKs, OKX agentic wallet) settles
and retries automatically. **402 is the expected unpaid response, not an
outage.** Browse prices at `https://x402.brunopessoa.com/catalog` (free).

---

## CI: endpoint verification

`.github/workflows/verify-endpoints.yml` greps every URL documented in the
SKILL.md files and curls it daily and on every push. Any response outside
2xx (or 402 for paid x402 SKUs) fails the build, so this repo cannot silently
rot into documenting dead endpoints again.

---

## Compatible agents

Works with any agent runner that supports the skills format: Claude Code,
Cursor, GitHub Copilot, Gemini CLI, Windsurf, Codex, Goose, and others.

---

## License

MIT
