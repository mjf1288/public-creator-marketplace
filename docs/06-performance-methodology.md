# Performance Methodology

The single feature no competitor can copy, and the one most likely to be built wrong.

---

## The principle

**Performance is computed from actual subscriber fills. Creator-supplied returns are never displayed anywhere.**

On Whop and Discord, creators claim their own results and buyers have no way to verify, so rational buyers discount every claim and price on marketing instead. Because the signal, the funded account and the order all sit with us, we can publish what subscribers actually got, net of fees.

That is simultaneously the strongest recruiting pitch to good creators, the strongest trust signal to subscribers, and a compliance asset — a verified record is defensible in a way that a creator's screenshot is not.

It also means bad creators will not want to list here. That is the intended effect.

## Round-trip construction

The unit of performance is a **closed round trip** per subscription, matched on `position_ref`.

1. Entry: confirmed proposal that filled, wholly or partly
2. Exit: whatever actually closed the position — a confirmed exit proposal, a subscriber's manual close, or an option expiring
3. Return: `(exit proceeds − entry cost) ÷ entry cost`, **net of all fees and commissions on both legs**
4. Still open at reporting time: excluded from closed-trip statistics and reported separately as open exposure

Partial fills are handled at the filled quantity. A subscriber who filled 2 of 3 shares has a valid round trip on 2.

Manual closes are included. Excluding them would let a creator's record ignore subscribers who bailed out, which is exactly the selection bias the whole methodology exists to prevent.

## What is displayed

For each agent, subject to the minimum sample thresholds:

| Statistic | Definition |
|---|---|
| Closed round trips | Count, and the period covered |
| Median return per round trip | Across all subscriptions |
| Interquartile range | 25th–75th percentile of subscriber returns |
| Share of round trips profitable | Not called a "win rate" |
| Median holding period | |
| Subscribers with at least one closed trip | Denominator transparency |
| Confirm rate | Confirmed ÷ notified |
| Median time to confirm | |
| Median slippage vs signal reference | Basis points |
| Open positions | Count, excluded from closed statistics |

## Dispersion is mandatory, not optional

**A single headline return number must never appear without its dispersion.** Different subscribers confirmed at different times, at different sizes, and got different fills. A median with no spread implies a precision that does not exist, and the spread is real information: an agent with a tight band is more reliably followable than one with a wide band, even at a lower median.

Design the listing around median plus interquartile band as one inseparable unit. If a surface cannot fit the band, it cannot show the median either.

## Minimum sample before any performance is shown

All of the following must be satisfied:

- ≥ 20 closed round trips
- ≥ 30 days since the agent went live
- ≥ 10 subscriptions with at least one closed round trip

Below the threshold, display "Not enough history yet" with the counts so far. Do not show a partial or provisional figure, and do not let a creator opt into showing one.

Ten subscriptions is also the aggregation floor that prevents a creator inferring individual subscriber behavior from an average.

## No ranking by return. Anywhere.

Listing surfaces may sort by: recency, subscriber count, alphabetical, creator-selected category, or subscriber-set filters. **Never** by return, and no derived badge, tier, score, star rating, "top performer" label, or default ordering that correlates with return.

This is not conservatism. FINRA Rule 2210(d)(1)(F) restricts a broker-dealer's communications from projecting or predicting performance, and a return-ranked leaderboard published by the broker is the broker communicating a performance implication. It is also the single feature most likely to be requested by Growth, because leaderboards convert. It has to be refused at the design stage, not litigated later.

Subscribers may sort their own filtered views by historical statistics they choose. The distinction is that Public is not publishing a ranking.

## Required context on every performance display

- The exact period covered
- Number of round trips and number of contributing subscriptions
- "Net of fees and commissions"
- "Past performance does not indicate future results"
- Annotation markers wherever the agent's manifest materially changed
- Explicit note that returns reflect actual subscriber fills, not the creator's signal prices

The last line is the differentiator stated plainly, and it is worth prominence.

## Never displayed

- Annualized or extrapolated returns from a short sample
- Projections or targets of any kind
- Hypothetical or backtested results
- Creator-supplied figures or screenshots
- Performance of an agent below sample threshold
- Any per-subscriber result, to anyone other than that subscriber
- Any comparison of one creator to another authored by Public

## What the creator sees

The same aggregates, plus diagnostics they cannot get anywhere else:

| Metric | Why it matters to them |
|---|---|
| Confirm rate, and trend | Whether their frequency is sustainable for their audience |
| Median time to confirm | How much slippage their signal design is causing |
| Median slippage vs reference | Whether their called prices are actually executable |
| Skip reasons, aggregated | Whether they are calling trades their audience cannot afford |
| Expiry rate by time of day | Whether they publish when their audience is present |

This is a real product for creators, not a compliance byproduct. A creator who learns their 9:31am signals get a 40% confirm rate while their 3pm signals get 75% will change their behavior, and their subscribers will do better. No other platform can tell them this because no other platform sees the orders.

Suppress every metric below the 10-subscription aggregation floor.

## Storage and integrity

Performance is derived from fills, and fills are immutable. Recompute on a schedule rather than incrementally mutating a stored figure, so any displayed number can be reproduced from source fills at audit time.

Retain the derivation: which fills contributed to which round trip, and which round trips contributed to a displayed statistic. If a regulator, a creator, or a subscriber asks how a number was produced, the answer must be reconstructable from records, not from a cache.
