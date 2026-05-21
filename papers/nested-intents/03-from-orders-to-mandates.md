# From Orders To Execution Mandates

[Previous: What Is A Nested Intent?](./02-what-is-a-nested-intent.md) | [Start](./README.md) | [Next: The Architecture](./04-architecture.md)

Nested Intents matter because they change the time horizon of financial instructions.

Most trading systems are built around orders. An order is an instruction placed into a specific execution context. It may be active for a moment, a day, or until canceled, but its validity is usually represented by a relatively small set of fields:

```text
asset
side
size
price
time in force
```

An order is not usually a full expression of the user's execution policy. It is a compact representation of what the user is willing to do under a narrow set of assumptions.

A mandate is different. A mandate is an authorization with conditions. It says:

```text
You may execute for me, but only under these rules.
```

Nested Intents turn intents into mandates.

## The Difference Between An Order And A Mandate

An order is operational:

```text
Buy 100 units at price P.
```

A mandate is policy-based:

```text
Buy 100 units when my model says the execution is valid, provided the price,
volatility, liquidity, and portfolio constraints still hold.
```

The order is a message to a venue.

The mandate is a rule set for execution.

The order must be updated when the user's policy changes.

The mandate contains the policy that determines whether execution is still allowed.

## Why Intent Systems Need This Shift

Intent systems already improve on raw orders. They let a user describe an outcome and let solvers compete to satisfy it.

But many intent systems are still short-horizon systems. They answer:

```text
Who can satisfy this intent now?
```

Nested Intents answer a deeper question:

```text
When, if ever, should this intent be satisfied?
```

That is the difference between immediate execution and optimal stopping.

## Persistent Execution

A Nested Intent can remain live across time.

The user can say:

```text
This mandate may remain open for 7 days.
This mandate may remain open for 30 days.
This mandate may remain open for a year.
This mandate may remain open indefinitely, until canceled or exhausted.
```

The duration alone is not the innovation. Long-lived orders already exist. The innovation is that the mandate remains protected by a verifier that evaluates current conditions at execution time.

A long-lived static order becomes dangerous as the world changes.

A long-lived Nested Intent can remain safe because execution depends on current validation.

## Silent Long-Term Auctions

Nested Intents enable silent long-term auctions.

In a normal auction, participants gather around a defined moment. They submit bids. The auction clears.

In a Nested Intent market, the auction object can exist silently over time:

```text
User publishes a long-term mandate.
Solvers and market makers monitor it.
Market conditions evolve.
When execution becomes attractive, proposals are submitted.
The nested verifier filters invalid proposals.
The best valid proposal executes.
```

The user does not need to constantly broadcast new orders. The mandate waits.

This changes the shape of liquidity. Liquidity can become conditional, latent, and programmable. It does not need to be continuously displayed as a fragile quote.

## Best Execution Over Time

Traditional BestEx is often framed as the best available execution at a moment:

```text
best price now
best venue now
best route now
```

Nested Intents extend this into time:

```text
best valid execution over the lifetime of the mandate
```

The word "valid" is essential. A bid is not good merely because it has the highest raw price or largest size. It is good only if it satisfies the user's nested conditions.

For example:

```text
Bid A:
  price improvement: high
  verifier: fails due to liquidity

Bid B:
  price improvement: medium
  verifier: passes

Best valid bid: Bid B
```

The mandate is not looking for the highest number in isolation. It is looking for the highest valid execution under the user's policy.

## Why This Reduces The Need For Constant Updating

In the old model, the user or market maker updates the order because the order cannot update its own meaning.

In the Nested Intent model, the predicate defines the meaning.

If market conditions move outside the user's range, execution stops automatically:

```text
canExecute(...) == false
```

If market conditions move back into the user's range, execution becomes possible again:

```text
canExecute(...) == true
```

The user does not need to send a cancel and replace message for every state transition. The predicate absorbs the state transition.

## Mandates And Solver Discretion

This shift also changes solver power.

Without a nested verifier, much of the execution policy lives inside the solver:

```text
The solver decides when execution is good enough.
```

With a nested verifier, the user defines the policy:

```text
The solver may execute only when the user's verifier says execution is allowed.
```

This does not eliminate solver discretion completely. Solvers can still choose what to monitor, when to quote, how to source liquidity, and how to price risk. But the final validity of execution is no longer merely a solver promise. It is a checked condition.

## The Role Of The Protocol-Owned Solver

An initial implementation can use a protocol-owned solver.

The protocol-owned solver:

- Monitors open Nested Intents
- Receives or sources bids
- Checks simple predicates
- Selects the best valid bid
- Routes execution through the appropriate Symmio or PartyB pathway
- Produces logs for auditability

This is practical for V0. It does introduce trust in the executor, which is addressed in [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md).

The important point is that the solver is not conceptually unconstrained. Even in V0, it should be modeled as an executor of mandates, not the owner of the user's execution policy.

## Why "Mandate" Is The Right Word

The word "intent" is useful because it captures desired outcome. But the word "mandate" captures authority.

A Nested Intent authorizes execution under rules:

```text
You may execute this trade for me.
You may not execute it if my verifier fails.
You may spend up to this compute budget checking it.
You may settle only the best valid proposal under the scoring rule.
```

That is stronger than an order.

It is closer to a programmable agency agreement between the user and the execution network.

## Transition To Architecture

Once we define Nested Intents as mandates, the architecture becomes clear:

```text
mandate creation
  -> monitoring
    -> proposal collection
      -> verification
        -> scoring
          -> execution
            -> audit or proof
```

The next chapter describes that architecture in detail: [The Architecture](./04-architecture.md).

