# The Architecture

[Previous: From Orders To Execution Mandates](./03-from-orders-to-mandates.md) | [Start](./README.md) | [Next: Best Valid Execution](./05-best-valid-execution.md)

Nested Intents require more than a data structure. They require an execution architecture that treats user-defined validity rules as first-class constraints.

The architecture has seven conceptual parts:

1. Intent authoring
2. Predicate registration
3. Compute budgeting
4. Market monitoring
5. Proposal collection
6. Verification and scoring
7. Settlement and proof

The implementation can start simple and become more verifiable over time.

## High-Level Flow

```text
User creates Nested Intent
  -> Intent stores base trade and nested predicate
    -> User attaches compute budget
      -> Solvers / market makers / protocol-owned executor monitor intent
        -> Execution proposal is submitted
          -> Nested verifier checks current validity
            -> Best valid proposal is selected
              -> PartyB passthrough or settlement module enforces execution
                -> Logs, attestation, or proof are produced
```

The key property is that execution is not valid merely because a solver submitted a proposal. It is valid only if the nested verifier approves that proposal against current state.

## Core Objects

### Base Intent

The base intent defines the desired trade.

Example fields:

```text
intentId
user
marketId
side
size
expiry
collateral
settlementPath
```

The base intent is the "what."

### Nested Predicate

The nested predicate defines the execution guard.

Example fields:

```text
predicateType
predicateCodeHash
predicateConfig
oracleRequirements
stateRequirements
maxVerificationCost
```

The nested predicate is the "when."

For V1, `predicateType` can be simple:

```text
PRICE_RANGE
```

and `predicateConfig` can contain:

```text
minPrice
maxPrice
```

Later, `predicateType` can support Muon-verified custom logic:

```text
MUON_CUSTOM_VERIFIER
```

### Compute Budget

The compute budget pays for verification.

Example fields:

```text
budgetToken
maxBudget
remainingBudget
maxCostPerCheck
priorityFee
refundAddress
```

This prevents arbitrary predicates from becoming free compute attacks.

### Execution Proposal

A proposal is submitted by a solver, market maker, or protocol-owned executor.

Example fields:

```text
proposalId
intentId
solver
executionPrice
executionSize
deadline
quoteHash
settlementCalldataHash
expectedVerificationCost
solverFee
```

The proposal is not accepted until the verifier approves it.

### Verification Result

The verification result records whether the nested predicate passed.

Example fields:

```text
intentId
proposalId
stateRootOrOracleSnapshot
predicateInputHash
predicateOutput
verificationCost
timestamp
verifierSignatureOrProof
```

The minimum result is a boolean. The stronger version includes enough data to audit or prove the decision.

## V0 Architecture: Protocol-Owned Solver

The pragmatic first implementation can use a protocol-owned solver.

```text
Nested Intent registry
  -> Protocol-owned solver watches open intents
    -> Solver sources or receives bids
      -> Solver checks min/max price
        -> Solver executes best valid bid
          -> Execution logs are published
```

This version is fast to ship and easy to reason about.

The trust assumption is explicit:

```text
Users trust the protocol-owned solver to select the best valid bid.
```

Even in V0, the system should avoid vague discretion. The selection rule should be deterministic and logged.

For the first price-only version:

```text
BestValidBid =
  best price
  subject to:
    minPrice <= executionPrice <= maxPrice
    bid arrived before cutoff
    intent is live
```

## V1 Architecture: Auditable Execution

V1 adds audit material.

Each execution should produce:

- Intent ID
- Bid set hash
- Cutoff time
- Selected bid
- Rejected bid hashes
- Rejection reasons
- Verifier input hash
- Verifier output
- Settlement calldata hash
- Execution transaction hash

This does not make cheating impossible, but it makes many forms of cheating detectable.

The trust model becomes:

```text
Users trust the solver during execution, but abuse can be detected after the fact.
```

## V2 Architecture: Canonical Bid Sequencing

V2 introduces an external ordering layer such as a Zellular sequencer or another canonical sequencing mechanism.

The sequencer establishes:

```text
This bid existed before the cutoff.
This bid arrived after the cutoff.
This was the canonical bid set for the execution window.
```

This reduces the executor's ability to claim:

```text
I never saw the better bid.
```

The trust model becomes:

```text
The solver cannot easily rewrite bid history.
```

## V3 Architecture: TEE Attestation

V3 can run selection logic inside a trusted execution environment.

The TEE receives:

```text
sequenced bid set
nested intent
oracle state
selection algorithm version
```

It outputs an attestation:

```text
Given these inputs, the selected bid is the best valid bid.
```

TEEs are practical because they can handle complex predicates and larger input sets more easily than early ZK circuits. The tradeoff is hardware trust.

## V4 Architecture: ZK Proof Of Best Valid Bid

V4 proves selection cryptographically.

A ZK proof can show:

1. The selected bid was in the committed bid set.
2. The selected bid arrived before the cutoff.
3. The selected bid satisfies the nested verifier.
4. No other valid bid has a better score.
5. The settlement calldata matches the selected bid.

The trust model becomes:

```text
The executor may execute, but cannot finalize without proof that execution obeyed the mandate.
```

## PartyB Passthrough

In the Ivy / Symmio framing, PartyB passthrough can act as the enforcement boundary.

The execution path is:

```text
Solver proposal
  -> Nested verifier
    -> PartyB passthrough check
      -> Symmio execution
```

The leaderless auction, protocol-owned solver, TEE, or ZK proof may decide which proposal should win. The PartyB passthrough enforces whether that winning proposal is allowed to execute.

This separation is useful:

```text
Selection layer:
  Which proposal wins?

Verification layer:
  Does the proposal satisfy the user's mandate?

Settlement layer:
  Execute only if verification passes.
```

## Muon And External State

Muon can provide the verification bridge for conditions that depend on external or computed data.

Example checks:

- Implied volatility calculation
- Liquidity threshold
- Funding rate
- Oracle price
- Portfolio value
- Cross-market state
- Custom model output

The nested predicate can reference Muon-provided data:

```text
canExecute(intent, bid, marketState, muonData, userLogic)
```

This is where Nested Intents become much more powerful than ordinary orders. The verifier can depend on computed facts, not just static fields.

## Execution Windows

Nested Intents can be checked continuously, periodically, or only when a proposal arrives.

Three possible models:

```text
Reactive:
  Check only when a solver submits a proposal.

Scheduled:
  Check every N blocks or every N seconds.

Triggered:
  Check when an oracle, price, or market-state threshold changes.
```

Reactive checking is cheapest. Scheduled checking is predictable. Triggered checking is expressive but requires more infrastructure.

V1 should likely favor reactive checking: a predicate is evaluated when there is a candidate execution worth verifying.

## Architecture Summary

Nested Intents require the protocol to treat validity as an executable function, not as a static field.

The architecture should make the following invariant true:

```text
No execution is valid unless the user's nested predicate passes.
```

The next chapter turns this into a selection rule: [Best Valid Execution](./05-best-valid-execution.md).

