# Open Decisions

What is blocked, on whom, and what it blocks. Nothing here stops the build; two things stop the launch.

---

## Launch blockers

### D1 — Amend Public's API terms

**Owner:** Legal, with Jannick. **Blocks:** launch, not build.

Public's [Individual API Program terms](https://public.com/disclosures/individual-api-program) prohibit dissemination of market data to third parties and prohibit Applications that offer or promote services to third parties. The [Market Data Addendum](https://public.com/disclosures/public-market-data-addendum) makes commercial use incompatible with Non-Professional status.

A creator selling a subscription product built on the Public API violates our own terms on day one. This is not an exchange licensing negotiation — it is our own document, which makes it the cheapest blocker on this list to clear and the one most likely to be forgotten because it does not look like a deal.

What is needed: a commercial API tier, or a creator-specific addendum, permitting a creator to operate a paid product on the API and to receive data for commercial purposes.

### D2 — The execution boundary

**Owner:** Jannick. **Recommendation: launch propose-and-confirm only.**

Selling signals is defensible under the publisher's exclusion. Auto-trading on behalf of paying subscribers is the [Weiss Research](https://www.sec.gov/files/litigation/admin/2006/ia-2525.pdf) fact pattern, where a publisher whose subscribers' brokers executed without pre-approval was found to have effectively held investment discretion — with **no revenue share from the brokers required** for liability.

This spec assumes the recommendation is accepted. If auto-execution is required at launch, most of `05-confirmation-flow.md` and `07-compliance-controls.md` change and creators must be registered advisers, which shrinks the founding cohort to almost nobody.

**This decision has effectively been made by building to it.** It should be confirmed explicitly rather than by default.

---

## Blocks the futures phase, not phase one

### D3 — Commercial market data

**Owner:** Jannick, with Legal and Finance. **Blocks:** phase 3, and Danielle's partner pipeline today.

Public currently solves market data as a bespoke legal negotiation per partner. That works at fifteen partners and cannot work for a self-serve marketplace, and it structurally excludes futures, which is the highest-ARPU segment and the segment agtrader needs a commercially licensed feed for.

| Option | Shape | Cost |
|---|---|---|
| A — Creators ship code, not data | Creator's logic runs against Public's data in the subscriber's own entitlement. NinjaTrader pattern; SignalStack proves it at production scale | ~$0 |
| B — Public as licensed redistributor | Full OPRA/CTA/UTP redistribution stack. Public already absorbs most of it as a broker | **Marginal ~$10–15k/mo** on a ~$29k/mo gross stack |
| C — Third-party vendor | Databento Plus $1,750/mo plus pass-throughs | ~$4,900/mo futures-only, ~$8,400/mo futures+options |

**Option A covers phase one** and should be the stated position, so market data does not become a launch dependency it does not need to be.

Option B is the durable moat. No comparable broker publishes a third-party redistribution right — IBKR, Schwab, Tradier, Alpaca and Tradovate are all per-account or per-seat. Being the only brokerage where a creator can build a commercial product without negotiating their own exchange licenses is a structural advantage, and at a marginal $10–15k/month it is cheap relative to what it unlocks.

**The real futures blocker is not price.** CME §6, "Creation of Other Derivative Works," is priced "available upon request" at CME's sole discretion ([2026 Derived Data Fees](https://www.cmegroup.com/market-data/files/2026-derived-data-fees.pdf)). A signal derived from CME data and sold to third parties is a derivative work. Contact is `CMEGroupDerivedData@cmegroup.com`, and this has the longest lead time of anything on this page — **start the conversation now even though futures is phase 3**, because the answer takes months and we cannot scope the phase without it.

Published CME rates for reference: $29,280/yr real-time distribution, $610/mo feed, $609/mo Non-Display C-2 ([January 2026 fee list](https://www.cmegroup.com/market-data/files/january-2026-market-data-fee-list.pdf)).

---

## Commercial decisions

| # | Decision | Owner | Status |
|---|---|---|---|
| D4 | Blended CAC, to compare against marketplace CAC | VP Brand Growth | **Outstanding.** Needed to set the steady-state take rate defensibly |
| D5 | Founding-cohort 0% permanent, not promotional | Jannick / Finance | Proposed |
| D6 | Steady-state ~15% | Jannick / Finance | **Matthew's proposal only** |
| D7 | Whether Public pays creators for promotion | Legal / Brand Growth | **Not started.** Paying for promotion is a different arrangement from hosting a product, and it is the arrangement that produced the 2024 fines. Recommend not doing it in phase one |
| D8 | Revenue recognition structure and 1099 treatment | Finance | Not started |
| D9 | Refund policy and who absorbs it | Finance / Support | Not started |

---

## Product decisions inside our own scope

| # | Decision | Recommendation |
|---|---|---|
| D10 | Bracket / OCO / OTO in phase 1.5 | **Yes, highest priority after launch.** An exit that requires confirmation is not a stop. Removes the largest honest weakness in phase one |
| D11 | Relative option selection ("30 delta, 45 DTE") | Defer to phase 2. If Public resolves criteria into a contract, Public exercises judgment inside the recommendation |
| D12 | Multi-leg options | Phase 2. Confirming a spread as one decision is fine; the order plumbing is the work |
| D13 | Crypto in phase 1.5 | Yes. 24/7 makes the overnight notification problem central rather than an edge case, which is worth solving deliberately |
| D14 | Path-token auth for TradingView | **Required, not optional.** TradingView cannot compute an HMAC and it is the tool our target cohort uses. Mitigate with rotation, IP logging, tighter rate limits, and required signing above a subscriber threshold |
| D15 | Exclusivity from creators | **No.** Allow dual-listing with Whop and Discord. Demanding exclusivity from an established business loses the founding cohort |
| D16 | Sandbox mandatory before live | Yes. It is where OSI formatting errors surface |

---

## Recruiting the founding cohort

Not a decision so much as a constraint that should be written down, because it is where the plan is most likely to go wrong.

Propose-and-confirm has a hard ceiling on signal frequency. Nobody confirms a 0DTE entry fast enough for the fill to resemble the signal.

| Cohort | Frequency | Fit |
|---|---|---|
| Swing / position equities | 2–10/week | **Target** |
| Options income and spreads | 2–8/week | Good, pending multi-leg |
| Intraday equity | 5–20/day | Marginal |
| SPX 0DTE rooms | 10–40/day | **Wrong for phase one** — the largest segment in the market map and the worst fit |
| thinkorswim script sellers | n/a | Phase 3. No webhook path exists |

The founding cohort must be recruited from TradingView-based swing and position creators. Recruiting from the largest and most visible segment would produce a launch where confirm rate collapses and the model looks broken when it is only mismatched.

**Follow-up on Danielle's creator interviews:** the instinct is right and the cohort is wrong — script creators have no webhook path. Redirected at TradingView-based swing and position creators, the interviews should produce three specific spec inputs: signals published per week, willingness to accept per-order subscriber confirmation, and what it would take to move billing off Whop. Those answers change this document. Open-ended workflow interviews do not.
