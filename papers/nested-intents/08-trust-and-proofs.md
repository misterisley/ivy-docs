# Trust, Sequencing, TEEs, And ZK

[Previous: Stopping The Speed Race](./07-stopping-the-speed-race.md) | [Start](./README.md) | [Next: Comparisons](./09-comparisons.md)

Nested Intents constrain execution, but the trust model depends on who evaluates bids, who orders proposals, and who proves the selected bid was best.

The first implementation can be trusted. The endgame should be provable.

## The Core Trust Question

If a protocol-owned solver executes Nested Intents, users must ask:

```text
Did the solver select the best valid bid?
```

The word "valid" is crucial.

The solver does not need to select the highest raw bid. It needs to select the highest-scoring bid among those that satisfy the user's nested verifier and protocol rules.

The main abuses are:

- Ignoring the best valid bid
- Delaying execution
- Executing a worse valid bid
- Censoring a valid bid
- Claiming no valid bid existed
- Including a late bid
- Executing when nested constraints did not pass
- Misreporting verification cost
- Preferencing one market maker or solver

The roadmap should progressively remove or constrain these forms of discretion.

## V0: Trusted Protocol-Owned Solver

The fastest path is:

```text
Protocol-owned solver executes Nested Intents directly.
```

The solver:

- Watches open intents
- Receives or sources bids
- Checks simple predicates
- Executes the best valid bid
- Records the result

This is acceptable for a first version if the trust assumption is explicit.

The claim should not be:

```text
The system is fully trustless.
```

The claim should be:

```text
The system introduces the Nested Intent execution model, starting with a trusted executor and moving toward provable BestEx.
```

## V1: Auditable Execution Logs

The first improvement is auditability.

Every execution should produce:

```text
intentId
intent hash
bid set hash
cutoff time
selected bid
selected bid score
rejected bid hashes
rejection reasons
verifier input hash
verifier output
verification cost
settlement calldata hash
execution transaction hash
```

This turns hidden discretion into inspectable behavior.

Audit logs do not prevent all abuse, but they make abuse easier to detect, dispute, and slash.

## V2: Canonical Sequencing

The next problem is bid ordering.

If the solver controls which bids count, the solver can say:

```text
I never saw the better bid.
```

A canonical sequencing layer changes that.

A Zellular sequencer or equivalent system can establish:

```text
Bid A arrived before cutoff.
Bid B arrived after cutoff.
The eligible bid set for this execution window is hash H.
```

Now the solver cannot easily rewrite the history of proposals.

Sequencing solves:

- Hidden late bids
- False omission claims
- Cutoff manipulation
- Bid set ambiguity

Sequencing does not by itself prove that the selected bid was best. It proves which bids were eligible to be considered.

## V3: TEE Attestation

A trusted execution environment can run the selection algorithm over the sequenced bid set.

Inputs:

```text
nested intent
sequenced bid set
oracle state
market state
scoring function version
verifier version
```

Output:

```text
selected bid
attestation that the correct algorithm selected it
```

TEE attestation can say:

```text
Given these inputs, this selected bid is the best valid bid.
```

The advantage of TEEs is practicality. They can handle complex custom logic, large bid sets, and evolving predicates earlier than ZK systems.

The tradeoff is hardware trust.

## V4: ZK Proof Of Best Valid Bid

The strongest end-state is a ZK proof.

The proof should establish:

1. The eligible bid set was committed.
2. The selected bid is in the eligible set.
3. The selected bid satisfies the nested verifier.
4. Every better-scoring bid fails validity or eligibility.
5. The execution calldata matches the selected bid.

The proof statement can be:

```text
I know:
  bidSet
  selectedBid
  verifierInputs
  scores

such that:
  selectedBid in bidSet
  selectedBid.arrivalTime <= cutoff
  nestedVerifier(intent, selectedBid, state) == true
  for every bid in bidSet:
    if score(bid) > score(selectedBid):
      nestedVerifier(intent, bid, state) == false
      or bid.arrivalTime > cutoff
      or protocolRulesPass(bid) == false
  settlementCalldata == encode(selectedBid)
```

This makes the executor a constrained actor. It may perform execution, but it cannot finalize arbitrary execution.

## Leaderless Auctions

Leaderless auctions can also help, especially when many solvers compete to execute a Nested Intent at the same time.

They solve a specific problem:

```text
No participant should get a free last look at everyone else's proposal.
```

A leaderless auction can use commitment or threshold encryption:

```text
Solvers commit proposals.
Commitment window closes.
Proposals are revealed.
Nested verifier filters invalid proposals.
Best valid proposal wins.
```

Leaderless auctions do not replace Nested Intents. They are a fair competition mechanism inside the execution window.

The clean distinction is:

```text
Nested Intents:
  define long-term execution permission.

Leaderless auctions:
  define fair short-term proposal competition.
```

For the first Ivy implementation, a protocol-owned solver may be simpler. Later, leaderless auctions can reduce privileged timing power.

## Progressive Trust Table

| Stage | Mechanism | What It Provides | Main Remaining Trust |
| --- | --- | --- | --- |
| V0 | Protocol-owned solver | Fast initial execution | Trust solver behavior |
| V1 | Audit logs | Detectable abuse | Logs may be incomplete |
| V2 | Sequencer | Canonical bid ordering | Selection still needs proof |
| V3 | TEE | Attested selection | Hardware trust |
| V4 | ZK proof | Cryptographic BestEx | Circuit complexity and cost |

## What Each Layer Proves

Sequencing proves:

```text
Which bids existed before the cutoff.
```

TEE attestation proves:

```text
The approved algorithm selected this bid over this input set.
```

ZK proves:

```text
The selected bid is mathematically the best valid bid under the committed rules.
```

PartyB passthrough enforces:

```text
The selected proposal cannot settle unless required execution checks pass.
```

Muon verifies:

```text
External or computed state used by the nested predicate.
```

## The Endgame

The solver should become less like a trusted broker and more like a constrained executor.

The desired final statement is:

```text
The executor may choose when to attempt execution,
but every completed execution proves it selected the best valid bid
under the user's Nested Intent.
```

That is provable BestEx.

Next: [Comparisons With Existing Intent Systems](./09-comparisons.md).

