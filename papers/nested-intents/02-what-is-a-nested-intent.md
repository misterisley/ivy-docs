# What Is A Nested Intent?

[Previous: The Speed Race](./01-the-speed-race.md) | [Start](./README.md) | [Next: From Orders To Execution Mandates](./03-from-orders-to-mandates.md)

A Nested Intent is an intent that contains another layer of execution logic inside it.

A normal intent says:

```text
I want outcome X. Find a way to execute it.
```

A Nested Intent says:

```text
I want outcome X, but execution is only valid if my nested rule set still approves it.
```

The base layer defines the desired action. The nested layer defines the conditions under which that action may still be executed.

## The Minimal Definition

```text
NestedIntent =
  BaseIntent
  + ExecutionPredicate
  + VerificationBudget
```

The `BaseIntent` answers:

```text
What does the user want?
```

The `ExecutionPredicate` answers:

```text
When is execution still allowed?
```

The `VerificationBudget` answers:

```text
Who pays for checking whether the predicate is true?
```

The execution predicate can be represented as:

```text
canExecute(intent, bid, marketState, oracleState, userLogic) -> true | false
```

Execution may proceed only if `canExecute(...)` returns true.

## The First Ivy / Symmio Version

The first version can be intentionally simple.

A bidder wants to place a long-lived bid. The bidder sets a minimum acceptable price and maximum acceptable price:

```text
minPrice <= executionPrice <= maxPrice
```

The solver or protocol-owned executor proposes execution terms. Before the trade is opened, a passthrough contract or verifier checks whether the proposed execution price is inside the user's allowed range.

If the price is valid:

```text
execute
```

If the price is invalid:

```text
reject
```

This is already a Nested Intent, because the intent contains a second rule that must be satisfied before execution can finalize.

## Example

A user wants to bid on an options market.

They define:

```text
market: ETH weekly call
side: buy
size: 10 contracts
minPrice: 95
maxPrice: 105
expiry: 7 days
computeBudget: 0.5 SYMMIO
```

A solver later proposes:

```text
executionPrice: 102
size: 10 contracts
```

The verifier checks:

```text
95 <= 102 <= 105
```

The result is true, so execution is allowed.

If the solver proposes:

```text
executionPrice: 111
```

the verifier checks:

```text
95 <= 111 <= 105
```

The result is false, so execution is blocked.

## Why This Is Not Just A Limit Order

A limit order usually protects one dimension: price. It is also usually tied to a venue, order type, and execution mechanism. It does not naturally carry arbitrary verification logic across time and execution contexts.

A Nested Intent generalizes the idea.

Price protection is only the first predicate. The nested layer can later check:

- Volatility
- Liquidity
- Funding rate
- Oracle state
- Portfolio exposure
- Solver reputation
- Market maker inventory
- Cross-market correlation
- Risk model output
- Custom user code
- Muon-verified external facts

The limit order asks:

```text
Is the price acceptable?
```

The Nested Intent asks:

```text
Is the execution acceptable under the user's full policy?
```

## Why This Is Not Just A Bytes Field

A common objection is:

```text
Isn't this just an intent with a bytes field that Muon or another verifier can read?
```

No.

The bytes field is only the transport format. It is not the primitive.

The primitive is the execution model around the bytes:

- The user-defined logic remains attached to the intent for its lifetime.
- The logic is checked whenever execution is attempted.
- The result determines whether execution is valid.
- The computation required to check it is economically priced.
- The executor is constrained by the predicate.
- The final execution can be audited or proven against the predicate.

Without that execution model, the bytes are inert configuration. With that execution model, the bytes become a programmable execution guard.

## The Nested Structure

The structure is nested because execution is not a single step. It contains an inner validation layer:

```text
User intent
  -> Solver quote or execution proposal
    -> Nested verifier checks user-defined constraints
      -> Valid proposal can execute
      -> Invalid proposal is rejected
```

Or more formally:

```text
submitIntent(user, baseIntent, nestedPredicate, budget)
observeMarket()
submitProposal(solver, intentId, proposedExecution)
result = canExecute(intent, proposedExecution, currentState)

if result == true:
  execute(proposedExecution)
else:
  reject(proposedExecution)
```

## Multiple Layers

Nested Intents can support multiple layers.

Layer 1 may be a simple price range:

```text
minPrice <= executionPrice <= maxPrice
```

Layer 2 may be a volatility constraint:

```text
impliedVolatility <= maxVolatility
```

Layer 3 may be a liquidity constraint:

```text
availableLiquidity >= minimumLiquidity
```

Layer 4 may be a portfolio constraint:

```text
postTradeExposure <= userRiskLimit
```

Layer 5 may be custom code:

```text
userModel(intent, bid, state) == true
```

The final predicate is the conjunction of all required layers:

```text
canExecute =
  priceCheck
  && volatilityCheck
  && liquidityCheck
  && exposureCheck
  && userModelCheck
```

The term "nested" becomes especially meaningful here. The main trading intent contains one or many sub-intents about the validity of execution.

## Adaptive Nested Intents

The advanced version is not limited to true or false.

A nested layer can also modify the intent:

```text
if volatility > threshold:
  reduce size by 50%

if liquidity improves:
  increase max execution size

if funding rate changes:
  adjust acceptable price range
```

This turns a Nested Intent into a smart execution policy. It can remain live for a long time while adapting to verified conditions.

The safe model is to distinguish two outputs:

```text
canExecute(...) -> true | false
transformIntent(...) -> updatedIntent
```

V1 should focus on `canExecute`. Later versions can introduce controlled transformation.

## The Key Sentence

A Nested Intent is not merely an intent with extra data.

It is a durable execution mandate whose validity is governed by a programmable, economically priced verifier.

Next: [From Orders To Execution Mandates](./03-from-orders-to-mandates.md).

