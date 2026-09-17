# PRD — Public Creator Marketplace, Phase One

Owner: Matthew Foster, API Trading
Status: Draft for engineering review

---

## Problem

Creators who sell trading signals have a checkout and an audience, and no way to execute. Every monetization rail available to them — TradingView, Whop, Discord, Substack — can bill a subscriber but cannot place an order. Creators who want execution bolt on a third-party webhook bridge, and the subscriber ends up paying twice: once for the signal, once for the plumbing.

For Public, that means the signal, the funded account and the order live in three different places, and none of them are ours.

## Why Public specifically

We are the only participant that can hold all three. That produces one thing no competitor can match: **we can compute a creator's track record instead of taking their word for it.** On Whop and Discord, buyers price on marketing because verification does not exist. That is a product differentiator and a compliance asset at the same time.

The prize is not subscription take rate. It is creator-led acquisition of self-selected active options traders.

## Phase one scope

**In:**
- Creator agent registration and a per-agent signal endpoint
- Signal ingestion over authenticated webhook
- Fan-out to subscriptions with subscriber-set sizing
- Server-side preflight per proposed order
- Propose-and-confirm execution on equities and options
- Subscription billing with a configurable take rate
- A listing surface with computed, fill-based performance
- Creator payouts

**Out:** futures, auto-execution, relative option selection, creator-set sizing, return-ranked leaderboards, the script language.

## Users

**Creator.** Has an existing paid audience and a strategy that already fires alerts from a charting tool. Wants distribution, billing and credibility without building infrastructure. Phase one target is swing and position traders, not intraday scalpers — see the frequency constraint below.

**Subscriber.** Already trades. Wants a specific creator's signals landing in their own account without copying and pasting into an order ticket. Keeps control of size and every individual decision.

## The frequency constraint, and what it means for recruiting

Propose-and-confirm has a hard ceiling on signal frequency. A creator firing twenty times a day produces twenty confirmation prompts, and confirm rate collapses. Once subscribers stop confirming, the product is broken regardless of signal quality.

This is a product constraint with a direct recruiting consequence:

| Cohort | Typical frequency | Phase one fit |
|---|---|---|
| Swing / position traders | 2–10 signals per week | **Target cohort** |
| Options income / spread sellers | Several per week | **Good fit** |
| Intraday equity traders | 5–20 per day | Marginal |
| SPX 0DTE alert rooms | 10–40 per day, seconds matter | **Wrong for phase one** |
| thinkorswim script sellers | n/a — no webhook path | Phase three |

The 0DTE rooms are the largest segment in our external market map and the worst phase-one fit: no human confirms a 0DTE entry fast enough to get a fill resembling the creator's. Recruiting has to start with lower-frequency creators or the launch metrics will look like a failure of the product rather than a mismatch of cohort.

Agents declare a frequency band at registration and are rate-limited to it. Exceeding it degrades the agent's listing status rather than silently dropping signals.

## Requirements

### Must have
- Creator registers an agent with instrument scope, frequency band and disclosures
- Per-agent signal endpoint with HMAC request signing and replay protection
- Signal deduplication by creator-supplied idempotency key
- Fan-out to all active subscriptions within a bounded latency budget
- Per-subscription sizing from the subscriber's own rule; creator input rejected
- Preflight per proposal, showing estimated cost, fees and buying-power impact
- Push notification and an in-app confirm surface showing complete order terms
- Proposal expiry with no fall-through to submission
- Order submission on confirmation, mapped to the existing Public order API
- Fill capture and performance computation from fills
- Subscription billing, take-rate accounting and creator payouts
- Creator-facing analytics limited to aggregates
- Full audit trail: signal received, proposals created, notified, confirmed, submitted, filled

### Should have
- Subscriber-level guardrails: max open positions, daily order cap, per-order notional ceiling, instrument allowlist
- Re-preflight at confirmation time with a staleness check
- Creator agent sandbox for testing before going live
- Exit signals referencing an earlier entry, so a creator can close what they opened

### Could have
- Bracket submission once OCO/OTO/bracket rollout lands, replacing separate exit signals
- Crypto instruments (no market data licensing constraint, but overnight confirmation is a real UX problem)
- Subscriber-configurable notification quiet hours

### Won't have in phase one
- Auto-execution
- Creator-set sizing or risk parameters
- Public selecting an option contract from creator-specified criteria
- Any ranking, badge, or sort ordering derived from returns

## Success metrics

Primary metric is **confirm rate**, because it is the leading indicator of whether propose-and-confirm is viable at all.

| Metric | Definition | Phase one target |
|---|---|---|
| Confirm rate | Confirmed ÷ notified proposals | > 60% |
| Time to confirm | Median notified → confirmed | < 90 seconds |
| Slippage vs signal reference | Subscriber fill vs price at signal receipt | Measured, not targeted |
| Funded account conversion | New funded accounts ÷ subscribers acquired | 25–35% (assumption, unvalidated) |
| Agent retention | Agents still publishing at day 60 | > 70% |
| Subscriber retention | Subscriptions active at day 60 | > 50% |

The funded-account conversion range is an assumption carried from the market sizing, not a measured rate. Blended CAC is still outstanding from Brand Growth and is required before this number is used to justify spend.

## Sizing

A founding cohort of roughly 15 creators averaging about 400 paid subscribers is approximately 6,000 self-selected active traders. At 25–35% funded-account activation that is 1,500–2,100 new funded accounts. Both the cohort size and the activation rate are assumptions.

## Dependencies

| Dependency | Owner | Blocking? |
|---|---|---|
| Terms amendment to the Individual API Program | Legal | **Yes** — current terms prohibit commercial applications built on the API |
| Execution boundary review | Legal / Compliance | **Yes** for launch, not for build |
| Take rate approval | Jannick | No — billing is configurable |
| Blended CAC | Brand Growth | No — needed for the business case, not the build |
| OCO/OTO/bracket rollout | Product | No — phase one uses separate exit signals |
| Commercial market data license | Unassigned | No — futures is out of scope |

## Phasing

**Phase 1 — Propose and confirm.** Equities and options. Everything in this document.
**Phase 1.5 — Crypto.** No licensing constraint. Needs an answer on overnight confirmation.
**Phase 2 — Auto-execution, gated.** Only where the creator is a registered adviser or operates under a Public-supplied adviser entity. Requires Gate 2 resolved.
**Phase 3 — Futures and the script language.** Requires Gate 1 resolved. Opens the thinkorswim script-seller cohort, the most under-served segment in the market.
