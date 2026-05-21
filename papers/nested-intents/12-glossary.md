# Glossary

[Previous: Objections And Answers](./11-objections.md) | [Start](./README.md)

## Base Intent

The outer intent that defines what the user wants to happen.

Example:

```text
Buy 10 option contracts.
```

See [What Is A Nested Intent?](./02-what-is-a-nested-intent.md).

## Nested Intent

A base intent plus an execution predicate that must pass before execution is valid.

```text
NestedIntent =
  BaseIntent
  + ExecutionPredicate
  + VerificationBudget
```

See [What Is A Nested Intent?](./02-what-is-a-nested-intent.md).

## Execution Predicate

The function that determines whether execution is allowed.

```text
canExecute(intent, bid, marketState, oracleState, userLogic) -> true | false
```

Also called the nested verifier or execution guard.

## Nested Verifier

The implementation of the execution predicate. It may be a simple price check, a Muon-verified custom function, a contract call, a TEE-executed program, or a ZK-verifiable circuit.

See [The Architecture](./04-architecture.md).

## Persistent Execution Mandate

A long-lived authorization to execute under defined conditions.

Unlike a simple order, a mandate carries execution policy across time.

See [From Orders To Execution Mandates](./03-from-orders-to-mandates.md).

## Best Valid Bid

The highest-scoring bid among those that satisfy all eligibility and validity constraints.

```text
BestValidBid =
  argmax(score(bid))
  subject to:
    intent is live
    bid arrived before cutoff
    nestedVerifier(intent, bid, state) == true
    verificationCost <= computeBudget
```

See [Best Valid Execution](./05-best-valid-execution.md).

## Best Valid Execution

The execution of the best valid bid under the user's Nested Intent and the protocol's scoring rule.

This is the Nested Intent version of BestEx.

## BestEx

Best Execution. In this paper, BestEx means more than best immediate price. It means best valid execution under the user's declared constraints.

```text
BestEx is not best price now.
BestEx is best valid execution over time.
```

See [Best Valid Execution](./05-best-valid-execution.md).

## Compute Budget

The amount the user is willing to spend to verify the nested predicate.

Compute budgets prevent arbitrary user-defined logic from becoming free computation for the protocol.

See [The Compute Market](./06-compute-market.md).

## Verification Cost

The cost of evaluating a predicate.

It may include:

- Oracle fetch cost
- State read cost
- Model execution cost
- Proof generation cost
- Settlement gas cost
- Risk premium

## Compute Market

The market that prices verification work for Nested Intents.

Users pay for verification. Executors prioritize checks where expected value and compute fees justify the cost.

See [The Compute Market](./06-compute-market.md).

## SYMMIO Compute Fee

A fee, potentially denominated in SYMMIO, used to pay for verification, monitoring, oracle work, proof generation, and other execution infrastructure.

## Protocol-Owned Solver

An executor controlled by the protocol that monitors intents, evaluates proposals, selects the best valid bid, and routes execution.

Useful for V0, but trusted unless constrained by logs, sequencing, TEEs, or ZK proofs.

See [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md).

## PartyB Passthrough

An enforcement boundary that checks whether a proposed execution is allowed before Symmio settlement proceeds.

In the Nested Intent architecture:

```text
proposal
  -> nested verifier
    -> PartyB passthrough
      -> Symmio execution
```

## Muon

A verification and oracle layer that can compute or attest to external state used by nested predicates.

Examples:

- Volatility calculations
- Liquidity checks
- Oracle price bands
- Custom user logic

## Zellular Sequencer

A canonical ordering layer that can prove which bids or proposals arrived before a cutoff.

Sequencing helps prevent a solver from hiding or rewriting bid history.

## TEE

Trusted Execution Environment.

A hardware-backed environment that can attest that a specific algorithm ran over specific inputs.

In Nested Intents, a TEE can attest that the selected bid was the best valid bid over a sequenced bid set.

## ZK Proof

Zero-knowledge proof.

In this context, a ZK proof can prove that the selected bid was the best valid bid without requiring users to trust the executor.

See [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md).

## Leaderless Auction

An auction design that prevents any single auction leader from seeing proposals early, censoring bids, or preserving a privileged last look.

Leaderless auctions can be used as the short-term competition layer inside long-term Nested Intent markets.

## Last Look

The unfair ability to observe other participants' proposals before deciding whether to submit, cancel, or improve one's own proposal.

Nested Intents plus fair auction mechanisms should minimize last-look abuse.

## Long-Term Silent Auction

A market structure where a user's mandate remains open over time and solvers compete to execute only when the user's nested conditions are satisfied.

See [From Orders To Execution Mandates](./03-from-orders-to-mandates.md).

## Optimal Stopping

A decision problem about when to act.

Nested Intents are programmable optimal stopping for financial execution because the user defines the conditions under which the system should stop waiting and execute.

## Speed Race

The competition to win markets by reacting faster than others: faster data, faster cancellation, faster networking, faster execution engines.

Nested Intents do not remove the need for efficient machines, but they reduce dependence on reaction speed as the primary protection against stale execution.

See [Stopping The Speed Race](./07-stopping-the-speed-race.md).

## Stale Execution

Execution against an instruction that no longer represents the user's actual desired policy because market conditions changed.

Nested Intents defend against stale execution by checking current validity at execution time.

## Execution Sovereignty

The user's ability to define and enforce their own execution policy instead of relying entirely on solver discretion.

Nested Intents increase execution sovereignty by making user-defined predicates mandatory before execution.

