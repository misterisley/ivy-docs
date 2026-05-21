# Best Valid Execution

[Previous: The Architecture](./04-architecture.md) | [Start](./README.md) | [Next: The Compute Market](./06-compute-market.md)

Nested Intents require a more precise definition of Best Execution.

In many markets, BestEx is discussed as if the problem were simply:

```text
Get the best price.
```

But for Nested Intents, raw price is not enough. A bid can offer an attractive price and still be invalid. It may violate the user's price band, exceed exposure limits, depend on insufficient liquidity, arrive after the cutoff, require too much verification compute, or fail a custom verifier.

The right object is not the best bid.

The right object is the best valid bid.

## Definition

```text
BestValidBid =
  argmax(score(bid))
  subject to:
    intent is live
    bid arrived before cutoff
    nestedVerifier(intent, bid, state) == true
    bid satisfies protocol rules
    verificationCost <= remainingComputeBudget
```

The validity constraints come before ranking.

Invalid bids do not compete.

## Why Validity Comes First

Consider three bids:

```text
Bid A:
  price: 110
  verifier: fails

Bid B:
  price: 104
  verifier: passes

Bid C:
  price: 102
  verifier: passes
```

If the user's maximum acceptable price is 105, then Bid A is not the best execution. It is not execution at all. It is invalid.

The valid set is:

```text
{ Bid B, Bid C }
```

The best valid bid is:

```text
Bid B
```

This sounds obvious in the price-only case, but it becomes more important as predicates become richer. A bid with a better price may fail because it creates too much portfolio exposure or because the liquidity behind it is not deep enough.

## Scoring Function

For V1, the scoring function should be simple.

For a buyer:

```text
score(bid) = -executionPrice
```

For a seller:

```text
score(bid) = executionPrice
```

Then select the best price among bids that pass the verifier.

Later, the score can become more expressive:

```text
ExecutionScore =
  PriceImprovement
  + ComputeFee
  + SolverReliabilityScore
  - VerificationCost
  - RiskPenalty
  - LatencyPenalty
  - SettlementFailurePenalty
```

The scoring function must be deterministic, versioned, and known before execution. Otherwise the executor can hide discretion inside the score.

## Validity Set

The validity set is the set of bids that satisfy all required checks:

```text
ValidBids(intent, bidSet, state) =
  [
    bid
    for bid in bidSet
    if intent.isLive
    if bid.arrivalTime <= cutoff
    if protocolRulesPass(bid)
    if nestedVerifier(intent, bid, state) == true
    if verificationCost(bid) <= intent.remainingComputeBudget
  ]
```

The final selection is:

```text
selectedBid = max(ValidBids, key = score)
```

If `ValidBids` is empty, nothing executes.

## Cutoffs

Cutoffs matter because they prevent the executor from waiting to see how the market moves before deciding which bids count.

The system should define:

```text
proposalOpenTime
proposalCloseTime
executionDeadline
```

Only proposals that arrive before `proposalCloseTime` are eligible.

This is where sequencing matters. If the executor controls bid visibility and ordering, it can abuse cutoffs. A Zellular sequencer or equivalent ordering system can establish a canonical bid set. See [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md).

## BestEx As A Proof Obligation

Nested Intents make BestEx easier to state as a proof obligation.

The executor should eventually prove:

```text
I selected bid B because:
  B was eligible.
  B passed the nested verifier.
  Every better-scoring bid either failed the verifier or was ineligible.
  The execution calldata matches B.
```

This is stronger than a generic claim that the solver "did its best."

It is a concrete, testable statement.

## From Best Price To Best Policy Satisfaction

For simple intents, BestEx is close to best price.

For richer intents, BestEx becomes best policy satisfaction.

Imagine a user defines:

```text
execute only if:
  price <= 105
  impliedVolatility <= 60%
  availableLiquidity >= 1,000,000
  portfolioDeltaAfterTrade <= 50
```

Now the best execution is not just the lowest price. It is the best bid among those that satisfy all policy constraints.

A bid at 101 with low liquidity may be worse than a bid at 103 with deep liquidity, depending on the scoring rule. The user's policy decides.

## Compute-Aware BestEx

Verification has cost.

If two bids produce similar economic value, but one requires enormous verification compute, the lower-compute bid may be better net execution.

This introduces a compute-aware score:

```text
NetExecutionScore =
  UserExecutionValue
  + ComputeFee
  - VerificationCost
  - RiskPenalty
```

The compute fee is not a bribe to execute invalid trades. It only affects priority among bids and intents that can be validly evaluated.

Validity still comes first:

```text
if nestedVerifier(...) == false:
  score = irrelevant
```

## BestEx And Long-Term Auctions

In a long-term silent auction, there may not be one obvious clearing moment. The mandate can remain open and executable only when conditions align.

The selection problem becomes:

```text
When should the system spend compute to evaluate?
When should it accept a proposal?
When should it wait?
```

This is why Nested Intents can be described as programmable optimal stopping.

The user defines the stopping condition:

```text
Execute when my predicate says the opportunity is valid.
```

The protocol defines the selection rule:

```text
Choose the best valid proposal under the scoring function.
```

The compute market defines the evaluation discipline:

```text
Do not spend unlimited resources checking low-value predicates.
```

## Failure Modes

Best Valid Execution must defend against several abuses:

### Ignoring Better Valid Bids

The executor chooses a worse valid bid while a better valid bid existed.

Mitigation:

- Audit logs
- Canonical bid sequencing
- TEE attestation
- ZK proof of no better valid bid

### Claiming A Bid Was Invalid

The executor falsely rejects a valid bid.

Mitigation:

- Verifier input and output logs
- Deterministic predicate execution
- Re-executable verification
- Proofs or attestations

### Including Late Bids

The executor allows a bid that arrived after the cutoff because it is favorable to the executor.

Mitigation:

- Sequencer timestamps
- Commitment windows
- Leaderless auction mechanisms

### Hiding The Bid Set

The executor claims the selected bid was best because it hides other bids.

Mitigation:

- Bid set commitments
- Public or sequenced proposal logs
- Slashing for omitted bids

### Abusing Compute Costs

The executor ignores a high-value bid by claiming verification was too expensive.

Mitigation:

- Published verification cost model
- User-defined max compute budget
- Deterministic compute metering
- Auditable refusal reasons

## The BestEx Formula

The simplest paper-ready formula is:

```text
BestValidExecution =
  best proposal under score(proposal)
  subject to:
    proposal arrived before cutoff
    intent is still live
    nestedVerifier(intent, proposal, state) == true
    verificationCost <= computeBudget
```

This formula is the heart of the system.

Everything else in the architecture exists to make it real:

- The verifier defines validity.
- The compute market pays for checking validity.
- The sequencer defines the eligible bid set.
- The solver selects and executes.
- The proof layer constrains the solver.

Next: [The Compute Market](./06-compute-market.md).

