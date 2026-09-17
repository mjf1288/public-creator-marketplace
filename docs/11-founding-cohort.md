# Founding Cohort — Recruiting List

**Owner:** Matthew Foster, Director of API Trading
**Target:** 15 creators live in phase one
**Sources:** `creator_market_map.md` (~85 creators), `internal_creator_pipeline.md` (~46 builders), plus a dedicated swing/position research pass, September 17 2026
**Constraint driving everything:** propose-and-confirm has a hard ceiling on signal frequency

---

## Three corrections to what we told Jannick

Verification changed the picture. Taking these first because two of them were my own top-ranked names.

**1. Dave Mabe is not a creator. He is a competitor to the bridge layer.** He was my strongest Tier 1 target. He sells MabeKit at **$495 / $1,000 / $5,000 per month**, software that "can send trades through DAS and Interactive Brokers" ([davemabe.com](https://davemabe.com/)). He does not sell signals. He sells the exact plumbing we are proposing to give away at 0%. He belongs in a partner-or-compete conversation, not the founding cohort. His IBKR content relationship is real, and it is a distribution relationship for a tool vendor, which reads very differently now.

**2. Collective2's auto-trading population is tiny.** I proposed recruiting from C2's publisher base as a volume multiplier. C2 publishes live AutoTrader counts, and the largest one found is **33** ([US Stock Momentum](https://collective2.com/details-list/151894080)), followed by 17, 7, 2, 1, 1. Total verified across the leaderboard: about 61 accounts. C2 is still worth recruiting from, but for **proven behaviour, not volume** — these are the only people in the entire market verified to already route third-party signals into their own brokerage account. That is a different and narrower argument than the one I made.

**3. Nobody is confirmed to be paying for a signal-to-broker bridge.** The "we remove a line item you already pay for" pitch has no evidence behind it. Not one creator in the set was confirmed as a paying TradersPost, SignalStack, PickMyTrade or Alertatron customer. TradersPost's own signal-source page names only platforms, no creators ([traderspost.io/signals](https://traderspost.io/signals)). **Drop that pitch.** The accurate pitch is the opposite and stronger: most of these creators have no automation layer at all. They publish to Discord, Telegram, email or Substack, and their subscribers place every order by hand.

Also out on verification: **Chart Observer** is a crypto paper-trading simulator with no trade calls ([TradingView profile](https://in.tradingview.com/u/Chart0bserver/)), and **TradeVolatility** is a 0DTE-oriented data platform selling levels and exports, not entry/exit calls ([platform](https://stocks.tradingvolatility.net/)). Both were on my warm list. Both are wrong for this cohort.

---

## Why swing and position, not the loud part of the market

Phase one requires the subscriber to confirm every order. That works when a signal's shelf life is measured in hours and breaks when it is measured in seconds.

| Cohort | Signals | Fit | Why |
|---|---|---|---|
| Options income — wheel, CSPs, credit spreads at 30–45 DTE | 2–15 per week, often batched | **Best fit in the market** | A signal at 30 DTE does not care about a 15-minute confirmation. Highest-tolerance cohort there is |
| Swing / position equities | 2–10 per week | **Target** | Multi-day holds absorb confirmation delay |
| Signal-based allocation / rotation | under 1 per week | Good, low volume | Very high confirm tolerance, few orders per subscriber |
| Intraday equity | 5–20 per day | Marginal | Confirm rate sags, slippage becomes visible |
| SPX / SPY 0DTE | 10–40 per day | **Disqualified** | No confirmation is fast enough for the fill to resemble the signal |
| Futures | any | **Disqualified** | No futures type on the order API, CME derived-data question open |
| thinkorswim script sellers | n/a | **Disqualified** | No webhook path. Phase 3 |

**The trap:** the 0DTE segment is the largest and loudest part of this market — 18 vendors in section A2 of our own market map — and the worst possible phase-one cohort. It is where the visible audiences are, which means the obvious recruiting targets are precisely the ones that would make confirm rate collapse and the model look broken when it was only mismatched.

**The subtler trap:** the map's ~47 TradingView indicator authors are not candidates either, regardless of reach. LonesomeTheBlue has 207,300 followers and sells a chart tool. He never publishes "buy AAPL at 227.50." No trade call means nothing to confirm. Of the ~85 creators in the map we gave Jannick, only six survived the frequency filter — which is why this list is mostly freshly sourced.

## Scoring model

**Hard disqualifiers:** 0DTE as the core product · futures-only · day-trading frequency · thinkorswim-only delivery · no published entry *and* exit prices.

| Dimension | Weight | High score |
|---|---|---|
| Frequency fit | 30 | 2–10 signals/week, multi-day or multi-week holds |
| Webhook readiness | 25 | Already fires alerts or already routes to a broker |
| Audience | 20 | Paying subscribers convertible to funded accounts |
| Warmth | 15 | Already on the Public API or in our pipeline |
| Migration friction | 10 | Unmonetized, monthly billing, or dual-listing is easy |

Webhook readiness is weighted nearly as heavily as frequency on purpose. A creator whose signals already reach a brokerage account has proven their half of the technical premise. A creator who has never automated anything is a support project, however good the calls are.

---

## Table A — Design partners (4 slots)

Scored on integration readiness and tolerance for breakage, not audience. These four exist to break the product before anyone's subscribers are watching.

| Rank | Creator | Why | Contact |
|---|---|---|---|
| 1 | **Jay Allen** | Built his own Claude MCP integration on the Public API unprompted and executed single- and multi-leg orders through it. Technical, warm, no audience at risk. First creator through the pipe | Internal — API pipeline |
| 2 | **JasonL_Capital** | Connected a Grok bot to Public via MCP for quotes, chains and Greeks. Warm as of Aug 31 2026 (DM D0AA6NAPLL9) | X / Slack DM |
| 3 | **Arvind** (acct 5OG52150) | Multi-leg credit strategies with his own dev team — engineering capacity to build against a spec. **Fix the multi-leg credit replace / `limitPrice` sign bug before asking** | Internal — acct 5OG52150 |
| 4 | **MAR1 QUANT** (EstoTrader, Collective2) | The single most spec-aligned creator found anywhere. $20/mo, **15.3-day average hold**, SPY/QQQ/IWM options, 1 live AutoTrader, and he already sends signals "with enough time to execute the strategy in manual mode" — that is propose-and-confirm, described by a creator, unprompted ([C2 detail](https://collective2.com/details-list/139591006)) | C2 strategy page |
| Alt | **Leon Gaban** (acct 5OP53560) | Right profile, wrong plumbing — 4-leg SPXW needs multi-leg. Also blocked: fill rate collapsed after mid-June, "list open orders" endpoint missing. **Fix, then recruit** | Internal — acct 5OP53560 |
| Alt | **Robert Render** (Zonktrader) | Only HubSpot deal reading as a tool builder rather than a retail lead. Stage: Qualified Lead | HubSpot |

## Table B — Options income (3 slots)

The highest-tolerance cohort in the market, and the reason to prioritise single-leg options in phase one.

| Rank | Creator | Price | Audience | Frequency / hold | Score | Why |
|---|---|---|---|---|---|---|
| 1 | **Options Income with The Wheel Strategy** | n.a. — Substack paid price not public | **9,000+ subscribers** | "Every Monday, I publish 15 fully structured trade setups" | 64 | Publishes "entry strikes, premium targets, management plans, and exit criteria" ([about](https://wheelstrategy.substack.com/about)). Weekly batched signals on CSPs and covered calls is the ideal confirm pattern — subscriber sits down once a week and confirms a batch |
| 2 | **CFU** (joincfu.com) | **$99/mo** | ~1,200 traders in Discord | Daily alerts from 6 traders; covered calls, CSPs, credit spreads, iron condors, LEAPs | 58 | Publishes "exact entries, exits" and "stop-loss levels" ([pricing](https://joincfu.com/pricing)). Real paying audience at a real price. Claims "95.1% Win Rate across 879+ Verified Trades" with no stated verification method — which is exactly the gap our fill-based track record fills |
| 3 | **Options Income Academy alerts** | **$795/quarter, $2,297/year** — no monthly | n.a. | "2–4 live alerts per week", **"durations that range from 3 to 60 days"** | 50 | Dead centre of the frequency band, and "full trade lifecycle access: from entry to exit" ([alerts page](https://trader.optionsincomeacademy.com/alerts)). Prepaid annual billing is the highest migration friction on this list |
| Alt | **The Options Edge** | **$199/mo** | n.a. | "30-45 days to expiration at entry", manage "around 21 DTE" | 48 | Most precise mechanical description found — exact strikes, reasoning at entry/adjustment/exit, "take profit at 50% of max credit" ([Whop](https://whop.com/the-options-edge-fdae/)) |
| Alt | **Lee's Vertical Spread Trader** | n.a. — inquiry required | n.a. | **1–3 alerts/week**, explicitly "NOT a day-trading, swing-trading" service | 53 | Publishes "premium prices to shoot for on both the opening and closing sides" ([product page](https://www.smartoptionseller.com/vertical-spread-trader)) — both legs of the round trip, which almost nobody does |

## Table C — Swing equities with audience (6 slots)

These generate the funded-account volume the business case rests on.

| Rank | Creator | Price | Audience | Frequency / hold | Score | Why |
|---|---|---|---|---|---|---|
| 1 | **StockWhale** | **$399/year** (annual only) | **2,000+ Discord**, 100,000+ X followers, 1,895 active | "exact entries, targets, and stop-losses across stocks"; **"if your looking for 50 call outs a day this is not it"**; "great swings" | 63 | The best cold target on the list. He rules out day-trading volume in his own marketing and publishes the full price triple ([store](https://whop.com/stockwhale/), [product](https://whop.com/stockwhale/stockwhale/)). Annual prepaid billing is the friction — he has collected a year up front, so lead with dual-listing, not migration |
| 2 | **The Swing Trader** (Dad The Trader) | n.a. — Substack | **8,000+ subscribers** | Monday newsletter plus intraday notes; **"I don't have time for 'day trading'"** | 62 | Explicit swing positioning at real scale ([about](https://theswingtrader.substack.com/about)). Substack at 10%+ means dual-listing is easy |
| 3 | **The Smart Stocks Newsletter** (Francesco, @FranVezz) | n.a. — Substack | **5,000+ subscribers** | **"5x weekly"** — squarely in the band | 62 | The only creator in the set who publishes a numeric weekly cadence that lands inside our target band ([about](https://smartstocksnewsletter.substack.com/about)) |
| 4 | **PrimeTrading** (Alex) | n.a. — Substack + private Discord | **6,000+ subscribers** | "focuslist w/ precise entry levels", swing | 60 | Already runs a private Discord alongside Substack, so multi-channel is normal for him ([about](https://primetrading.substack.com/about)) |
| 5 | **Crents Suite** | n.a. — "Premium" tier | **3,704 members** listed | "Long-term & swing trade ideas", "swing trade ideas with entries and targets" | 59 | Largest verified Discord audience in the swing cohort ([DiscordServers](https://discordservers.com/server/1497918444702797956)). Publishes entries and targets but not exits — needs verification before a slot is committed |
| 6 | **The Tactical Allocation Letter** | n.a. — Substack | **19,000+ subscribers** — largest audience found | "396 trades over a decade" — under 1/week | 59 | Already publishes **"entry price, exit price, return, and duration" on every signal** ([about](https://tacticalallocationdesk.substack.com/about)). That is the closest thing to our verified-performance product that exists in the wild, self-published. Very low order volume per subscriber — recruit for the credibility and the audience, not the trade count |
| Alt | **Pivotal Trends** | **$55/mo**, 3-day trial | ~1,000 members | Store titled "Position trading & Discord"; "200+ charts monthly" | 53 | The best pure fit in the map we gave Jannick, but "not just copy trades" suggests he may not publish exact prices ([Whop](https://whop.com/pivotaltrends/pivotaltrends/)). Verify before committing a slot |
| Alt | **Stocks with Josh** | **$49/week** | 918 on the SWJ Trader tier, 8,557 store-wide, 4.6 over 769 reviews | n.a. | 49 | Large reviewed audience, but nothing published about frequency or holding period ([Whop](https://whop.com/marketplace/stocks-with-josh/)) |

## Table D — Marketplace plays (2 slots)

| Rank | Target | Why | The play |
|---|---|---|---|
| 1 | **US Stock Momentum** (KDT95, Collective2) | **Highest-scoring candidate on the entire list at 70.** 52 subscribers and **33 live AutoTraders** — the largest verified population of people already routing third-party signals into their own brokerage account. Monthly rebalance, **36.4-day average hold**, top-10 S&P 500 momentum names ([detail](https://collective2.com/details-list/151894080)) | Small audience, perfect behaviour. He is paying C2 30–50% for capability we give away. Recruit him as the reference case |
| 2 | **Collective2 publisher raid** — Orange Cat Management ($25/mo, **48.2-day** holds, 7 AutoTraders, CSPs and covered calls), Adaptive Investments ($495/mo, 17 AutoTraders, 2.3-day holds), Easy Value ($99/mo, 2.9-month holds), Fearless Value ($69/mo, 26.9-day holds), MAR1 QUANT | C2 verifies holding periods publicly, so the frequency filter is free — the "Trade Speed" column does our qualification for us. And C2's 30–50% take rate is the largest take-rate gap in the market | Approach publishers directly. Keep the C2 partnership conversation ("Matthew over at C2") separate and slower |

## Warm leads deliberately excluded

| Excluded | Why |
|---|---|
| **agtrader** | SPX 0DTE **and** needs index futures — disqualified twice. Genuinely the best technical talent in our pipeline and genuinely wrong for this launch. Revisit at futures |
| **Dave Mabe** | Tool vendor at $495–$5,000/mo, not a signal seller. Partner-or-compete conversation |
| **Chart Observer** | Crypto paper-trading simulator, no trade calls |
| **TradeVolatility** | 0DTE-oriented data platform selling levels, not calls |
| **Chris Pollio** (acct 5OS42960) | ~$1B annual volume, but a trader with no subscribers. His blocker is order latency (~240ms vs <100ms), a separate roadmap item |
| **DegenInvestor69** | 0DTE, latency-sensitive |
| **ScalarField, Astral, Kimpton, Sharkquant, OptionBots** | Platforms, not creators with audiences. Different motion — revisit as distribution channels **into** the marketplace |
| **The Options Fox, Options Alerts** | Intraday; Options Alerts advertises "50+ daily trades" |
| **Wheel Weekly** | Explicitly "not alerts telling you what to do" — a public journal, not a signal service |
| **Primetime Trading Group, The Traders Club, Stock Hours** | From our own map, all three publish nothing about holding period, and The Traders Club's lead magnet is a day-trading e-book. Cannot qualify on available evidence |

---

## Sequencing — and the mistake to avoid

**Do not put the biggest audience in first.** The instinct is to launch with the 19,000-subscriber letter because it produces the most accounts. That is backwards. The first creator through the pipe finds the OSI formatting bugs, the notification failures and the staleness edge cases. Finding those in front of 19,000 subscribers costs us that creator permanently, and creators talk to each other.

| Weeks | Who | Goal |
|---|---|---|
| 1–2 | Jay Allen, JasonL_Capital in sandbox | Break the webhook contract privately |
| 3–4 | Arvind, MAR1 QUANT live at small size | First real fills. First confirm-rate reading |
| 5–6 | Options income cohort | Second instrument type under load. Weekly batched confirms |
| 7–8 | Swing equities cohort, staged one at a time | Volume, once the product stops surprising us |
| 9–10 | US Stock Momentum and the C2 publishers | Only after there is a working reference to show |

## What we offer, by segment

| Segment | What they have now | The pitch |
|---|---|---|
| **Unmonetized technical builders** (Jay Allen, JasonL_Capital) | An integration and no rail | Nothing to migrate, no revenue to cannibalise. Fastest yes on the board |
| **Collective2 publishers** | A marketplace that executes, at **30–50%** | Same capability at 0%, and we are the broker — the track record comes from real fills, not simulated accounts. Largest take-rate gap available |
| **Substack creators** | 8,000–19,000 subscribers, no execution | 10%+ becomes 0%, and their subscribers stop hand-entering every trade. Keep the Substack |
| **Whop and Discord creators** | Checkout and a member list, no execution | 0% versus 3% plus processing, or Discord's 10%. Keep the Discord — no exclusivity |
| **Anyone claiming a win rate** (CFU's 95.1%, creditspread.net's backtested 85%) | Self-reported numbers a skeptical buyer discounts to zero | We compute performance from actual subscriber fills, net of fees. Nobody else can offer this, because nobody else is the broker |

**Universal founding-cohort terms:** 0% take rate, permanent for these creators, not promotional · no exclusivity, dual-list freely · direct line to me and to engineering · their feedback shapes the spec, and say so honestly.

## The ask

Fire your existing alert at a sandbox endpoint and tell us what breaks. Low commitment for anyone with alerts already configured, and the highest-value thing they can do for us — it surfaces the OSI problems, frequency mismatches and manifest gaps before subscribers are involved.

**What we must not promise:** a launch date. The terms amendment is a Legal deliverable and it gates launch. Telling fifteen creators we go live in six weeks and then slipping on our own paperwork is the fastest way to lose the cohort.

---

## Two things that weaken the business case

**1. The ~400-paid-subscriber assumption is anchored on an unreliable number.** The plan we gave Jannick sized the cohort at "~15 creators averaging ~400 paid subscribers ≈ 6,000 traders → 1,500–2,100 funded accounts," anchored on Stock Hours at 370 members and Pivotal Trends at 1,000. Verification found Whop's own displays contradict themselves — the Stock Hours store shows **"370 members" and "Join 1,000+ traders" on the same page**, and Primetime shows "4K members", "Join 4,500+ Traders" and "350+ members" simultaneously. One of the two numbers the estimate rests on is unreliable at the source.

Worse, the large audiences on this list are **Substack subscriber counts, which include free readers** — Substack does not publish paid counts, and every price cell for those five creators is `n.a.` because the paid tier price is not publicly displayed. The paying-subscriber figures we can actually verify are much smaller: 52 on C2's biggest strategy, 918 on the Stocks with Josh tier, 10 on Ike's tier, ~1,200 at CFU.

I would not defend 6,000 traders as a measured figure. It is a reasonable planning assumption with one weak anchor, and the first two founding creators will replace it with a real number. Say that to Jannick before someone else finds it.

**2. Not one candidate in Table C is confirmed to fire a webhook today.** Webhook readiness is 25% of the score and almost every audience creator scored 8 out of 25 on it, because no evidence either way was found. The four design partners and the C2 publishers are the only candidates with proven automation behaviour, and they are also the ones with almost no audience. **The scale and the readiness are in different people.** That is the central recruiting problem, and it is the argument for the sequencing above rather than a reason to change the cohort.

## What Danielle's interviews should produce

The instinct to talk to creators first is right; the cohort was wrong. Script creators are thinkorswim people with no webhook path — phase 3. Redirected at swing, position and options-income creators, the interviews should answer three questions that change the spec:

1. **How many signals do you publish in a normal week?** Fills the frequency gap — most creators on this list publish no numeric cadence at all.
2. **Would you accept your subscribers confirming each order individually?** The product's central bet. MAR1 QUANT already describes doing this voluntarily. If experienced creators say their audience won't, we need to know before launch.
3. **How many of your subscribers actually pay, and what would it take to move billing off Whop or Substack?** Directly repairs the weak anchor in the sizing above.

Open-ended workflow interviews produce colour and no decisions. These three produce numbers we can design against.
