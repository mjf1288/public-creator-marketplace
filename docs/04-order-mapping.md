# Order Mapping — Signal Intent to Public Order API

How a proposal becomes an order on the existing Public order API. No new order primitives are required for phase one.

---

## Existing API surface this maps onto

Confirmed against the current Public API:

| Parameter | Accepted values |
|---|---|
| `type` | `EQUITY`, `OPTION`, `CRYPTO` |
| `side` | `BUY`, `SELL` |
| `order_type` | `LIMIT`, `MARKET`, `STOP`, `STOP_LIMIT` |
| `quantity` or `amount` | Shares/contracts, or notional dollars |
| `limit_price` | Required for `LIMIT`, `STOP_LIMIT` |
| `stop_price` | Required for `STOP`, `STOP_LIMIT` |
| `session` | `CORE`, `EXTENDED` (equities) |
| `open_close` | `OPEN`, `CLOSE` (options) |
| `time_in_force` | `DAY`, `GTC` |
| `account_id` | |

Also available and used here: **preflight**, returning estimated cost, buying-power impact and fees; order placement returning an order id asynchronously; order cancellation by id.

**Notable:** the order API has no futures instrument type. Futures is out of phase-one scope for market data licensing reasons, and the API surface independently confirms it.

## Field mapping

| Signal field | Order parameter | Transform |
|---|---|---|
| `instrument_type` | `type` | Direct |
| `symbol` | `symbol` | Direct. OSI symbol for options |
| `side` | `side` | Direct |
| `order_type` | `order_type` | Direct |
| `limit_price` | `limit_price` | Direct. Not adjusted per subscriber |
| `stop_price` | `stop_price` | Direct |
| `time_in_force` | `time_in_force` | Direct, default `DAY` |
| `session` | `session` | Direct, default `CORE` |
| `open_close` | `open_close` | Direct for options |
| — | `quantity` / `amount` | **Computed from the subscription, never from the signal** |
| — | `account_id` | From the subscription |

The limit price is passed through unchanged. Public does not improve, skew or per-subscriber adjust the creator's price — that would be Public modifying the recommendation.

## Sizing computation

| Rule | Computation | Emits |
|---|---|---|
| `FIXED_QUANTITY` | `sizing_value` directly | `quantity` |
| `FIXED_NOTIONAL` | `sizing_value` dollars | `amount` for equities/crypto; for options, `floor(value ÷ (premium × 100))` as `quantity` |
| `PCT_BUYING_POWER` | `buying_power × pct` | `amount` for equities/crypto; contracts via premium for options |

Rules applied in order after the raw computation:

1. Clamp to `max_order_notional`
2. Options round **down** to whole contracts. If the result is zero contracts, `SKIPPED` with `INSUFFICIENT_BUYING_POWER` rather than rounding up to one
3. Equities: use `amount` where fractional is supported, `quantity` where it is not — check instrument fractional eligibility
4. If preflight shows insufficient buying power, `SKIPPED`. **Never** silently reduce size to fit; the subscriber configured a size and a reduced fill is a surprise
5. `EXIT` signals size from the **actual open position** matched on `position_ref`, not from the sizing rule. An exit closes what is open, capped at the position held

Every proposal stores a `sizing_explanation` string rendered on the confirm screen. A subscriber must never see a bare quantity without knowing how it was derived.

## Exit handling without bracket orders

Bracket, OCO and OTO support is in flight but not required for phase one.

Phase one: the creator publishes an `ENTRY` and, later, a separate `EXIT` carrying the same `position_ref`. Public matches the exit to fills from the original entry for that subscription and proposes a closing order sized to the position actually held.

Consequences to design for:
- A subscriber who declined the entry has no position. The exit is `SKIPPED` with no notification and no error surfaced to the creator beyond aggregate counts
- A subscriber who partially filled exits only the filled quantity
- A subscriber who closed the position manually exits nothing. `SKIPPED`
- A creator emitting an `EXIT` for an unknown `position_ref` gets `422`

**When brackets land**, the entry proposal can carry optional `stop_loss` and `take_profit` from the signal and submit as a bracket on confirmation. That removes the exit-confirmation step entirely, which materially improves the product: the subscriber confirms once and the protective legs are already resting. Treat this as the highest-value phase 1.5 item, because a stop that requires a confirmation is not a stop.

Until then, the creator agreement and the listing must both state that exits require a separate confirmation and may not execute if the subscriber does not act.

## Preflight

Preflight runs twice.

**At fan-out**, to populate the confirm screen with estimated cost, fees and buying-power impact, and to catch guardrail failures before notifying. Runs asynchronously, attached to the proposal after creation, so fan-out is not serialized behind it.

**At confirmation**, immediately before submission, to catch drift. See the staleness rules in the data model document.

Preflight failures at fan-out produce `SKIPPED` with `PREFLIGHT_FAILED`. Preflight failures at confirmation return the subscriber to a re-confirmation state rather than submitting.

## Submission

On confirmation and successful re-preflight, place the order with the mapped parameters and the subscription's `account_id`. Order placement is asynchronous — store the returned order id, transition to `SUBMITTED`, and reconcile fills from the order lifecycle.

The confirm surface must not claim the order is filled. It is submitted. Fill notification is a separate event.

## Idempotency at the broker boundary

A confirm retry must not place two orders. Carry a client order identifier derived from `proposal_id` on submission so a duplicate submission is rejected at the broker layer as well as the application layer. Belt and braces here — a duplicated live order is the worst failure this system can produce.

## Market hours

| Condition | Behavior |
|---|---|
| Signal during core hours | Normal |
| Signal outside core hours, `session=CORE` | Proposal created, notification deferred to pre-open, `expires_at` extended. Must be disclosed to the subscriber |
| Signal outside core hours, `session=EXTENDED` | Normal if the instrument supports extended trading |
| Symbol halted | Existing proposals `VOIDED`, new ones `SKIPPED` |

Overnight signals are a genuine design problem. A signal at 2am with a 15-minute TTL expires before anyone sees it. Deferring to pre-open changes the price context the creator was reasoning about. Phase one recommendation: agents declare whether they publish outside core hours, and if so their signals default to `GTC` with a TTL measured in hours, with the delay disclosed on the listing. Revisit for crypto in phase 1.5, where the market never closes and this becomes the central UX question rather than an edge case.
