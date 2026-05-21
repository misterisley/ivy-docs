# Comparisons With Existing Intent Systems

[Previous: Trust, Sequencing, TEEs, And ZK](./08-trust-and-proofs.md) | [Start](./README.md) | [Next: Examples](./10-examples.md)

Nested Intents should be compared carefully. They are not "intents" in the broad generic sense, and they are not just auctions, limit orders, or arbitrary calldata.

The distinguishing feature is:

```text
A persistent execution mandate with a programmable verifier and compute-priced validation.
```

## Ordinary Limit Orders

A limit order says:

```text
Buy or sell at this price or better.
```

It is simple and powerful, but narrow.

Nested Intents generalize this:

```text
Execute if my full execution policy is satisfied.
```

Price can be one condition among many.

| Feature | Limit Order | Nested Intent |
| --- | --- | --- |
| Main condition | Price | Arbitrary verifier |
| Time horizon | Often venue-specific | Persistent mandate |
| Expressiveness | Low | High |
| Compute pricing | Not native | Native requirement |
| BestEx definition | Best price | Best valid execution |
| Stale protection | Cancel/replace | Predicate validity |

## RFQ Systems

Request-for-quote systems ask market makers to provide prices for a user request.

RFQ is useful for sourcing liquidity, but it is often immediate or near-term. The user asks, market makers respond, and execution happens quickly.

Nested Intents can include RFQ-like proposals, but the mandate can remain live across time.

```text
RFQ:
  Who will quote me now?

Nested Intent:
  Who can validly execute this mandate whenever my conditions are satisfied?
```

RFQ is a quoting workflow. Nested Intents are an execution-validity primitive.

## CoW-Style Batch Auctions

Batch auctions aggregate orders and clear them together. They can improve fairness and reduce some forms of MEV.

Nested Intents are complementary.

A batch auction can clear a group of proposals, while Nested Intents define which proposals are valid for each user.

```text
Batch auction:
  How do we clear many orders together?

Nested Intent:
  Is this execution valid for this user's mandate?
```

A future system could batch Nested Intent executions, but batching is not the primitive itself.

## General Intent Protocols

General intent protocols let users express desired outcomes and let solvers find execution paths.

Nested Intents add a stronger time and validity model.

```text
General intent:
  Here is what I want.

Nested Intent:
  Here is what I want, and here is the verifier that determines whether execution is still allowed over time.
```

The difference is not syntactic. It is operational. The verifier is a first-class execution guard.

## Anoma-Style Intents

Anoma-style intent systems emphasize counterparty discovery, solving, and privacy-preserving coordination around desired state transitions.

Nested Intents can be seen as a specialized execution-policy primitive within the broader intent universe.

The focus is narrower and more financial:

- Long-term trading mandates
- Best valid execution
- Compute-priced verification
- Solver constraint
- Execution proofs

Nested Intents do not attempt to replace all intent protocols. They define a specific kind of durable financial intent.

## NEAR-Style Intents

NEAR-style intent systems focus on allowing users to express cross-chain or cross-application outcomes that solvers fulfill.

Nested Intents differ by focusing on persistence and execution validity over time.

```text
NEAR-style framing:
  I want this cross-chain outcome.

Nested Intent framing:
  This outcome may execute only when my verifier says the execution remains valid.
```

Again, the ideas can be complementary. A Nested Intent could route through a multi-chain solver network. But the nested predicate remains the core innovation.

## Paradigm Leaderless Auctions

Leaderless auctions prevent privileged last look and auction-leader abuse.

They are useful for the execution window of a Nested Intent.

```text
Nested Intent:
  Defines the long-term mandate.

Leaderless auction:
  Defines fair short-term competition among execution proposals.
```

The two mechanisms solve different problems.

Nested Intents answer:

```text
When is execution allowed?
```

Leaderless auctions answer:

```text
How do proposals compete fairly once execution is being attempted?
```

## Smart Contracts

One could ask:

```text
Isn't this just a smart contract?
```

Not exactly.

A smart contract is a general computation and settlement environment. A Nested Intent is a specific financial execution object with:

- A base trade
- A persistent execution predicate
- A compute budget
- A solver or executor workflow
- A BestEx scoring rule
- A settlement path
- Audit or proof obligations

The predicate may be implemented with smart contracts, Muon, TEEs, or ZK circuits. But the primitive is the execution mandate, not the compute substrate.

## Stop Orders And Conditional Orders

Traditional markets already have conditional orders:

- Stop loss
- Stop limit
- Take profit
- Iceberg orders
- Pegged orders
- Trailing stops

Nested Intents can be understood as a generalization of conditional orders.

The difference is that the condition can be programmable, externally verified, compute-priced, and tied to BestEx proof obligations.

```text
Conditional order:
  Execute when simple venue-supported condition triggers.

Nested Intent:
  Execute when the user's programmable verifier approves execution.
```

## The Clean Comparison

| System | Main Question | Nested Intent Difference |
| --- | --- | --- |
| Limit order | Is price acceptable? | Full execution policy can be checked |
| RFQ | Who quotes now? | Mandate can remain valid over time |
| Batch auction | How do we clear together? | Each execution is predicate-gated |
| General intents | What outcome does user want? | Validity over time is first-class |
| Leaderless auction | How do proposals compete fairly? | Defines permission before competition |
| Smart contract | What code can execute? | Financial mandate with BestEx semantics |
| Conditional order | Did trigger fire? | Programmable verifier and compute market |

## The Sharpest Distinction

Nested Intents are not a new way to pass arbitrary data.

They are a new way to bind execution to a persistent user-defined validity function.

That is why the primitive deserves its own name.

Next: [Examples And Use Cases](./10-examples.md).

