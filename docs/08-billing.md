# Billing, Take Rate and Payouts

The commercial layer. Status: the take rate below is Matthew's proposal, not an approved number.

---

## Strategic position

**The prize is creator-led account acquisition, not marketplace revenue.** A creator with 400 subscribers who brings 100 funded accounts is worth far more than 15% of their subscription revenue. Every decision here should be read through that lens: take rate is a recruiting instrument, not a revenue line.

Sizing from the plan given to Jannick: roughly 15 creators × roughly 400 subscribers ≈ 6,000 traders, converting to 1,500–2,100 funded accounts at 25–35% activation. Those activation rates are assumptions and blended CAC is still outstanding from Brand Growth — the comparison that decides how aggressive the take rate should be is marketplace CAC against paid CAC, and we cannot complete it yet.

## Take rate

| Cohort | Rate |
|---|---|
| Founding cohort (first ~15 creators) | **0%, permanent for those creators** |
| Steady state | **~15%** |

Competitive anchors: TradingView 0% (promotional through Oct 1 2026), Whop 3%+, Discord 10% via a 90/10 split, Substack 10%+, Darwinex 20% performance / 5% management, Collective2 30–50%. Schwab's thinkorswim gives script sellers no store, no checkout and no revenue share at all.

15% sits above Whop and level with Discord, and is justified only if execution, verified performance and fill diagnostics are visibly worth the difference. If those features are weak, 15% is not defensible and we should not pretend otherwise.

**Making founding-cohort 0% permanent rather than promotional is deliberate.** A creator being asked to move their business to an unlaunched marketplace is taking real risk, and a rate that expires converts them into a churn problem in twelve months. Permanent 0% for fifteen creators costs little and buys the reference customers everything else depends on.

## Why creators should move billing here

The honest answer is that most will not move immediately, and the spec should assume dual-listing rather than exclusivity.

| | Whop / Discord | Public |
|---|---|---|
| Take rate | 3–10%+ | 0% founding, 15% steady |
| Execution in subscriber accounts | No | Yes |
| Verified track record from real fills | No | Yes |
| Fill quality and confirm-rate diagnostics | No | Yes |
| Subscriber must fund a brokerage account | No | **Yes** |

That last row is the real friction and it cuts against us. A Whop subscriber needs a credit card; a Public subscriber needs a funded brokerage account. That is a materially higher bar and it is why the acquisition value is high — but it also means a creator moving here will lose some portion of their existing audience in transit.

**Do not require exclusivity.** A creator should be able to run their Discord and list here simultaneously. Demanding exclusivity from someone with an established business is the fastest way to lose the founding cohort, and dual-listing costs us nothing since the subscribers who convert here are the ones who wanted execution.

## Billing mechanics

| Element | Decision |
|---|---|
| Currency | USD, phase one |
| Interval | Monthly. Annual is a should-have |
| Trial | Creator-configurable, 0–14 days |
| Proration | On upgrade; no refund on downgrade, applies next period |
| Failed payment | 3 retries over 7 days, subscription `PAST_DUE`, signals not delivered while past due |
| Cancellation | Immediate stop of new proposals; access through the paid period |
| Refunds | Public-administered under a published policy. Not creator-discretionary |

**Signals must not be delivered while a subscription is `PAST_DUE`.** Delivering to an unpaid subscriber creates an unpriced relationship, and delivering to some and not others breaks the identical-delivery control in C9.

Price changes grandfather existing subscribers until they cancel. A creator raising prices on an existing base is the fastest route to complaints that land on Public, since we are the one billing.

## Payouts

| Element | Decision |
|---|---|
| Basis | Subscription revenue collected, net of payment processing and take rate |
| Schedule | Monthly, target by the 5th for the prior month |
| Holdback | 14-day rolling reserve against chargebacks |
| Minimum | $25, rolls forward |
| Tax | W-9 / W-8BEN collected at onboarding, 1099 issued |
| Statement | Per-agent revenue, subscriber counts, churn, deductions itemized |

Payout timing is a real competitive lever. Discord pays monthly on or before the 15th. Targeting the 5th is a concrete, easily-communicated advantage that costs us only operational discipline, and creators talk to each other about getting paid.

## The hard constraint

**No component of creator compensation may reference trading activity.** Not order count, not notional, not commissions, not fill volume, not a volume-tiered bonus, not a "most active agent" award. Revenue share on subscription revenue only. See C7 — this is a conflicts issue that Public owns as the broker, and it is the constraint most likely to be eroded by a well-meaning incentive program.

## Open commercial questions

These need decisions from outside this spec, tracked in `10-open-decisions.md`:

1. Blended CAC from Brand Growth, to complete the marketplace-CAC comparison
2. Whether founding-cohort 0% is approved as permanent
3. Whether Public funds any creator acquisition directly, and if so how that interacts with C6 — paying a creator for promotion is a different arrangement from hosting their product, and it is the arrangement that produced the 2024 fines
4. Refund policy authority and who absorbs the cost
5. Whether subscription revenue is Public revenue with a creator payout, or creator revenue with a Public fee — this is an accounting and tax structure question with different 1099 consequences
