# Data Model and Proposal Lifecycle

---

## Entities

### Creator
The person or entity publishing. One creator may own several agents.

| Field | Notes |
|---|---|
| `creator_id` | |
| `legal_entity_name`, `entity_type` | Individual or business |
| `agreement_version`, `agreement_accepted_at` | Which creator agreement version they are bound by |
| `payout_method_id` | |
| `registration_status` | `NONE`, `RIA`, `CTA`, `IAR`. Phase one is `NONE` only; the others gate phase two |
| `disclosure_block` | Creator-authored, required, shown on the listing |
| `status` | `PENDING_REVIEW`, `ACTIVE`, `SUSPENDED`, `TERMINATED` |

### Agent
A signal stream. This is the unit a subscriber subscribes to and the unit performance is computed for.

| Field | Notes |
|---|---|
| `agent_id`, `creator_id` | |
| `name`, `description` | Creator-authored |
| `instrument_scope` | Allowed instrument types and optionally an underlying allowlist. Signals outside scope are rejected |
| `frequency_band` | `LOW` (<10/wk), `MEDIUM` (<5/day), `HIGH` (<20/day). Sets rate limits and listing eligibility |
| `default_proposal_ttl_seconds`, `max_proposal_ttl_seconds` | |
| `signing_secret_id`, `auth_mode` | `HMAC` or `PATH_TOKEN` |
| `status` | `SANDBOX`, `PENDING_REVIEW`, `LIVE`, `PAUSED`, `SUSPENDED`, `RETIRED` |
| `price_cents`, `billing_interval` | |
| `listing_visibility` | `PUBLIC`, `UNLISTED`, `HIDDEN` |

An agent's `instrument_scope` and `frequency_band` are declared before launch and are part of what a subscriber relies on. Changing either requires re-review and must notify active subscribers.

### Subscription
The link between a subscriber and an agent. **This is where all sizing and risk configuration lives.**

| Field | Notes |
|---|---|
| `subscription_id`, `subscriber_id`, `agent_id`, `account_id` | |
| `status` | `ACTIVE`, `PAUSED`, `PAST_DUE`, `CANCELLED` |
| `sizing_rule` | `FIXED_QUANTITY`, `FIXED_NOTIONAL`, `PCT_BUYING_POWER` |
| `sizing_value` | Shares/contracts, dollars, or percent depending on rule |
| `max_order_notional` | Hard per-order ceiling |
| `max_open_positions` | Fan-out skips new entries when at the cap |
| `max_daily_orders` | |
| `instrument_allowlist` | Optional subscriber-side narrowing |
| `auto_confirm` | **Must not exist.** Recorded here explicitly so no one adds it later |
| `notification_channels` | Push, email, or both |

Set by the subscriber, only ever by the subscriber. No API path may allow a creator, a Public employee, or a support tool to write these fields on a subscriber's behalf.

### Signal
The creator's intent, as received. Immutable.

| Field | Notes |
|---|---|
| `signal_id`, `agent_id`, `idempotency_key` | |
| `action`, `instrument_type`, `symbol`, `side`, `order_type` | |
| `limit_price`, `stop_price`, `time_in_force`, `session`, `open_close` | |
| `position_ref` | Links `EXIT` to `ENTRY` |
| `note` | Creator's rationale, displayed verbatim |
| `received_at`, `reference_quote`, `reference_quote_at` | Reference quote is captured at receipt and used for slippage measurement, never for display as a fill price |
| `subscriptions_targeted` | Count |

### OrderProposal
One per subscription per signal. The core working entity.

| Field | Notes |
|---|---|
| `proposal_id`, `signal_id`, `subscription_id`, `account_id` | |
| `state` | See state machine |
| `computed_quantity` or `computed_amount` | Result of applying the subscription's sizing rule |
| `sizing_explanation` | Human-readable, shown on the confirm screen: "3 shares — 2% of $18,400 buying power" |
| `preflight_result` | Estimated cost, fees, buying-power impact, timestamp |
| `expires_at` | |
| `notified_at`, `confirmed_at`, `submitted_at` | |
| `broker_order_id` | Set on submission |
| `skip_reason` | Set when never notified: `INSUFFICIENT_BUYING_POWER`, `POSITION_CAP`, `DAILY_ORDER_CAP`, `NOTIONAL_CEILING`, `INSTRUMENT_NOT_ALLOWED`, `ACCOUNT_RESTRICTED`, `MARKET_CLOSED`, `PREFLIGHT_FAILED` |
| `terminal_reason` | Set on `EXPIRED`, `DECLINED`, `VOIDED`, `REJECTED` |

