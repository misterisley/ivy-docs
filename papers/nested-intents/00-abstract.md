# Abstract

[Start](./README.md) | [Next: The Oldest Problem Of Computerized Finance](./01-the-speed-race.md)

Nested Intents are a new execution primitive for programmable finance. A normal intent describes a desired outcome: swap this asset, open this position, buy this option, route this order, or satisfy this quote. A Nested Intent adds a second layer: execution is only valid if a verifier confirms that the user's conditions still hold at the moment of execution.

Existing intent infrastructures, including CoW Swap, NEAR Intents, and Anoma, already support declarative execution requirements: a signed intent can include conditions that must be satisfied before settlement is valid. Nested Intents extend this model from declarative validation to programmable execution policy. In the proposed design, an intent is not limited to a static true-or-false constraint. It can reference externally verified state, off-chain computation, oracle inputs, or user-defined logic that determines whether execution is allowed and, in more advanced versions, how the intent should adapt before execution.

For example, a **nested intent** may initially define a minimum acceptable price of `10 USD`, but its nested policy could permit that threshold to adjust to `9.50 USD` at execution time if specified external inputs, oracle data, and on-chain conditions justify the change.

In its simplest form, the nested layer is price protection:

```text
execute only if:
  minPrice <= executionPrice <= maxPrice
```

In its more general form, the nested layer is a programmable execution predicate that determines whether settlement is permitted:

```text
canExecute(intent, bid, marketState, oracleState, userLogic) -> true | false
```

In its most expressive form, the nested layer can also modify the underlying intent fields before execution, subject to predefined rules and verifiable inputs:

```text
transformIntent(intent, marketState, oracleState, userLogic) -> updatedIntent
canExecute(updatedIntent, bid, marketState, oracleState) -> true | false
```

This turns an intent from a short-term instruction into a persistent mandate. The user is no longer forced to constantly cancel, replace, or update orders as the market changes. The user's execution policy travels with the intent. If the policy evaluates to true, the intent may execute. If the policy evaluates to false, execution is blocked. If the policy specifies that particular data or inputs should modify the original terms, the Nested Intent can also be transformed in flight before execution.

The larger claim of this paper is that Nested Intents attack one of the deepest failures of computerized finance: the conversion of market quality into machine speed.

Since markets became electronic, the industry has repeatedly addressed execution problems by increasing machine speed: faster data feeds, faster matching engines, faster colocated servers, faster network paths, faster cancellation, faster repricing, and faster market makers.

Crypto markets are now reproducing this same paradigm at an accelerated pace. Bitcoin introduced a settlement network with roughly ten-minute block times. Ethereum added programmable execution with block times measured in seconds. Solana pushed public-chain execution toward sub-second latency. Hyperliquid, MegaETH, and other high-performance systems now compete on throughput, latency, and their ability to support increasingly sophisticated high-frequency strategies. Although this infrastructure is technically impressive, it risks importing the same market-structure outcome seen in traditional finance: the greatest rewards accrue to firms that can afford the fastest systems, privileged infrastructure, and colocated execution.

The goal of this paper is to propose an alternative path. Rather than ending automation or market making, Nested Intents aim to reduce the structural dependence on high-frequency reaction speed as the primary mechanism for protecting sophisticated algorithmic users from stale execution.

That race produced useful infrastructure, but it also distorted the purpose of finance. Capital, talent, and engineering effort moved away from better models of value and toward increasingly narrow latency advantages. The question became less "Who understands the price better?" and more "Whose machine saw and reacted first?"

Nested Intents propose a different settlement between computers and markets.

Instead of forcing every user, solver, and market maker to constantly update orders at lightspeed, Nested Intents let them publish durable conditional liquidity. Their logic can remain valid across time. Their constraints can protect them from stale execution. Their models can define when a trade is worth doing. Their compute can be paid for explicitly.

## Standardizing Programmable Execution

A sophisticated market maker can already approximate parts of this behavior with proprietary automated market-making infrastructure. Such a firm may continuously update prices, cancel stale quotes, and use private models to achieve execution quality equal to or better than what a standardized Nested Intent system initially provides. The primary objective of Nested Intents is therefore not to improve execution for the small set of firms that already win by updating and canceling faster than everyone else.

The objective is to create a common protocol for programmable execution policy. By standardizing how conditional, adaptive intents are expressed, verified, traded, and settled, Nested Intents can make advanced execution logic available to a broader set of participants. A shared language for programmable intents improves coordination, reduces duplicated infrastructure, and allows more participants to compete from a comparable baseline of execution capability.

