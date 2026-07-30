# LegendTrade

Legend (legend.trade) is a competitive social trading platform — "the esports layer for trading". Users trade perpetual contracts across 500+ crypto, equity, and commodity markets with leverage, wrapped in game mechanics: Arena head-to-head duels with ELO ratings and wagers, clans, leaderboards, a social feed, referrals, and copy trading. Orders execute on Hyperliquid's on-chain order book, with Legend charging a builder-code platform fee on top of Hyperliquid's volume-tiered maker/taker fees. Available on iOS, Android, web, and a Telegram bot.

Backed by: electric-capital — https://www.legend.trade/

## API surface

Legend runs a **Hyperliquid-compatible read API** at `https://api.legend.trade`, self-identifying as `@legend/hl-node`:

- `POST /info` — multiplexed market and account queries selected by request `type`
- `GET /sparklines` — Legend-custom 24h close-price candles
- `GET /health` — build SHA and per-dependency latency
- `POST/DELETE /wallets/*` — authenticated fill-tracking surface
- `wss://api.legend.trade/ws` — subscription WebSocket, 3 market-data and 13 account channels

## Artifacts

| Artifact | Method |
|---|---|
| `openapi/` | generated from the API's self-describing root + live probes |
| `asyncapi/` | generated from the root document's `wsSubscriptions` |
| `llms/` | **searched** — Legend publishes a real `llms.txt` |
| `conventions/`, `errors/`, `lifecycle/`, `conformance/`, `data-model/`, `authentication/` | derived |
| `security/legendtrade-domain-security.yml` | probed |
| `mcp/`, `skills/`, `overlays/` | generated |
| `well-known/` | probed — all paths 404, recorded as a negative result |

**Note on the published spec.** Legend's `llms.txt` links an `openapi.json` at `docs.legend.trade/api-reference/openapi.json`. As of 2026-07-19 that file is the unmodified Mintlify "OpenAPI Plant Store" sample scaffold and does not describe the Legend API — it was deliberately not harvested. The spec in `openapi/` was generated from the API's own self-describing root document and verified against live endpoints.

**Not present** (probed, absent): status page, trust center, security.txt, deprecation policy, SLA, changelog, first-party SDKs, CLI, sandbox, OAuth, webhooks, rate-limit documentation.
