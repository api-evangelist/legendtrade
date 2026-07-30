---
name: Stream Legend market and account events
description: >-
  Subscribe to Legend's Hyperliquid-compatible WebSocket for live trades,
  candles, asset context, fills, order updates, and account state.
api: asyncapi/legendtrade-ws-asyncapi.yml
operations: [receiveTrades, receiveCandle, receiveActiveAssetCtx, receiveUserFills, receiveOrderUpdates, receiveClearinghouseState]
generated: '2026-07-19'
method: generated
source: https://api.legend.trade (self-describing root, `wsSubscriptions`)
---

# Stream Legend market and account events

Legend exposes a subscription-based WebSocket at `wss://api.legend.trade/ws`.
It is **Hyperliquid-compatible** — the channel names below are Legend's own,
published in its API root document, and match Hyperliquid's WebSocket vocabulary.

## Before you start

- Endpoint: `wss://api.legend.trade/ws`
- A plain HTTPS `GET /ws` returns **404** — the path only responds to a
  WebSocket upgrade. Do not treat the 404 as "the endpoint is gone".
- Legend publishes **no message payload schemas**. Channel names are certain;
  message bodies are not. Parse defensively and consult Hyperliquid's
  WebSocket docs for the upstream shapes.

## Market data channels (public)

| Channel | Carries |
|---|---|
| `trades` | Executed trades |
| `candle` | Candle updates |
| `activeAssetCtx` | Context for the active asset |

These mirror the assets returned by `POST /info {"type":"meta"}` — resolve the
universe over REST first, then subscribe by symbol.

## Account data channels

| Channel | Carries |
|---|---|
| `userFills` | Fills against the account |
| `openOrders` | Current open orders |
| `orderUpdates` | Order lifecycle transitions |
| `clearinghouseState` | Margin, balance, position state |
| `notification` | Account notifications |
| `webData3` | Aggregated client state payload |
| `twapStates` | State of running TWAP orders |
| `userEvents` | Account events |
| `userFundings` | Funding payments |
| `userNonFundingLedgerUpdates` | Ledger movements other than funding |
| `activeAssetData` | Per-asset account data |
| `userTwapSliceFills` | Fills from individual TWAP slices |
| `userTwapHistory` | Historical TWAP orders |

## Choosing between WebSocket and `/info`

Every account channel has a REST equivalent as a `POST /info` request type
(`openOrders`, `clearinghouseState`, `userFills`). Use REST to establish initial
state on connect, then the WebSocket to stay current — do not poll `/info` in a
loop when a channel exists.

## Rules and gotchas

- **Reconnect on your own.** No reconnect, heartbeat, or backoff contract is
  published. Implement exponential backoff and re-subscribe after reconnect;
  re-establish state from `/info` because replay is not guaranteed.
- **TWAP is split across three channels.** `twapStates` (running),
  `userTwapSliceFills` (per-slice fills), `userTwapHistory` (completed). A TWAP
  order is not fully understood from any one of them.
- **Funding is separate from the ledger.** `userFundings` carries perpetual
  funding; `userNonFundingLedgerUpdates` carries everything else. Summing
  account movement requires both.
- **Prices are decimal strings** across Legend's surfaces. Do not coerce to float.
- **No event ordering or delivery guarantee is documented.** Treat the stream as
  at-most-once and reconcile against REST state.

## Related artifacts

- AsyncAPI: `asyncapi/legendtrade-ws-asyncapi.yml`
- Conventions: `conventions/legendtrade-conventions.yml`
- Data model: `data-model/legendtrade-data-model.yml`