The expected result is a market with more meaningful competition at the model layer. If participants no longer need proprietary high-speed infrastructure merely to avoid stale execution, they can compete more directly on pricing models, risk models, liquidity provision, and execution policy. This should lead to deeper competition and, over time, fairer and more efficient pricing.

This is also a market-structure argument against durable monopolies in execution infrastructure. The long-term solution is not to rely only on central authorities to restrain the fastest actors. A healthier approach is to introduce open primitives that reduce the moat around speed itself, allowing new entrants to compete through better models and better mechanisms rather than privileged infrastructure alone.

## Further Use-Cases of Nested Intents

Nested Intents also enable a different design space for marketplaces and exchanges. By moving competition away from pure execution speed and toward programmable pricing, risk, and execution models, they make it possible to build venues such as Ivy Markets around longer-lived, model-aware liquidity rather than immediate cancellation and replacement cycles.

In the current paradigm, advanced execution often becomes a contest in specialized computer engineering: faster chips, lower-latency infrastructure, optimized networking, and increasingly sophisticated high-frequency systems. Nested Intents do not deny the value of engineering, but they shift the locus of competition back toward the financial substance of the market: pricing models, execution models, risk constraints, liquidity quality, and market statistics.

This matters not only for market makers, many of whom can already approximate these systems through proprietary infrastructure, but also for exchange design itself. If intents can remain valid for long periods because their execution conditions adapt to changing state, exchanges no longer need to force every order into immediate execution. They can support long-duration silent auctions in which participants submit conditional bids over weeks or months, and execution occurs only when the best valid outcome can be selected under the relevant constraints.

This structure gives both sides of the market greater confidence. Sellers can observe durable indications of interest while knowing that those bids remain governed by explicit validity rules. Buyers can submit bids that evolve with market conditions while remaining constrained by their own risk models and pricing assumptions. The result is a marketplace in which stale orders are less dangerous because validity is continuously defined by the nested policy rather than by the moment at which the order was first submitted.

This may also reduce some forms of market instability. During periods of stress, conventional systems often respond through rapid cancellation, widening spreads, or withdrawal of liquidity. Exchanges have historically attempted to manage these dynamics with circuit breakers, but open financial systems cannot rely on centralized halts as a complete solution. Nested Intents offer a more native mechanism: liquidity can remain present, but executable only when the conditions specified by both buyers and sellers continue to hold.


This changes Execution from:

```text
best price immediately reachable right now
```

into:

```text
best valid execution over time under the user's declared constraints
```

That is why Nested Intents matter. They do not merely add a bytes field to an intent. They introduce a new execution model:

- Persistent execution mandates
- Programmable execution guards
- Compute-priced verification
- Auditable or provable BestEx
- Solver competition constrained by user-defined policy

Nested Intents do not remove solvers. They make Solvers more accountable. Solvers can still discover opportunities, quote prices, source liquidity, run pricing models, and execute. But they cannot validly execute unless the user's nested verifier says execution is still allowed.

The first implementation can be modest: min/max price constraints checked by a PartyB passthrough contract or equivalent verifier before Symmio execution. The roadmap can then expand toward Muon-verified custom predicates, compute-aware ranking, long-term silent auctions, canonical bid sequencing, TEE attestation, and eventually ZK proofs that the executed bid was the best valid bid.

The endgame is a market where speed still matters operationally, but no longer defines the core game. Participants compete to build better pricing models, better risk models, better execution policies, and better verification infrastructure.

Nested Intents are programmable optimal stopping for financial execution.

They are a way to stop the race of the machine and return competition to the quality of the model.

## Paper Thesis

Nested Intents solve the oldest problem of computerized finance by separating execution validity from execution speed.

Computers made markets faster, but they also made markets fragile, reactive, and latency dominated. Nested Intents let users express policies that remain valid over time, so the system can wait for the right execution moment instead of forcing every participant into a constant race to update first.

## Chapter Links

- The historical problem is developed in [The Oldest Problem Of Computerized Finance](./01-the-speed-race.md).
- The primitive is defined in [What Is A Nested Intent?](./02-what-is-a-nested-intent.md).
- The BestEx model is formalized in [Best Valid Execution](./05-best-valid-execution.md).
- The compute market is explained in [The Compute Market](./06-compute-market.md).

