# Objections And Answers

[Previous: Examples](./10-examples.md) | [Start](./README.md) | [Next: Glossary](./12-glossary.md)

Nested Intents are easiest to misunderstand if they are reduced to a data field, a solver feature, or a fancy limit order. This chapter answers the main objections.

## Objection 1: "Isn't This Just A Bytes Field?"

No.

A bytes field can carry arbitrary configuration, but configuration is not an execution model.

Nested Intents require:

- Persistent attachment of the predicate to the intent
- Mandatory verification before execution
- Economic pricing of verification compute
- Solver or executor constraint
- Best valid bid selection
- Audit, attestation, or proof obligations

Without those properties, the bytes are just data.

With those properties, the bytes encode a programmable execution guard.

## Objection 2: "Isn't This Just A Smart Contract?"

No.

A smart contract is a general-purpose execution environment. A Nested Intent is a financial object with a specific market-structure role.

It defines:

```text
what the user wants
when execution is valid
who pays for verification
how proposals are ranked
how settlement is constrained
how execution can be audited or proven
```

A smart contract may implement part of this. So may Muon, a TEE, or a ZK circuit. But the primitive is the mandate, not the compute substrate.

## Objection 3: "Isn't This Just A Limit Order?"

The first version can look like a limit order because min/max price is the simplest predicate.

But a limit order is a special case:

```text
nestedVerifier(intent, bid, state) =
  minPrice <= executionPrice <= maxPrice
```

Nested Intents generalize this to arbitrary verifiable execution policy:

```text
nestedVerifier(intent, bid, state, userLogic) -> true | false
```

The price-only case is the bootstrap path, not the boundary of the idea.

## Objection 4: "Does This Remove Solvers?"

No.

Nested Intents need solvers, market makers, executors, keepers, verifiers, and settlement agents.

The difference is that solvers become constrained by user policy.

Without Nested Intents:

```text
Solver decides whether execution is acceptable.
```

With Nested Intents:

```text
Solver proposes execution.
User's verifier decides whether execution is valid.
```

The solver still competes on liquidity, price, routing, monitoring, and execution quality.

## Objection 5: "Does This Make Markets Slow?"

No.

Nested Intents do not slow down infrastructure. They reduce dependence on reaction speed as the main safety mechanism.

Fast systems are still useful for:

- Monitoring
- Verification
- Sequencing
- Proof generation
- Settlement

The goal is not slowness. The goal is to replace:

```text
protect users by updating faster
```

with:

```text
protect users by checking validity at execution time
```

## Objection 6: "What If Verification Is Too Expensive?"

Then the intent must pay for it, or it should not be evaluated.

This is why the compute market is essential.

```text
verificationCost <= computeBudget
```

If a user wants to run a complex model, they can. But they must fund the compute. Executors can prioritize predicates where expected value justifies evaluation.

This is not a weakness. It is the mechanism that prevents abuse.

## Objection 7: "What If The Solver Lies?"

That depends on the stage of the roadmap.

V0:

```text
The solver is trusted.
```

V1:

```text
The solver produces audit logs.
```

V2:

```text
A sequencer establishes which bids were eligible.
```

V3:

```text
A TEE attests that the selection algorithm was followed.
```

V4:

```text
A ZK proof proves the selected bid was the best valid bid.
```

The long-term goal is not to ask users to trust the solver. It is to make the solver provably constrained.

## Objection 8: "Can Users Submit Malicious Or Infinite Logic?"

They should not be able to execute unbounded logic.

Custom predicates need constraints:

- Code hash
- Version
- Input schema
- Max runtime
- Max verification cost
- Allowed oracle dependencies
- Failure behavior
- Upfront compute budget

The system should encourage standard predicates and allow custom predicates only through bounded execution environments.

## Objection 9: "Who Decides The Best Bid?"

The protocol must define a deterministic scoring rule.

For V1:

```text
best price among valid bids
```

For later versions:

```text
best score among valid bids
```

where:

```text
score =
  price improvement
  + compute fee
  + solver quality
  - verification cost
  - risk penalty
```

The scoring rule should be known before proposals are ranked. Otherwise the executor can hide discretion in subjective scoring.

## Objection 10: "Can A High Compute Fee Buy Execution?"

No.

A compute fee can buy evaluation priority. It cannot make an invalid bid valid.

The order is:

```text
1. Check eligibility.
2. Check nested verifier.
3. Score valid proposals.
4. Execute best valid proposal.
```

If the verifier fails, the bid is invalid regardless of fee.

## Objection 11: "What If No Valid Proposal Exists?"

Then nothing executes.

This is not a failure. It is correct behavior.

Nested Intents are designed to protect users from invalid execution. An idle mandate is better than a bad fill.

## Objection 12: "Why Is This The Oldest Problem Of Finance?"

Strictly, finance had execution problems before computers. But the specific problem addressed here is the oldest problem of computerized finance:

```text
Once orders became electronic, stale instructions became exploitable at machine speed.
```

The industry responded by making machines faster.

Nested Intents respond by making instructions more expressive and self-protecting.

## Objection 13: "Why Would Market Makers Use This?"

Because it lets them expose conditional liquidity without being as exposed to stale execution.

A market maker can publish:

```text
I will trade while my risk model says this is valid.
```

That can be more capital efficient than continuously updating a simple order.

## Objection 14: "Why Would Users Use This?"

Because users can define execution quality more precisely.

Instead of trusting an intermediary to interpret their preferences, they can encode:

- Price limits
- Volatility limits
- Liquidity requirements
- Portfolio constraints
- Solver constraints
- Oracle conditions
- Custom model output

This gives users execution sovereignty.

## Objection 15: "What Is The Simplest Useful Version?"

The simplest useful version is:

```text
Base intent:
  desired trade

Nested predicate:
  minPrice <= executionPrice <= maxPrice

Executor:
  protocol-owned solver

Enforcement:
  PartyB passthrough check before Symmio execution

Audit:
  execution logs
```

This version is small enough to ship and strong enough to prove the concept.

## Final Answer To The Skeptic

Nested Intents are not a new syntax for orders.

They are a new market primitive:

```text
persistent execution mandates
with programmable validity
and priced verification compute
```

They shift competition away from constant order replacement and toward better execution policy.

Next: [Glossary](./12-glossary.md).

