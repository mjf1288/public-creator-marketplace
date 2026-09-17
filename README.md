# Public Creator Marketplace — Specification

Owner: Matthew Foster, Director of API Trading
Status: **Draft for engineering review**
Target: engineering handoff in 2–3 weeks

---

## What this is

A marketplace inside Public where creators publish trading signals and subscribers execute them in their own Public accounts, with a human confirmation on every order.

Phase one deliberately requires **no new market data license and no discretionary authority**. Both are gates that need decisions above this project, and phase one is designed to ship without either.

## The one-paragraph architecture

A creator registers an **agent** — a named signal stream with a declared instrument scope, frequency band and disclosures. The creator's own charting tool (TradingView, TrendSpider) POSTs a signal to a per-agent Public endpoint. Public fans that one signal out to every active subscription, sizes each one using **the subscriber's own** size rule, runs preflight against that subscriber's account, and pushes a proposed order for the subscriber to confirm. Unconfirmed proposals **expire**. Nothing auto-executes. Public computes performance from actual subscriber fills, not from the creator's claimed entry.

## Why this clears both gates

| Gate | Why phase one clears it |
|---|---|
| **Market data licensing** | The signal originates in the creator's own charting tool. Public redistributes no data to the creator, and the creator receives no market data from us. Each subscriber's quotes are served under their existing Non-Professional entitlement, unchanged. |
| **Automated execution** | Every order is confirmed by the subscriber. Sizing is subscriber-set, never creator-set. Unconfirmed proposals expire rather than execute. The creator never sees subscriber accounts, positions or identities. |

## Documents

| File | What it covers | Status |
|---|---|---|
| `docs/00-prd.md` | Problem, scope, phasing, success metrics | Draft |
| `docs/01-signal-ingestion.md` | Webhook contract, auth, idempotency, rate limits | Draft |
| `docs/02-agent-manifest.md` | How a creator declares an agent | Draft |
| `docs/03-data-model.md` | Entities and the proposal state machine | Draft |
| `docs/04-order-mapping.md` | Signal intent → Public order API | Draft |
| `docs/05-confirmation-flow.md` | Subscriber UX, timeout and staleness semantics | Draft |
| `docs/06-performance-methodology.md` | Computing and displaying track records | Draft |
| `docs/07-compliance-controls.md` | Control → rule → engineering requirement | Draft |
| `docs/08-billing.md` | Subscriptions, take rate, payouts | Draft |
| `docs/09-metrics.md` | Instrumentation and success criteria | Draft |
| `docs/10-open-decisions.md` | What needs a decision and from whom | Draft |
| `schemas/` | JSON Schema for signal, manifest, proposal | Draft |

## Where this sits in the wider plan

The three paths put to Jannick are one pipeline, not alternatives:

- **Webhooks are the acquisition funnel.** Creators already pay third parties to bridge alerts into brokers. Meeting them at the tool they already use is the cheapest way in.
- **Signal = Agent is the wedge.** This repository. Public is the only venue where the signal, the funded account and the order live together, which is why Public can compute performance instead of the creator claiming it.
- **An AI-native script language is the moat.** Phase three, and it only matters once the wedge is proven.

The strategic prize is **creator-led account acquisition, not take rate.**

## Why creators should want to run here

| | Whop / Discord | Collective2 | TradingView | **Public** |
|---|---|---|---|---|
| Checkout and billing | Yes | Yes | Yes | Yes |
| Execution in subscriber accounts | No | No | No | **Yes** |
| Verified track record from real fills | No | Simulated only | No | **Yes** |
| Fill quality and confirm-rate diagnostics | No | No | No | **Yes** |
| Take rate | 3–10%+ | 30–50% | 0% promotional | **0% founding, 15% steady** |

The two rows nobody else has a path to are verified performance from real fills and fill-quality feedback. Build those well and the recruiting pitch writes itself. Build them badly and 15% is not defensible.

## Hard constraints — do not design around these

1. **No auto-execution in phase one.** Not as a setting, not as an opt-in, not for advanced users.
2. **The creator never sets position size.** A signal sized to a specific account is advice tailored to that person.
3. **Unconfirmed proposals expire.** Timeout must never fall through to submission.
4. **Identical signals to all subscribers at the same instant.** No early access, no priority tier.
5. **The creator sees aggregate counts only.** No subscriber identity, account value, positions or fills.
6. **Public never picks the instrument.** Phase one requires an exact symbol from the creator, including exact OSI option symbols.
7. **Performance is computed from subscriber fills.** Creator-supplied returns are never displayed.

Each constraint maps to a specific regulatory exposure. See `docs/07-compliance-controls.md`.

## Out of scope for phase one

- Futures (needs the commercial market data license — Gate 1)
- Auto-execution without per-order confirmation (Gate 2)
- Relative option contract selection, e.g. "30 delta, 45 DTE" (Public would be choosing the contract, which inserts our judgment into the recommendation)
- Creator-authored position sizing or risk management
- Performance leaderboards ranked by return
- The AI-native script language (phase three)
