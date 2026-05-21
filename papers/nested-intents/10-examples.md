# Examples And Use Cases

[Previous: Comparisons](./09-comparisons.md) | [Start](./README.md) | [Next: Objections And Answers](./11-objections.md)

Nested Intents become clearer through examples.

Each example has the same structure:

```text
base intent:
  what the user wants

nested predicate:
  when execution is allowed

execution result:
  what happens when the predicate passes or fails
```

## Example 1: Simple Price-Protected Bid

Base intent:

```text
Buy 10 option contracts.
```

Nested predicate:

```text
minPrice <= executionPrice <= maxPrice
```

Concrete values:

```text
minPrice: 95
maxPrice: 105
```

Solver proposal:

```text
executionPrice: 102
```

Verifier:

```text
95 <= 102 <= 105
```

Result:

```text
execute
```

If the solver proposes `111`, the result is:

```text
reject
```

This is the V1 Nested Intent.

## Example 2: Volatility-Aware Option Bid

Base intent:

```text
Buy ETH call options.
```

Nested predicate:

```text
execute only if:
  executionPrice <= maxPrice
  impliedVolatility <= maxIV
```

The user is not merely saying "buy below this price." They are saying the option must be attractive relative to volatility.

Verifier:

```text
priceCheck = executionPrice <= maxPrice
ivCheck = muonCalculatedIV <= maxIV

canExecute = priceCheck && ivCheck
```

This moves execution closer to a real pricing model.

## Example 3: Liquidity-Protected Trade

Base intent:

```text
Open a position in a market.
```

Nested predicate:

```text
execute only if:
  availableLiquidity >= minimumLiquidity
  expectedSlippage <= maxSlippage
```

Why it matters:

A price may look good, but if liquidity is shallow, execution quality may be poor. The nested predicate lets the user protect against thin markets.

## Example 4: Portfolio-Aware Execution

Base intent:

```text
Buy BTC exposure.
```

Nested predicate:

```text
execute only if:
  postTradePortfolioDelta <= maxDelta
  postTradeMarginUsage <= maxMarginUsage
```

This is useful for users who want execution to depend on their whole portfolio, not just one order.

Verifier:

```text
postTradeState = simulatePortfolio(intent, proposal, currentPortfolio)

canExecute =
  postTradeState.delta <= maxDelta
  && postTradeState.marginUsage <= maxMarginUsage
```

The intent can remain open, but only executes while the user's risk envelope allows it.

## Example 5: Funding-Rate Constraint

Base intent:

```text
Open a perpetual position.
```

Nested predicate:

```text
execute only if:
  fundingRate <= maxFundingRate
```

This prevents execution when the headline price is acceptable but carry cost is not.

## Example 6: Solver Reputation Constraint

Base intent:

```text
Execute through any approved solver.
```

Nested predicate:

```text
execute only if:
  solverReputation >= minimumReputation
  solverIsNotSlashed == true
```

This lets users include counterparty-quality constraints.

The best price from a bad solver may not be the best valid execution.

## Example 7: Long-Term Silent Auction

Base intent:

```text
Sell a structured option package within 30 days.
```

Nested predicate:

```text
execute only if:
  packagePrice >= minimumAcceptablePrice
  impliedVolatility is inside user's target band
  availableLiquidity is sufficient
  settlement risk is below threshold
```

Flow:

```text
Day 1:
  intent is posted

Day 2-14:
  conditions do not pass

Day 15:
  several market makers submit proposals

Verifier:
  filters invalid proposals

Executor:
  selects best valid proposal
```

The user did not need to constantly update the order. The mandate waited.

## Example 8: Adaptive Intent

Base intent:

```text
Buy an option position.
```

Nested transformation:

```text
if impliedVolatility > 70%:
  reduce size by 50%

if impliedVolatility > 90%:
  block execution
```

This requires a more advanced version because the nested layer modifies the intent rather than merely approving or rejecting it.

Safe architecture:

```text
transformIntent(intent, state) -> updatedIntent
canExecute(updatedIntent, proposal, state) -> true | false
```

V1 should probably not start here, but this is an important future path.

## Example 9: Custom Market Maker Model

Base intent:

```text
Provide conditional liquidity for an options market.
```

Nested predicate:

```text
execute only if:
  marketMakerModel(intent, proposal, state) == true
```

The market maker supplies:

```text
model hash
input schema
oracle dependencies
max runtime
compute budget
```

The protocol or Muon executes the verifier and charges the compute budget.

This is the moment where Nested Intents become a bridge between pricing models and execution.

## Example 10: Moon Temperature

Absurd examples are useful because they show the abstraction.

Base intent:

```text
Execute trade X.
```

Nested predicate:

```text
execute only if:
  verifiedMoonTemperature is between A and B
```

This is not necessarily a useful financial policy. The point is that the protocol does not need to know in advance what every useful predicate will be.

The system only needs to enforce:

```text
predicate is bounded
predicate is paid for
predicate is verified
predicate returns true before execution
```

## Example 11: Compute-Aware Bid Selection

Two bids:

```text
Bid A:
  execution value: 100
  verification cost: 2
  verifier passes

Bid B:
  execution value: 105
  verification cost: 20
  verifier passes
```

If the scoring rule is raw execution value, Bid B wins.

If the scoring rule is net execution value:

```text
NetScore = executionValue - verificationCost
```

then:

```text
Bid A net score = 98
Bid B net score = 85
```

Bid A wins.

This must be defined in advance. The user and protocol need to know whether compute costs affect ranking and how.

## Example 12: No Valid Bid

Open intent:

```text
buy if price <= 100
```

Bids:

```text
Bid A: price 101
Bid B: price 104
Bid C: price 110
```

Valid set:

```text
empty
```

Result:

```text
no execution
```

This is a feature. The system should not execute merely because bids exist. It should execute only when valid bids exist.

## Example Summary

Nested Intents can start with price protection and grow toward model-driven execution.

The same shape holds at every level:

```text
desired trade
  + validity predicate
  + compute budget
  + best valid selection
  = programmable execution mandate
```

Next: [Objections And Answers](./11-objections.md).

