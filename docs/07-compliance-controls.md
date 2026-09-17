# Compliance Control Matrix

Each control, the exposure it addresses, and the engineering requirement that implements it.

This document exists so Legal can review a specific list and eng can see that each constraint has a reason. **No control here should be relaxed without a written Legal sign-off referencing the specific row.**

Caveat to state plainly to Legal: no SEC or FINRA guidance directly addresses creator or agent marketplaces hosted by a broker-dealer. Every mapping below is analogical, drawn from the closest available authority. That is the honest posture and it argues for the conservative design, not against it.

---

## The two triggers everything else follows from

1. **Communication becomes tailored to an individual.** Impersonal, generally circulated advice for a fee is protected by the publisher's exclusion. Advice tailored to a specific person's situation is not.
2. **The arrangement gives someone effective power to trade without a per-order customer decision.** Not formal discretionary authority — effective power.

Every control below defends one or both.

---

## Control matrix

### C1 — No auto-execution in phase one

**Exposure.** In [In re Weiss Research](https://www.sec.gov/files/litigation/admin/2006/ia-2525.pdf) (IA-2525), a publisher whose subscribers' brokers executed its recommendations without pre-approval was found to have "effectively had investment discretion," lost the publisher's exclusion, and was ordered to pay $1,641,141 in disgorgement plus a $350,000 penalty. Weiss **received no fees from the broker-dealers** — no revenue share is required for liability. The SEC's own [auto-trading guidance](https://www.sec.gov/about/reports-publications/investorpubsautotradinghtm) states that newsletter publishers whose subscribers auto-trade are generally investment advisers. [15 U.S.C. 78c(a)(35)](https://www.law.cornell.edu/uscode/text/15/78c) reaches a person who decides what is bought or sold "even though some other person may have responsibility."

**Engineering requirement.** No state transition from `NOTIFIED` or `EXPIRED` to `SUBMITTED`. Enforced as a database constraint, not only application logic, with a test that attempts the illegal transition. No configuration flag, admin tool, or support path may bypass confirmation.

### C2 — Subscriber sets size, creator never does

**Exposure.** Position sizing against a specific account is advice tailored to that person, which defeats the publisher's exclusion. For futures the exposure is independent and stricter: [17 CFR 4.14(a)(9)](https://www.law.cornell.edu/cfr/text/17/4.14) requires that advice not be tailored to the recipient's situation and that the adviser not direct accounts. Position-aware sizing is a registration trigger even in pure alerting.

**Engineering requirement.** Signal ingestion rejects `quantity`, `amount`, `notional`, `risk_percent`, `percent_of_account`, `leverage`, `account_id`, `subscriber_id` with `422`. Rejection, not silent ignoring. Sizing fields are writable only by the authenticated subscriber on their own subscription; no creator, employee, or support path may write them.

### C3 — Timeout expires, never executes

**Exposure.** Fall-through execution on timeout converts inaction into consent, which is the substance of discretion regardless of how the terms are written.

**Engineering requirement.** As C1. Expiry is terminal.

### C4 — One confirmation per order, no batching

**Exposure.** "Approve all," standing consent, or session-level approval each grant effective power to trade without a per-order decision.

**Engineering requirement.** Confirmation is keyed to a single `proposal_id`. No endpoint accepts multiple proposal ids. No subscription field authorizes future proposals. The absence of `auto_confirm` is recorded in the data model deliberately so it is not added later as a convenience.

### C5 — No performance ranking, leaderboard, or derived badge

**Exposure.** [FINRA Rule 2210(d)(1)(F)](https://www.finra.org/rules-guidance/rulebooks/finra-rules/2210) restricts a member's communications from predicting or projecting performance. A return-ranked leaderboard published by the broker is the broker making a performance implication about a third party's future results.

**Engineering requirement.** No sort, filter default, badge, tier, score, or rating derived from return on any Public-authored surface. Sorting limited to recency, subscriber count, alphabetical, and creator-selected category. See `06-performance-methodology.md`.

### C6 — Creator authors their own content; Public reviews but does not write

**Exposure.** [FINRA Regulatory Notice 17-18](https://www.finra.org/rules-guidance/notices/17-18) sets out adoption and entanglement: a firm that becomes involved in preparing third-party content, or explicitly endorses it, takes on responsibility for it as its own communication. The 2024 enforcement actions on influencer arrangements landed on exactly this — **M1 Finance $850,000, Moomoo $750,000, TradeZero $250,000, Cobra Trading $200,000** — and **none of them involved third-party execution.** Entanglement alone was sufficient.

**Engineering requirement.** Creator-authored fields (`summary`, `disclosure_block`, signal `note`) are stored and rendered verbatim with creator attribution. No Public-generated summarization, rewriting, tone adjustment, or commentary on creator content anywhere in the product. Review workflow supports approve and reject only — no edit-in-place. No Public-supplied talking points or template copy for creators.

**This is the row most likely to be violated by a well-intentioned growth or content team**, because supplying creators with copy is standard marketing practice and here it is the specific fact pattern that produced fines.

### C7 — Flat revenue share, never turnover-linked

**Exposure.** Compensation tied to trading volume gives the creator a direct financial interest in subscriber turnover, creating a conflict that a broker-dealer hosting the arrangement owns. [FINRA Rule 2040](https://www.finra.org/rules-guidance/rulebooks/finra-rules/2040) governs payments to unregistered persons for securities activity.

**Engineering requirement.** Payouts computed from subscription revenue only. No field, rate, bonus, or tier in the payout model may reference order count, notional, commissions, or fill volume. Note that eToro's CopyTrader avoids much of this by **not paying US Popular Investors at all** — removing compensation entirely. A revenue-shared marketplace cannot borrow that defense, and Collective2's position works because Collective2 is not the broker. We are, so we carry this ourselves.

### C8 — Creator has no visibility into subscriber accounts

**Exposure.** Access to individual account information plus signal authorship moves the relationship toward an advisory one and creates tailoring risk, even without formal authority.

**Engineering requirement.** Creator analytics served from a read model with no join path to subscriber identity, account value, positions, or individual fills. All aggregates suppressed below 10 contributing subscriptions. Signal acceptance returns a count only, never a breakdown.

### C9 — Identical signals to all subscribers, simultaneously

**Exposure.** Differentiated delivery is tailoring. It is also a fairness problem: systematically earlier notification produces systematically better fills.

**Engineering requirement.** Single fan-out per signal with no tiering. Spread between first and last notification under 1s p99. No priority, premium, or early-access tier at any price.

### C10 — Supervision of creator communications

**Exposure.** [FINRA Rule 3110](https://www.finra.org/rules-guidance/rulebooks/finra-rules/3110) requires supervisory systems, and [Regulatory Notice 21-29](https://www.finra.org/rules-guidance/notices/21-29) addresses vendor oversight, including circumstances where vendor personnel performing member functions are treated as associated persons.

**Engineering requirement.** All creator-authored content — manifest text, disclosures, and every signal `note` — retained immutably with timestamps and reviewer identity. Notes are surfaced for post-review sampling with keyword flagging. Retention aligned to the firm's communications retention schedule.

### C11 — Public retains exclusive control of risk controls

**Exposure.** [Rule 15c3-5(d)](https://www.law.cornell.edu/cfr/text/17/240.15c3-5) requires a broker-dealer to have direct and exclusive control over its risk management controls.

**Engineering requirement.** All marketplace order flow passes existing Public risk controls unchanged. No creator-configurable or marketplace-specific bypass, elevated limit, or exemption. Subscriber guardrails are additive restrictions layered on top of firm controls, never relaxations of them.

### C12 — Registration gate for phase two

**Exposure.** Auto-execution is only available where the creator is a registered investment adviser or CTA, or operates under a registered entity we supply. Futures additionally require CTA registration and NFA membership.

**Engineering requirement.** `registration_status` on the creator record, phase one restricted to `NONE`. Any auto-execute capability gated on a verified non-`NONE` status, with verification against IAPD or NFA BASIC recorded and re-verified on a schedule. Not self-attested.

### C13 — Complete audit trail

**Exposure.** Both books-and-records obligations and the practical need to reconstruct, for any order, that a human confirmed it.

**Engineering requirement.** Immutable event log: signal received with raw body and signature, proposals created, guardrail evaluations and skips, notifications dispatched, confirmations with timestamp and device context, submissions with client order id, fills. Any displayed performance figure reconstructable from source fills.

### C14 — Terms amendment is a launch blocker

**Exposure.** Public's [Individual API Program terms](https://public.com/disclosures/individual-api-program) prohibit dissemination of market data to third parties and prohibit Applications that offer or promote services to third parties. The [Market Data Addendum](https://public.com/disclosures/public-market-data-addendum) makes commercial use incompatible with Non-Professional status. **A creator selling a product built on the Public API is out of compliance with our own terms on day one.**

**Engineering requirement.** None — this is a Legal deliverable. But it blocks launch, not build. Build proceeds; launch does not happen until the amendment is executed. Tracked in `10-open-decisions.md`.

---

## Supporting authority worth having on hand for the Legal conversation

- **Publisher's exclusion survives paid, customized email alerts.** The 2024 *Lingley v. Seeking Alpha* dismissal found filter features "merely allow the subscriber to filter generally available content" — the strongest available authority for scanner and alert tiers ([Katten analysis](https://katten.com/judge-dismisses-case-against-seeking-alpha-implications-for-publishers-of-financial-information)).
- **Signal generation without order generation is not an algorithmic trading strategy.** [FINRA Regulatory Notice 16-21](https://www.finra.org/rules-guidance/notices/16-21) carves out an algorithm that "solely generates trading ideas... but is not equipped to automatically generate orders."
- **The risk is not Reg BI.** It is advertising, entanglement, supervision and conflicts. Framing it as suitability sends Legal down the wrong path.

## Risk tiers, for reference

| Tier | What it is | Phase |
|---|---|---|
| 0 | Research, scanners, screeners | Phase 1 |
| 1 | Alerts and signals, no execution path | Phase 1 |
| 2 | Propose-and-confirm | **Phase 1 — this spec** |
| 3 | Auto-execute, unregistered creator | **Do not build with paid creators** |
| 4 | Auto-execute under a registered adviser | Phase 2, gated on C12 |

Tier 3 is listed to be explicit that it is excluded, not merely unbuilt. It is the Weiss fact pattern with a revenue share attached.
