# Signal Ingestion Specification

How a creator's charting tool delivers a signal to Public.

---

## Design principle

**A signal is an intent, not an order.** It carries what the creator wants done and never carries an account, a quantity, or a dollar amount. Public converts intent into a per-subscriber order using that subscriber's own configuration.

This is not a stylistic choice. A signal carrying size for a particular account is advice tailored to that person, which breaks the publisher's exclusion and, for futures, the CTA exemption independently.

## Endpoint

```
POST https://api.public.com/creator/v1/agents/{agent_id}/signals
```

One endpoint per agent. The URL is not a secret — authentication is by signature, not by URL obscurity, because TradingView webhook URLs are routinely pasted into screenshots and support threads.

## Authentication

HMAC-SHA256 over the raw request body, keyed on a per-agent signing secret.

| Header | Required | Notes |
|---|---|---|
| `X-Public-Signature` | Yes | `sha256=<hex digest>` over the exact raw body bytes |
| `X-Public-Timestamp` | Yes | Unix seconds. Rejected if more than 300s from server time |
| `X-Public-Agent-Id` | Yes | Must match the path parameter |
| `Idempotency-Key` | Yes | Creator-supplied. See deduplication |

The signed payload is `timestamp + "." + raw_body` to prevent replay of a captured body under a new timestamp.

**TradingView constraint that matters:** TradingView's alert webhooks cannot compute an HMAC — they send a static body to a static URL with no signing capability. Phase one therefore needs a second, weaker mode:

**Mode B — bearer token in URL path.** A high-entropy, per-agent, rotatable token embedded in the path:

```
POST https://api.public.com/creator/v1/agents/{agent_id}/signals/t/{token}
```

Mode B is rate-limited more aggressively, requires the token be rotatable in one click, and must surface a "this URL is a credential" warning in the creator UI. Mode A is required for any agent above a subscriber threshold — eng and Compliance to set the threshold. Do not ship Mode B without rotation and a visible last-used-from-IP log.

## Request body

```json
{
  "idempotency_key": "tv-4417-20260917T143000Z",
  "action": "ENTRY",
  "instrument_type": "EQUITY",
  "symbol": "AAPL",
  "side": "BUY",
  "order_type": "LIMIT",
  "limit_price": 227.50,
  "stop_price": null,
  "time_in_force": "DAY",
  "session": "CORE",
  "position_ref": "aapl-swing-2026-09-17",
  "proposal_ttl_seconds": 900,
  "note": "Breakout above prior day high, holding above 20DMA.",
  "signal_time": "2026-09-17T14:30:00Z"
}
```

### Field reference

| Field | Type | Required | Notes |
|---|---|---|---|
| `idempotency_key` | string | Yes | Unique per agent. Replays return the original result |
| `action` | enum | Yes | `ENTRY`, `EXIT`, `CANCEL` |
| `instrument_type` | enum | Yes | `EQUITY`, `OPTION`. `CRYPTO` phase 1.5 |
| `symbol` | string | Yes | Ticker, or exact OSI symbol for options |
| `side` | enum | Yes | `BUY`, `SELL` |
| `order_type` | enum | Yes | `LIMIT`, `MARKET`, `STOP`, `STOP_LIMIT` |
| `limit_price` | number | Conditional | Required for `LIMIT` and `STOP_LIMIT` |
| `stop_price` | number | Conditional | Required for `STOP` and `STOP_LIMIT` |
| `time_in_force` | enum | No | `DAY` (default), `GTC` |
| `session` | enum | No | `CORE` (default), `EXTENDED`. Equities only |
| `open_close` | enum | Conditional | `OPEN`, `CLOSE`. Options only |
| `position_ref` | string | Conditional | Required on `EXIT` and `CANCEL`. Ties an exit to its entry |
| `proposal_ttl_seconds` | integer | No | Agent default if omitted. Clamped to agent maximum |
| `note` | string | No | Creator's rationale, shown to subscribers verbatim. Max 500 chars |
| `signal_time` | ISO 8601 | No | Creator's own timestamp, for diagnostics only |

