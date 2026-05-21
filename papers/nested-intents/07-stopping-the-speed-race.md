# Stopping The Speed Race

[Previous: The Compute Market](./06-compute-market.md) | [Start](./README.md) | [Next: Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md)

Nested Intents do not make computers slow. They make speed less central.

The goal is not to return to manual markets. The goal is to stop using faster reaction as the main defense against stale execution.

The current electronic market pattern is:

```text
Market changes
  -> old quote becomes unsafe
    -> participant races to cancel or update
      -> fastest machine avoids being picked off
```

Nested Intents change the pattern:

```text
Market changes
  -> verifier evaluates current state
    -> invalid executions are blocked
      -> participant does not need to win every cancellation race
```

This is the key move.

## Speed Is A Bad Substitute For Policy

A market maker does not actually want to update quotes for the sake of updating quotes. The market maker wants its displayed liquidity to reflect its current risk model.

In traditional systems, the way to keep quotes aligned with the model is to update them constantly.

Nested Intents allow a different expression:

```text
This liquidity is available only while my model says it is valid.
```

Now the model can be checked at execution time.

The machine still performs work. But the machine is checking policy, not merely racing to replace stale messages.

## From Reaction Time To Validity Time

In a latency market, the important time is reaction time:

```text
How quickly can I react after the world changes?
```

In a Nested Intent market, the important time is validity time:

```text
At the moment execution is attempted, is the user's policy still satisfied?
```

This changes the meaning of being "fast."

Fast no longer means only:

```text
I canceled before you hit me.
```

It also means:

```text
I can efficiently detect and prove when an execution window is valid.
```

The second form of speed is healthier. It rewards better verification, better models, better data, and better execution policy.

## Why Pricing Models Become More Important

If a user can attach a pricing model to execution, then the model becomes part of the market interface.

For example:

```text
execute only if:
  my volatility model marks this option underpriced
  market liquidity is above threshold
  post-trade exposure remains within risk limits
```

Now a better model can directly control execution.

The participant with the better model does not need to continuously spray and cancel orders to keep the market aligned with that model. The model itself becomes the nested predicate.

This encourages investment in:

- Better volatility models
- Better liquidity models
- Better risk models
- Better oracle construction
- Better execution scoring
- Better portfolio-aware policy
- Better optimal stopping rules

That is the desired direction.

## How This Helps Market Makers

Market makers constantly manage adverse selection. They quote prices, but they fear being executed when their quotes are stale.

Nested Intents let market makers publish conditional liquidity:

```text
I am willing to buy this option if:
  implied volatility is below X
  underlying price is inside range Y
  funding rate is below Z
  my inventory is below limit L
```

This liquidity can stay open longer because it is guarded.

Market makers can become less dependent on pure cancellation speed and more willing to expose liquidity under rich conditions.

## How This Helps Users

Users often cannot express their true execution policy in ordinary orders.

They may want:

```text
Buy only if price is good relative to volatility.
Sell only if liquidity is deep enough.
Open the position only if my portfolio risk remains acceptable.
Execute only if oracle data confirms the state.
```

Without Nested Intents, users must either:

- Simplify their policy into a crude order
- Trust an intermediary
- Monitor and update constantly
- Avoid leaving orders open

Nested Intents let users express policy directly.

## How This Helps Solvers

Solvers remain important.

But their role changes from pure speed competition to opportunity discovery under constraints.

A solver can compete by:

- Finding valid execution windows
- Sourcing better liquidity
- Reducing verification cost
- Improving proposal quality
- Building better state monitors
- Proving execution quality
- Offering stronger settlement guarantees

The solver's job becomes:

```text
Find the best valid execution.
```

not merely:

```text
Be first to execute any available order.
```

## What Speed Still Does

Nested Intents do not abolish latency.

Speed remains useful for:

- Observing markets
- Detecting execution windows
- Submitting proposals before cutoffs
- Running verification efficiently
- Settling before deadlines
- Avoiding stale oracle inputs

The point is not that speed becomes irrelevant. The point is that speed is no longer the primary protection mechanism.

With Nested Intents, the user's protection is the verifier.

Speed serves verification. It does not replace it.

## The New Competition

Old competition:

```text
Who can update fastest?
```

New competition:

```text
Who can produce the best valid execution under the user's policy?
```

Old edge:

```text
lower latency path
faster cancellation
faster matching-engine access
```

New edge:

```text
better pricing model
better risk model
better verification efficiency
better liquidity sourcing
better proof of execution quality
```

This is why Nested Intents can be described as a market-structure correction.

## Why This Solves A Computer-Age Problem

The core computer-age problem is not that machines execute. The problem is that static instructions become stale faster than humans or ordinary systems can update them.

Nested Intents solve this by changing what is stored.

Instead of storing only:

```text
price
size
side
```

the system stores:

```text
execution policy
verification budget
validity predicate
settlement constraints
```

The stored object is no longer just an order. It is a rule for deciding whether execution is allowed.

That is the deep solution.

If the rule travels with the intent, the user does not need to race every state change. The rule handles state changes.

## Finance After Nested Intents

A mature Nested Intent market would look different from a latency-first market.

Liquidity can be:

- Long-lived
- Conditional
- Model-driven
- Portfolio-aware
- Oracle-aware
- Compute-priced
- Auditable
- Provably constrained

Execution can be judged by:

- Whether the mandate was live
- Whether the bid arrived before cutoff
- Whether the verifier passed
- Whether the selected bid was the highest-scoring valid bid
- Whether the compute spent was within budget
- Whether settlement matched the selected proposal

That is much closer to real Best Execution than "the fastest machine got there first."

## The Main Claim Restated

Nested Intents stop the speed race by moving user protection from reaction time into execution validity.

They do not eliminate fast machines. They subordinate fast machines to verified policy.

That is how finance can return to improving pricing models instead of endlessly improving engines.

Next: [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md).

