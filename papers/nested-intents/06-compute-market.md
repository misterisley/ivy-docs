# The Compute Market

[Previous: Best Valid Execution](./05-best-valid-execution.md) | [Start](./README.md) | [Next: Stopping The Speed Race](./07-stopping-the-speed-race.md)

Nested Intents become powerful when users can define arbitrary verification logic. But arbitrary verification cannot be free.

If the system allows anyone to attach any predicate to an intent, a user could submit:

```text
Calculate implied volatility using a heavy model.
Check a large portfolio.
Fetch many oracle inputs.
Run a complex Muon verifier.
Evaluate a custom model with billions of parameters.
```

Some of those predicates may be valuable. Some may be spam. Some may cost more to evaluate than the execution opportunity is worth.

The solution is a compute market.

## The Basic Rule

Every Nested Intent must pay for the computation required to verify it.

```text
No free arbitrary predicates.
```

The user attaches a compute budget:

```text
computeBudget: amount of SYMMIO or another accepted token
maxCostPerCheck: maximum spend for one verification attempt
priorityFee: optional fee for higher evaluation priority
```

When a solver or verifier evaluates the predicate, the cost is charged against the budget.

## Why SYMMIO Fits

In the Ivy / Symmio framing, SYMMIO can become the credit layer for verification compute.

SYMMIO can pay for:

- Predicate evaluation
- Muon oracle computation
- External data verification
- Keeper monitoring
- Proposal ranking
- Proof generation
- TEE execution
- Sequencer inclusion
- Settlement gas reimbursement

This gives the token a direct role in execution infrastructure:

```text
SYMMIO prices the computation required to prove executability.
```

## Compute-Aware Ranking

Executors should not evaluate every possible predicate blindly. They should prioritize checks based on expected value.

A simple ranking formula:

```text
ExecutionPriority =
  ExpectedExecutionValue
  + ComputeFee
  - VerificationCost
  - RiskPenalty
```

If a bid has high expected value and low verification cost, it should be evaluated first.

If a bid has low expected value and extremely high verification cost, it may not be worth evaluating unless the user pays enough compute fee.

This is not unfair. It is necessary. Without compute pricing, arbitrary smart intents become a denial-of-service vector.

## Validity Versus Priority

Compute fees should affect priority, not validity.

A user cannot pay to make an invalid execution valid.

The distinction is:

```text
Validity:
  Does the predicate return true?

Priority:
  Is it economically worth spending compute to check this predicate now?
```

A high compute fee can cause the system to check an intent earlier.

It cannot cause the system to execute if `canExecute(...)` returns false.

## Example

Two open Nested Intents exist.

```text
Intent A:
  expected value: 100
  compute fee: 5
  verification cost: 2
  risk penalty: 1

Intent B:
  expected value: 120
  compute fee: 1
  verification cost: 80
  risk penalty: 5
```

Scores:

```text
Intent A priority = 100 + 5 - 2 - 1 = 102
Intent B priority = 120 + 1 - 80 - 5 = 36
```

Intent B has higher raw execution value, but it is much more expensive to verify. Intent A is likely checked first.

This is how the system avoids wasting compute on expensive predicates unless the value justifies it.

## Compute Budgets And User Sovereignty

The user should be able to choose how much verification they are willing to fund.

Simple intent:

```text
check price range only
low budget
```

Complex intent:

```text
check volatility, liquidity, portfolio exposure, and oracle state
higher budget
```

Institutional intent:

```text
run custom pricing model and risk constraints
large budget
```

This creates a natural market segmentation. Retail users can use cheap standard predicates. Market makers and sophisticated users can pay for more expressive logic.

## Standard Predicate Library

The protocol should support a standard library of common predicates.

Examples:

```text
PRICE_RANGE
MAX_SLIPPAGE
MIN_LIQUIDITY
MAX_VOLATILITY
MAX_FUNDING_RATE
MAX_PORTFOLIO_EXPOSURE
ORACLE_PRICE_BAND
SOLVER_ALLOWLIST
SOLVER_REPUTATION_MINIMUM
```

Standard predicates are important because they are:

- Easier to audit
- Easier to price
- Easier to cache
- Easier to prove
- Easier for users to understand

Custom predicates should exist, but the common path should be simple.

## Custom Predicate Market

Advanced users can supply custom logic.

The custom predicate may be:

- A Muon app
- A versioned verifier contract
- A WASM module
- A signed off-chain model output
- A TEE-executed program
- Eventually, a ZK-verifiable circuit

The protocol should require:

```text
code hash
version
input schema
cost estimate
max runtime
allowed oracle dependencies
failure behavior
```

The goal is not to allow arbitrary unbounded computation. The goal is to allow expressive bounded computation.

## Verification Cost Model

The cost model should be transparent.

Possible cost components:

```text
baseCost
oracleFetchCost
stateReadCost
modelExecutionCost
proofGenerationCost
settlementGasCost
riskPremium
```

The final cost:

```text
VerificationCost =
  baseCost
  + oracleFetchCost
  + stateReadCost
  + modelExecutionCost
  + proofGenerationCost
  + settlementGasCost
  + riskPremium
```

For V1, this can be simple and conservative. For later versions, costs can become more precise.

## Caching And Reuse

Many predicates may depend on the same state.

For example, thousands of intents may need:

- Current ETH price
- Current implied volatility
- Current liquidity at a venue
- Current funding rate

The system can cache shared inputs:

```text
one oracle update
many predicate checks
```

This reduces total verification cost and makes the compute market more efficient.

## Preventing Compute Griefing

Potential attacks:

- Submit extremely expensive predicates with low budget.
- Force solvers to inspect many worthless intents.
- Hide expensive computation behind innocent-looking config.
- Create predicates that often fail after consuming compute.
- Depend on unstable oracle inputs to cause repeated rechecks.

Mitigations:

- Require upfront compute budget.
- Require declared max cost per check.
- Reject predicates that exceed max runtime.
- Maintain standard predicate libraries.
- Charge failed checks when work was actually performed.
- Let executors ignore low-priority predicates.
- Use reputation or deposits for custom predicate authors.

## Why The Compute Market Is Philosophically Important

The compute market is not just an implementation detail. It is what makes the whole idea honest.

If Nested Intents are supposed to move competition away from machine speed and back toward better models, then model complexity must be priced. A sophisticated model should be allowed, but it should pay for the resources it consumes.

This creates a clean trade:

```text
You may express richer execution policy,
but you must fund the verification of that policy.
```

That is how the system supports complexity without being overwhelmed by it.

## Connection To Best Valid Execution

Compute cost appears in two places:

```text
Validity:
  verificationCost <= remainingComputeBudget

Ranking:
  score includes verification cost or compute fee
```

The next chapter returns to the market-structure question: if verification logic can protect users across time, how does that stop the speed race?

Read next: [Stopping The Speed Race](./07-stopping-the-speed-race.md).

