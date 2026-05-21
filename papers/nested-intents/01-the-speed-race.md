# The Oldest Problem Of Computerized Finance

[Previous: Abstract](./00-abstract.md) | [Start](./README.md) | [Next: What Is A Nested Intent?](./02-what-is-a-nested-intent.md)

The oldest problem of computerized finance is not that computers made markets fast. Speed itself is not the enemy. Fast settlement, fast risk checks, fast price discovery, and fast communication are all useful.

The problem is that electronic markets gradually turned execution quality into a speed contest.

Once orders became machine-readable and venues became electronic, the central market-structure question shifted. Before computers, market participants competed through relationships, inventory, judgment, local information, and model quality. After computers, more and more of the game moved into reaction time:

- Who receives market data first?
- Who updates stale quotes first?
- Who cancels before being adversely selected?
- Who reaches the matching engine first?
- Who detects cross-venue price differences first?
- Who colocates closest to the venue?
- Who buys the better network path?
- Who writes the lower-latency engine?

This is the speed race of the machine.

It is a rational outcome of the current market design. If an order is static and the world changes around it, then the trader must update it quickly. If a stale order can be picked off, then cancellation speed becomes protection. If a quote is only safe for milliseconds, then market makers invest in infrastructure that can refresh it in microseconds. If two traders understand the same price but one arrives earlier, the earlier machine wins.

The result is not just faster finance. It is finance organized around the fear of being stale.

## The Stale Order Problem

The basic problem is simple.

A user submits an order:

```text
buy asset X at price P
```

At the moment the order is submitted, price `P` may be reasonable. But the world changes:

- The underlying asset moves.
- Volatility changes.
- Liquidity disappears.
- Funding rates shift.
- Correlations break.
- Inventory constraints change.
- An oracle updates.
- A venue pauses.
- A market maker's risk model changes.

The old order may no longer represent the user's true execution policy.

Traditional systems solve this by making the user or market maker update the order:

```text
cancel old order
submit new order
cancel old order
submit new order
cancel old order
submit new order
```

At scale, this becomes the heartbeat of electronic finance. The market is full of orders that are valid only because someone is constantly refreshing them.

The faster the world changes, the faster orders must be updated. The faster orders must be updated, the more valuable speed becomes. The more valuable speed becomes, the more competition moves from pricing intelligence into machine infrastructure.

## The First-Order Fix Created A Second-Order Problem

Computers were introduced to improve markets. They made matching, routing, settlement, and risk management more efficient. But the first-order improvement created a second-order problem.

If every participant can react faster, then the market becomes less forgiving of slow reaction. A quote that was safe for minutes becomes safe for seconds. A quote that was safe for seconds becomes safe for milliseconds. Eventually, price protection depends on continuous machine attention.

This creates several distortions.

### Capital Is Spent On Latency Instead Of Models

A trading firm can improve by building a better pricing model, or by getting closer to the matching engine. In a latency-dominated market, the second investment may be easier to monetize.

This does not mean fast infrastructure is useless. It means the market over-rewards marginal speed once the core execution problem has been framed as "who updates first?"

### Users Outsource Execution Policy To Intermediaries

Most users cannot update execution conditions at machine speed. They therefore rely on brokers, solvers, market makers, routers, and protocols to protect them.

That creates a trust problem. The intermediary has discretion. The user hopes the intermediary executes well, avoids stale fills, and does not abuse timing.

### Best Execution Becomes Hard To Verify

If the best possible execution depends on constantly changing conditions, then proving BestEx becomes difficult. Was a better quote available? Did it arrive before the cutoff? Did the solver ignore it? Was a quote valid or stale? Did the executor delay? Did it execute at the right moment?

Without a verifiable policy, BestEx becomes a story told after the fact.

### Market Quality Becomes Timing Quality

The deepest distortion is philosophical. Finance should reward better understanding of value, risk, and liquidity. But latency races often reward proximity, infrastructure, and timing games.

The machine does not ask who had the better model. It asks who arrived first.

## Why This Problem Has Existed Since Computers Entered Finance

The moment orders became electronic, finance inherited a new constraint: if a machine can execute against a stale instruction before the instruction is updated, stale instructions become dangerous.

That danger created the cancellation race.

The cancellation race created the need for speed.

The need for speed created latency markets.

Latency markets created a constant arms race in infrastructure.

That arms race has repeated across decades and venues:

- Listed equities
- Futures
- Options
- FX
- Crypto exchanges
- On-chain MEV markets
- Cross-chain bridging
- DeFi liquidation markets
- Intent and solver systems

The venue changes. The pattern remains.

Whenever execution validity is represented by a static order, and the market can change faster than the user can update that order, speed becomes the protection layer.

Nested Intents replace that protection layer.

## The Core Reframe

The current model says:

```text
To stay safe, update faster.
```

Nested Intents say:

```text
To stay safe, attach the validity condition to the order.
```

This is the shift from static orders to executable policies.

A static order is fragile because it becomes stale when the world changes.

A Nested Intent is more durable because it contains the rule that determines whether it is still valid.

Instead of asking a user to continuously update:

```text
price = newPrice
price = newPrice
price = newPrice
```

the user can publish:

```text
execute only if my verifier returns true
```

The verifier can encode price range, volatility, liquidity, portfolio exposure, oracle state, or custom model output.

The market can move. The intent can remain open. Execution simply waits until the nested verifier allows it.

## Why This Points Back To Pricing Models

If execution safety comes from speed, firms compete on engines.

If execution safety comes from verifiable policy, firms can compete on policy and models.

That means:

- Market makers can publish richer conditional liquidity.
- Users can define deeper execution constraints.
- Solvers can compete to find valid execution windows.
- Protocols can rank proposals by value net of verification cost.
- Better pricing and risk models become directly expressible as execution predicates.

This does not eliminate speed. Systems still need to observe, verify, and execute. But speed becomes a servant of the model, not the model's replacement.

The right target is not a slow market. The target is a market where speed is not the primary source of edge.

## The Link To Nested Intents

The next chapter defines the primitive:

```text
base intent + nested execution predicate = Nested Intent
```

That primitive is how finance can move from:

```text
race to update before stale execution
```

to:

```text
wait until the user's declared execution policy is satisfied
```

Read next: [What Is A Nested Intent?](./02-what-is-a-nested-intent.md).

