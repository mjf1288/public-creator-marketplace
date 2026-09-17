# Metrics and Instrumentation

---

## North star

**Funded accounts activated through the marketplace, trading at 30 days.**

Not GMV, not subscription revenue, not signal volume. The strategic case is creator-led account acquisition, so the metric is funded accounts that are still trading a month later. A marketplace with high subscription revenue and no funded-account activation has failed at its actual purpose.

## Metric hierarchy

### L1 — the one number

| Metric | Definition | Phase-one target |
|---|---|---|
| Activated marketplace accounts | New funded accounts attributed to a creator listing, with ≥1 confirmed fill, still active at 30 days | Directional; no target until CAC comparison is complete |

### L2 — the drivers

| Metric | Definition | Target | Why |
|---|---|---|---|
| **Confirm rate** | Confirmed ÷ notified | **>60%** | The product's viability test. Below ~40% propose-and-confirm does not work as a model and the frequency thesis is wrong |
| Time to confirm, median | Notification → confirmation | **<90s** | Drives slippage and therefore whether verified performance looks good |
| Subscriber activation | Subscribers with ≥1 confirmed fill within 7 days of subscribing | >70% | A subscriber who never confirms churns |
| Creator retention | Founding cohort still publishing at 90 days | >80% | Fifteen creators is a small enough base that losing three is a crisis |
| Subscriber 30-day retention | Still subscribed at 30 days | >65% | |

### L3 — diagnostics

**Ingestion:** signals received, validation error rate by error type, auth failures by mode, rate-limit events, duplicate suppressions.

**Fan-out:** signal-to-first-notification p50/p99, spread first-to-last p99, proposals per signal, skip rate by reason.

**Execution:** submission success rate, rejection rate by reason, fill rate, slippage vs signal reference (median and p90), partial fill rate, staleness re-confirmation rate.

**Exits:** share of entries with a confirmed exit, share closed manually, share still open past the creator's exit signal. **This is the most important diagnostic in phase one** — it directly measures the unmanaged-position risk created by not having brackets, and it is the number that justifies prioritizing the bracket rollout.

**Creator health:** signals per week vs declared band, confirm rate by creator, expiry rate by time of day, scope violations, suspension events.

**Commercial:** subscriptions started and cancelled, MRR, involuntary churn from failed payments, payout volume.

## Alert thresholds

| Condition | Severity |
|---|---|
| Any `EXPIRED → SUBMITTED` transition attempt | **Critical.** Should be structurally impossible; a single occurrence is an incident |
| Duplicate broker order for one proposal | **Critical** |
| Fan-out p99 > 5s | High |
| First-to-last notification spread p99 > 3s | High — becomes a fairness problem |
| Auth failure rate > 5% for one agent | Medium — usually a creator whose token rotated |
| Confirm rate for an agent < 25% over 7 days | Medium — creator conversation, not enforcement |
| Signals dropped without a recorded terminal state | **Critical** |

The first and last rows are the ones to page on. A silently dropped signal is worse than a visible failure, because the creator believes their subscribers were notified.

## Instrumentation requirements

Every proposal state transition emits an event with `proposal_id`, `signal_id`, `agent_id`, `subscription_id`, from-state, to-state, reason, and timestamp. The state machine is the event stream; do not build a parallel analytics path that can disagree with it.

Latency measured from signal receipt at the edge, not from internal queue entry. The creator's experience starts when they fire the webhook.

Slippage always measured against the signal's `reference_quote` captured at receipt, never against a later quote.

## What is deliberately not measured, or not surfaced

- **No per-subscriber metric visible to creators.** Aggregates only, suppressed below 10 contributing subscriptions
- **No creator leaderboard on any internal dashboard either.** An internal ranking becomes an external one — it gets screenshotted, shared with creators, and quoted in recruiting decks. Rank internally on confirm rate or retention if ranking is needed; never on return
- **No conversion metric that rewards confirmation.** Confirm rate is a product-health diagnostic, and it must never become a target that anyone is incentivized to lift through UX pressure. If an experiment proposes lifting confirm rate by adding urgency, the answer is no; see C4 and the confirmation-flow constraints

That last point is worth naming explicitly for whoever owns growth on this surface. Confirm rate is simultaneously our most important health metric and the one most dangerous to optimize directly.

## Phase-one review gate

Before phase 1.5, ten sessions of real production data reviewed against:

1. Confirm rate above 40% — below this, propose-and-confirm does not work and the cohort thesis needs revisiting
2. Median time to confirm under 3 minutes
3. Zero critical alerts
4. Share of entries reaching a confirmed exit above 70%, or the bracket rollout moves ahead of everything else
5. Slippage vs reference within a band we can publish without embarrassment

Failing 1 or 4 means changing the product, not scaling it.