### Explicitly rejected fields

If any of these appear, reject the whole request with `422` and a clear error. They must not be silently ignored, because silent ignoring lets a creator believe they are setting size.

`quantity`, `amount`, `notional`, `account_id`, `percent_of_account`, `risk_percent`, `leverage`, `subscriber_id`, `target_subscribers`

## Options and the contract selection line

Phase one requires an **exact OSI symbol**:

```
AAPL260320C00280000
└┬─┘└──┬─┘│└───┬───┘
 │     │  │    └ strike, $280.00
 │     │  └ C call / P put
 │     └ expiry 2026-03-20
 └ underlying
```

Public will **not** accept relative selection such as "30 delta call, 45 DTE" in phase one. If Public resolves criteria into a specific contract, Public is exercising judgment inside the recommendation, and the analysis of whether we have become a participant in the advice changes materially. That needs Legal before it ships, and it is not worth blocking phase one on.

Practical consequence: creators must emit the OSI symbol from their charting tool. TradingView can do this via alert message templating. This needs to be in creator onboarding documentation with a worked example, and it is the single most likely source of creator integration friction. Budget support time for it.

## Deduplication

Chart platforms fire duplicate alerts — on bar close plus bar reopen, on reconnect, on manual re-save of an alert.

- `idempotency_key` unique per `(agent_id, key)`, retained 7 days
- A replay returns `200` with the original `signal_id` and creates no new proposals
- A different body under a previously used key returns `409`
- Additionally, a soft duplicate guard: identical `(agent_id, symbol, side, action)` within 60 seconds is flagged for the creator's diagnostics and, above an agent-configured threshold, held for creator review rather than fanned out

## Rate limits

| Scope | Limit | On breach |
|---|---|---|
| Per agent, burst | 10 signals / minute | `429`, retry-after |
| Per agent, sustained | Agent's declared frequency band | Accepted, agent flagged, listing status degraded |
| Per agent, hard ceiling | 100 signals / day | Rejected, agent suspended pending review |
| Mode B token | Half the Mode A limits | Same |

Sustained-band breaches must not silently drop signals. A creator who cannot trust delivery will leave, and a dropped signal after a subscriber saw the alert elsewhere is a support and trust problem.

## Responses

| Code | Meaning |
|---|---|
| `202` | Accepted, fan-out queued. Returns `signal_id` and `subscriptions_targeted` |
| `200` | Idempotent replay. Returns the original `signal_id` |
| `400` | Malformed body |
| `401` | Bad or missing signature |
| `403` | Agent suspended, not live, or outside declared instrument scope |
| `409` | Idempotency key reused with a different body |
| `422` | Semantically invalid — unknown symbol, prohibited field present, missing conditional field |
| `429` | Rate limited |

Success response:

```json
{
  "signal_id": "sig_01J8XQ",
  "status": "ACCEPTED",
  "subscriptions_targeted": 412,
  "received_at": "2026-09-17T14:30:00.412Z"
}
```

`subscriptions_targeted` is a count only. It must never be broken down in any creator-facing response.

## Fan-out latency budget

One signal to 400 subscriptions is 400 preflight calls and 400 notifications. Fan-out must be fair — subscription 400 cannot be materially later than subscription 1, or early subscribers get systematically better fills and we have built a priority tier by accident.

| Stage | Budget |
|---|---|
| Signal accepted → all proposals created | < 2s p99 |
| Proposal created → notification dispatched | < 1s p99 |
| Spread between first and last notification | < 1s p99 |

Preflight is the expensive step. Proposals should be created and notified on a snapshot sizing computation, with preflight results attached asynchronously and refreshed at confirmation time. Do not serialize fan-out behind preflight.

## Ordering

Signals from one agent must be processed in received order. An `EXIT` arriving before its `ENTRY` has been submitted must queue behind it, not race it. Per-agent ordered processing with per-subscription independent execution.
