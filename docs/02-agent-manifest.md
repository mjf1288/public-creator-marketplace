# Agent Manifest

How a creator declares an agent. This is the creator's product definition and the contract a subscriber relies on.

---

## Design intent

The manifest exists so that everything a subscriber needs to evaluate a creator is **declared in advance and enforced by the platform**, rather than claimed in marketing copy. A creator who declares "2 to 8 signals a week, SPY and QQQ options only" is held to it by rate limits and scope checks.

That is the core of why this is a better place to run than Whop or Discord: on those platforms a creator's claims are unverifiable, so buyers discount all of them. Here the claims are structural.

## Manifest

```json
{
  "name": "Weekly Trend Continuation",
  "summary": "Swing entries on large-cap breakouts with a 3–10 day hold.",
  "instrument_scope": {
    "instrument_types": ["EQUITY", "OPTION"],
    "underlying_allowlist": ["SPY", "QQQ", "AAPL", "MSFT", "NVDA"],
    "options_only_single_leg": true
  },
  "frequency_band": "LOW",
  "publishes_outside_core_hours": false,
  "default_proposal_ttl_seconds": 900,
  "max_proposal_ttl_seconds": 3600,
  "exit_policy": "CREATOR_SIGNALS_EXITS",
  "price_cents": 4900,
  "billing_interval": "MONTHLY",
  "disclosure_block": "Creator-authored. Required.",
  "auth_mode": "PATH_TOKEN"
}
```

## Field reference

| Field | Required | Notes |
|---|---|---|
| `name`, `summary` | Yes | Creator-authored. Reviewed at onboarding, not written by Public |
| `instrument_scope.instrument_types` | Yes | `EQUITY`, `OPTION`. Signals outside scope rejected `403` |
| `instrument_scope.underlying_allowlist` | No | Optional narrowing. Recommended — it makes the agent legible |
| `options_only_single_leg` | Yes | Phase one supports single-leg options only. Multi-leg is phase two |
| `frequency_band` | Yes | `LOW` <10/wk, `MEDIUM` <5/day, `HIGH` <20/day. Sets rate limits and listing eligibility |
| `publishes_outside_core_hours` | Yes | Drives TTL defaults and the deferred-notification disclosure |
| `default_proposal_ttl_seconds` | Yes | 60–3600. Per-signal overrides clamped to `max` |
| `exit_policy` | Yes | `CREATOR_SIGNALS_EXITS` or `SUBSCRIBER_MANAGES_EXITS`. Displayed prominently |
| `price_cents`, `billing_interval` | Yes | Creator sets price. Public does not set or cap it |
| `disclosure_block` | Yes | Creator-authored risk disclosure. Minimum length enforced, content not supplied by Public |
| `auth_mode` | Yes | `HMAC` or `PATH_TOKEN` |

## Why `exit_policy` is displayed prominently

Until bracket orders land, an exit requires the subscriber to confirm a second time. A subscriber who misses that confirmation is holding an unmanaged position. That is the single largest honest weakness in phase one and it must be disclosed at the point of subscription, not buried in terms.

`SUBSCRIBER_MANAGES_EXITS` agents are legitimate — a creator who only calls entries and expects subscribers to manage their own exits is a normal product. It just has to be labelled.

## Lifecycle

```
SANDBOX → PENDING_REVIEW → LIVE ⇄ PAUSED
                             ↓
                         SUSPENDED → RETIRED
```

**Sandbox** is mandatory and is a genuine feature, not a formality. The creator gets a real endpoint, real fan-out to simulated subscriptions, real proposal objects and real preflight, and no orders. They can fire their actual TradingView alert at it and see exactly what a subscriber would see.

Requirement to leave sandbox: at least 5 successfully parsed signals, at least one covering each declared instrument type, and zero validation errors in the most recent 3.

This is where OSI symbol formatting problems surface. Catching them in sandbox instead of in production with 400 live subscribers is the difference between a launch and an incident.

**Review** covers the manifest text, the disclosure block, and the declared scope. Reviewing creator-authored copy is required supervision. Rewriting it into Public's voice is the mistake that produced the 2024 enforcement actions against other brokers — review and reject, do not author.

**Suspension** is automatic on: hard rate ceiling breach, repeated signals outside declared scope, or a validation error rate above threshold. Automatic suspension needs a fast human appeal path, because a wrongly suspended creator with a paying audience is an urgent commercial problem.

## Changing a live manifest

| Change | Handling |
|---|---|
| `summary`, `disclosure_block` | Re-review, no subscriber notice |
| `price_cents` | Existing subscribers grandfathered until they cancel. New price for new subscriptions only |
| `instrument_scope` widening | Re-review plus notice to active subscribers |
| `frequency_band` increase | Re-review plus notice, and performance history annotated at the change point |
| `exit_policy` | Re-review plus **affirmative** subscriber acknowledgement, because it changes what they are responsible for |
| `name` | Allowed once per 90 days, history annotated |

Performance history must be annotated wherever the manifest materially changed. A creator whose strategy changed shape should not present a continuous track record as though it did not.

## What the creator gets that they cannot get elsewhere

Worth stating explicitly, because this is the recruiting pitch and it should drive what we build:

| | Whop / Discord | Collective2 | TradingView | **Public** |
|---|---|---|---|---|
| Checkout and billing | Yes | Yes | Yes | Yes |
| Execution in subscriber accounts | No | No | No | **Yes** |
| Platform-verified track record | No | Partial, on simulated accounts | No | **Yes, from real fills** |
| Fill quality and slippage data | No | No | No | **Yes** |
| Confirm-rate feedback | No | No | No | **Yes** |
| Take rate | 3–10%+ | 30–50% | 0% promotional | **0% founding, 15% steady** |

The two rows nobody else has any path to are verified performance from real fills and fill-quality feedback. A creator who can prove their signals were actually executable at the prices they called has something no Whop seller can produce. Build those well and the recruiting pitch writes itself.
