---
name: Discover Legend markets and price history
description: >-
  Look up which perpetual markets Legend supports, their leverage and precision
  limits, and pull 24h price history for them — using Legend's public,
  unauthenticated read API.
api: openapi/legendtrade-info-openapi.yml
operations: [getHealth, postInfo, getSparklines, getCoinSparkline]
generated: '2026-07-19'
method: generated
source: openapi/legendtrade-info-openapi.yml + live probes of https://api.legend.trade
---

# Discover Legend markets and price history

Legend (legend.trade) is a competitive social trading platform whose orders
execute on Hyperliquid. Its public API at `https://api.legend.trade` is
**read-only and unauthenticated** for market data. Use it to answer "what can I
trade on Legend, at what leverage, and what has the price done today?"

## Before you start

- Base URL: `https://api.legend.trade`
- No API key is needed for any operation in this skill.
- There is **no rate-limit policy published**. Be conservative: batch coins into
  a single `getSparklines` call rather than looping per coin.

## 1. Confirm the service is healthy — `getHealth`

```
GET /health
```

Returns `status`, the deployed `gitSha`, an ISO-8601 `timestamp`, and per-dependency
checks (`hlVisor`, `postgres`) with `latencyMs`. If `status` is not `healthy`, or
`hlVisor` is degraded, market data may be stale — surface that rather than
presenting numbers as current.

## 2. List the tradable universe — `postInfo` with `type: meta`

```
POST /info
Content-Type: application/json

{"type": "meta"}
```

Returns `universe[]`. Each entry carries:

- `name` — the ticker, e.g. `BTC`
- `maxLeverage` — the cap for that asset (BTC was 40, ETH 25 at capture)
- `szDecimals` — size precision
- `marginTableId` — the margin tier table that governs it
- `isDelisted` — **check this**; delisted markets still appear in the array

Filter out `isDelisted: true` before presenting a tradable list.

## 3. Pull 24h price history — `getSparklines`

```
GET /sparklines?coins=BTC,ETH
```

Returns an object keyed by coin symbol, each with `candles[]` of `{t, c}` where
`t` is **epoch milliseconds** and `c` is the close price as a **decimal string**.

Parse `c` as a decimal, not a float, if you are doing arithmetic — Legend returns
strings deliberately to preserve precision.

For a single coin, `getCoinSparkline` (`GET /sparklines/{coin}`) is equivalent.

## 4. Other read types — `postInfo`

The same endpoint serves `metaAndAssetCtxs`, `candleSnapshot`, and `recentTrades`
without auth. Account types (`openOrders`, `frontendOpenOrders`,
`clearinghouseState`, `userFills`) exist but are account-scoped.

## Rules and gotchas

- **`/info` is POST-only.** `GET /info` returns 404.
- **Unknown types 400.** Sending a `type` outside the supported set returns
  `400 {"error":"Unknown request type: <value>"}`. Validate before sending.
- **Errors are a bare envelope.** Every error is `{"error": "<message>"}` — no
  code, no type URI, no RFC 9457 problem details. Branch on HTTP status.
- **No idempotency contract.** Legend documents none; do not assume safe retry
  semantics beyond the inherent idempotence of these GET/read operations.
- **No versioning.** Paths are unversioned and no version header exists. The
  `gitSha` on `/health` is the only build identifier — pin nothing to it.
- **Payload shapes beyond `meta` are not published.** Legend documents no object
  reference. Treat unmodelled responses defensively and consult Hyperliquid's
  docs (https://hyperliquid.gitbook.io/hyperliquid-docs) for the upstream shape.

## Related artifacts

- Conventions: `conventions/legendtrade-conventions.yml`
- Errors: `errors/legendtrade-problem-types.yml`
- Data model: `data-model/legendtrade-data-model.yml`
