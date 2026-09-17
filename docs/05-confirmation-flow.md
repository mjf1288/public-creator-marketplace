# Confirmation Flow

The subscriber experience from notification to fill. This surface carries most of the product's compliance weight and most of its perceived quality.

---

## Constraint set

Every item below is a hard requirement, and each maps to a specific exposure documented in `07-compliance-controls.md`.

1. **One confirmation per order.** No "approve all," no "confirm the next N," no session-level approval, no standing consent.
2. **Off by default.** A new subscription notifies but cannot pre-authorize.
3. **Timeout expires, never executes.**
4. **Full order terms shown** before confirmation: symbol, side, order type, limit or stop price, computed quantity, estimated cost, estimated fees, buying-power impact.
5. **No countdown pressure.** Show expiry as a time, not a ticking urgency device.
6. **No streak, completion or participation mechanics.** Nothing that rewards confirming or penalizes declining.

Items 5 and 6 look like UX preferences. They are not. A design that pressures a subscriber into confirming is evidence that the confirmation was not a real decision, which is precisely the argument that turns propose-and-confirm into de facto discretion.

## Notification

Push, with email fallback if push is unavailable and the subscriber opted in.

Content: creator name, agent name, symbol, side, and expiry time. It must **not** contain a one-tap confirm from the notification itself — confirmation requires seeing the full order terms, and a notification cannot show cost, fees and buying-power impact.

Deep link to the proposal. If the proposal is already terminal by the time it opens, show the terminal state and why.

## Confirm screen

Required elements, in this order:

**1. Provenance.** Creator name, agent name, and signal receipt time. The subscriber must know this originated with the creator, not Public.

**2. The creator's note, verbatim.** Their rationale, unedited, attributed to them. Public does not summarize, rewrite, or add its own commentary. Editing creator content is adoption of it.

**3. Order terms.**

```
BUY 3 × AAPL
Limit $227.50 · Day · Core hours

Sizing        3 shares — 2% of $18,400 buying power
Est. cost     $682.50
Est. fees     $0.00
Buying power  $18,400 → $17,717.50

Expires 2:45 PM ET
```

**4. The sizing explanation.** Never a bare quantity. The subscriber configured a rule and needs to see the rule produced this number.

**5. Confirm and Decline as equal-weight actions.** Decline is not a secondary or greyed control.

**6. A persistent link to change sizing** — which applies to future proposals, never to this one. Editing the current proposal's size would make the subscriber's own confirmation the sizing decision, which is fine in principle but adds a second decision to every order and collapses confirm rate. Sizing is a settings decision, made once.

## What the confirm screen must not contain

- Any performance claim about the agent, whether historic or implied
- Any Public-authored encouragement, framing, or characterization of the trade
- Any indication of how many other subscribers have confirmed
- Any suggestion that declining is unusual, or a "you're missing out" pattern
- Any ranking, badge, or score derived from returns

The third item deserves emphasis. Showing "312 subscribers confirmed" manufactures social proof for a securities transaction and simultaneously leaks aggregate subscriber behavior. It is the kind of feature that gets built because it lifts conversion, and it is exactly the wrong thing here.

## Confirmation handling

On confirm:

1. Compare-and-set `NOTIFIED → CONFIRMED`. If already terminal, return that state idempotently
2. Re-run preflight
3. Re-fetch quote and evaluate staleness
4. If clean, submit with a client order id derived from `proposal_id`
5. Transition to `SUBMITTED`, show a submitted state, and never claim a fill

Double-tap and network retry must be safe. This runs on phones with poor connectivity, and a duplicated live order is the worst outcome this system can produce.

## Staleness at confirmation

Confirmation can arrive well after notification, by which point the market has moved.

| Condition at confirm | Behavior |
|---|---|
| Limit price now through the market beyond tolerance | Do not submit. Re-confirmation prompt showing the current quote and what changed |
| Buying power no longer covers the order | Do not submit. Show the shortfall and offer to decline |
| Account restricted | `SKIPPED`, explain |
| Symbol halted | `VOIDED`, explain |
| Market now closed and `session=CORE` | Offer to queue for next open with explicit re-confirmation, or decline |

The re-confirmation prompt is a second deliberate decision, which is defensible. Silently submitting a stale limit order is not, and it produces the complaint that the creator's signal was "wrong" when in fact the fill was.

## Expiry

At `expires_at` with no action: transition to `EXPIRED`, notify the subscriber that it expired, and record it. No auto-submission under any circumstance, including a configuration flag, an admin override, or a support tool.

Expired proposals count against the agent's confirm rate. That is intentional. A creator whose subscribers cannot keep up should see it in their own analytics.

## Fill notification

Separate from confirmation. On fill, notify with symbol, filled quantity, average price and fees. On partial fill, show filled and remaining. On rejection, show the broker reason in plain language.

Slippage against the signal's reference quote is shown to the subscriber. This is unusual transparency and it is the right call: a subscriber who can see their fill quality trusts the platform, and a creator who can see aggregate slippage learns whether their signals are actually executable.

## Guardrails, and how skips surface

A proposal skipped before notification is never shown as an action. It appears in the subscriber's activity history with the reason.

| Skip reason | Subscriber-facing message |
|---|---|
| `INSUFFICIENT_BUYING_POWER` | Not enough buying power for your configured size |
| `POSITION_CAP` | You're at your maximum open positions for this agent |
| `DAILY_ORDER_CAP` | You've hit your daily order limit for this agent |
| `NOTIONAL_CEILING` | Order exceeded your per-order limit |
| `INSTRUMENT_NOT_ALLOWED` | Outside the instruments you allowed |
| `ACCOUNT_RESTRICTED` | Account restriction prevented this order |
| `MARKET_CLOSED` | Market closed and this agent trades core hours only |

Skips are not failures and should not read as errors. They are the guardrails the subscriber configured working correctly.

## Accessibility and the quiet-hours problem

An agent that publishes outside core hours generates notifications at times a subscriber may not see. Phase one: agents declare `publishes_outside_core_hours`, subscribers see it before subscribing, and TTLs extend accordingly.

Subscriber-configurable quiet hours are a should-have. If a proposal arrives during quiet hours, hold the notification and extend expiry — but only if the extended expiry is still within the agent's declared maximum. Otherwise let it expire and record it, rather than surfacing a proposal whose price context is hours stale.