### Fill
Captured from the order lifecycle. The **only** input to performance.

| Field | Notes |
|---|---|
| `fill_id`, `proposal_id`, `broker_order_id` | |
| `filled_quantity`, `average_price`, `fees`, `filled_at` | |
| `position_ref` | Carried from the signal, for round-trip matching |

---

## Proposal state machine

```
                    ┌──────────────┐
                    │   CREATED    │
                    └──────┬───────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
       (guardrail       (ok)         (preflight
        or cap hit)       │            failed)
            │             ▼              │
            │      ┌─────────────┐       │
            │      │  NOTIFIED   │       │
            │      └──────┬──────┘       │
            │             │              │
            │   ┌─────────┼─────────┐    │
            │   │         │         │    │
            │ (confirm) (decline) (ttl)  │
            │   │         │         │    │
            │   ▼         ▼         ▼    ▼
            │ ┌────────┐ ┌────────┐ ┌────────┐
            └▶│ SKIPPED│ │DECLINED│ │EXPIRED │
              └────────┘ └────────┘ └────────┘
                  ▲
        ┌─────────┴──────────┐
        │                    │
   ┌──────────┐        ┌──────────┐
   │CONFIRMED │───────▶│SUBMITTED │
   └──────────┘        └────┬─────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │  FILLED  │  │ PARTIAL  │  │ REJECTED │
        └──────────┘  └──────────┘  └──────────┘
```

Plus `VOIDED`, reachable from `CREATED` or `NOTIFIED` when the creator sends `CANCEL` for the signal, or when the underlying is halted.

### Transition rules

| From | To | Trigger |
|---|---|---|
| `CREATED` | `NOTIFIED` | Guardrails pass, notification dispatched |
| `CREATED` | `SKIPPED` | Any guardrail, cap, or preflight failure. `skip_reason` required |
| `NOTIFIED` | `CONFIRMED` | Subscriber confirms before `expires_at` |
| `NOTIFIED` | `DECLINED` | Subscriber explicitly declines |
| `NOTIFIED` | `EXPIRED` | `expires_at` passes with no action |
| `NOTIFIED` / `CREATED` | `VOIDED` | Creator `CANCEL`, or symbol halted |
| `CONFIRMED` | `SUBMITTED` | Order accepted by the order API |
| `CONFIRMED` | `REJECTED` | Order API rejects |
| `SUBMITTED` | `FILLED` / `PARTIAL` / `REJECTED` | Order lifecycle |

### The transition that must not exist

**There is no path from `EXPIRED` to `SUBMITTED`, and no path from `NOTIFIED` to `SUBMITTED` that does not pass through `CONFIRMED`.**

Timeout means the intent is abandoned. This is the single most important invariant in the system: it is the difference between a notification product and discretionary trading authority. It should be enforced at the database level, not only in application code, and it needs a test that specifically attempts the illegal transition.

### Idempotency on confirm

Confirmation is a user action on a mobile device with unreliable connectivity. Double-tap and retry must not produce two orders. Confirm takes the `proposal_id` and is idempotent — the second call returns the same `broker_order_id`. Transition `NOTIFIED → CONFIRMED` must be a compare-and-set on state.

### Staleness at confirmation

A proposal can be confirmed up to `expires_at`, by which time the market has moved.

On confirm, before submission:
1. Re-run preflight
2. Re-fetch the quote
3. If a `LIMIT` price is now through the market beyond a configured tolerance, or buying power no longer covers the order, do **not** submit. Return the proposal to the subscriber with a re-confirmation prompt showing what changed
4. If the account has become restricted, transition to `SKIPPED`

This adds a round trip and it is worth it. Submitting a stale limit order that fills far from the creator's reference is the most likely source of subscriber complaints about a signal being "wrong."

---

## What the creator can read

Creator analytics are served from a separate read model that physically cannot join to subscriber identity.

**Available:** signals published, proposals created, aggregate confirm rate, aggregate time-to-confirm, aggregate fill statistics and dispersion, subscriber count, subscriber count change, revenue.

**Not available, and not derivable:** subscriber identity, account values, positions, individual fills, individual confirm behavior, per-subscriber anything.

Aggregates must be suppressed below a minimum cohort size — recommend 10 — or a creator with three subscribers can infer individual behavior from an average.
