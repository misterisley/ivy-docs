# Nested Intents

## A Paper On Programmable Best Execution

Nested Intents are a proposed financial primitive for turning short-lived trading instructions into persistent, programmable execution mandates.

The central thesis is simple:

> Finance has spent the computer age trying to win execution by making machines faster. Nested Intents propose a different path: make the user's execution policy durable, verifiable, and economically priced, so competition moves back toward better pricing models instead of faster engines.

This paper is split into cross-connected chapters. Read it linearly, or jump directly into the section that matches the question you are trying to answer.

## Reading Map

1. [Abstract](./00-abstract.md)
2. [The Oldest Problem Of Computerized Finance](./01-the-speed-race.md)
3. [What Is A Nested Intent?](./02-what-is-a-nested-intent.md)
4. [From Orders To Execution Mandates](./03-from-orders-to-mandates.md)
5. [The Architecture](./04-architecture.md)
6. [Best Valid Execution](./05-best-valid-execution.md)
7. [The Compute Market](./06-compute-market.md)
8. [Stopping The Speed Race](./07-stopping-the-speed-race.md)
9. [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md)
10. [Comparisons With Existing Intent Systems](./09-comparisons.md)
11. [Examples And Use Cases](./10-examples.md)
12. [Objections And Answers](./11-objections.md)
13. [Glossary](./12-glossary.md)

## One Sentence

A Nested Intent is a base intent plus a persistent execution predicate:

```text
canExecute(intent, bid, marketState, oracleState, userLogic) -> true | false
```

The base intent says what the user wants.

The nested predicate says when execution is still allowed.

## The Claim

Traditional electronic markets compress financial competition into latency. Whoever sees the opportunity, updates the quote, and reaches the matching engine first often wins. That created decades of investment into colocation, microwave towers, FPGA trading systems, kernel bypass, custom networking, and ever-shorter reaction times.

That race is not the same thing as better finance. It is often just a race to be first to react.

Nested Intents change the unit of competition. Instead of asking every participant to keep replacing stale orders faster than everyone else, they let participants publish long-lived conditional liquidity. The logic that protects the user travels with the intent. Execution is allowed only when the verifier says the user's conditions still hold.

This makes Best Execution a programmable optimal stopping problem:

```text
BestValidExecution =
  best proposal
  subject to:
    intent is live
    proposal arrived before cutoff
    nestedVerifier(intent, proposal, state) == true
    computeBudget >= verificationCost
```

## The Practical Starting Point

The first version can be simple:

```text
minPrice <= executionPrice <= maxPrice
```

Even this simple nested condition matters. It lets a user leave an execution mandate open without trusting the executor to avoid stale or hostile fills.

Later versions can support richer verifiers:

- Volatility constraints
- Liquidity constraints
- Funding-rate constraints
- Portfolio exposure constraints
- Oracle-verified external conditions
- Solver reputation constraints
- User-supplied pricing models
- Muon-verified custom logic
- Adaptive rules that modify the intent when verified conditions change

## The Economic Constraint

Arbitrary logic cannot be free. A user could submit a verifier that is trivial, or they could submit a model that is absurdly expensive to run. Nested Intents therefore require a compute market.

Each intent carries a compute budget, potentially denominated in SYMMIO. Executors and verifiers can prioritize work by expected value net of verification cost:

```text
ExecutionPriority =
  ExpectedExecutionValue
  + ComputeFee
  - VerificationCost
  - RiskPenalty
```

This makes expressive execution policy viable without turning the protocol into a free computation sink.

## The Endgame

At first, a protocol-owned solver can execute Nested Intents and produce audit logs. Over time, the executor should become more constrained:

- A sequencer can prove which bids arrived before the cutoff.
- A TEE can attest that the solver ran the correct selection algorithm.
- A ZK circuit can prove the selected bid was the best valid bid under the user's verifier.

The endgame is provable BestEx:

> Every execution must prove that it selected the best valid bid under the user's Nested Intent.

## Core Links

- For the market-structure motivation, read [The Oldest Problem Of Computerized Finance](./01-the-speed-race.md).
- For the formal definition, read [What Is A Nested Intent?](./02-what-is-a-nested-intent.md).
- For the execution algorithm, read [Best Valid Execution](./05-best-valid-execution.md).
- For the compute model, read [The Compute Market](./06-compute-market.md).
- For the trust roadmap, read [Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md).

