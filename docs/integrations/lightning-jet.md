# Lightning Jet

Lightning Jet optimizes liquidity. Lightning Foundry optimizes the node. Ooga Booga Land makes
the infrastructure visible and understandable.

[Lightning Jet](https://github.com/drneski/lightning-jet) is an independent, open-source
circular rebalancer for LND. Foundry does not build rebalancing. It manages Jet, or any engine
that keeps the contract below, and keeps the decisions about capital for itself.
[Decision 0009](../decisions/0009-rebalancing-in-lightning-jet.md) records why.

## Who decides what

| | Foundry | The engine |
|---|---|---|
| Channels and peers | Opens, sizes, closes and replaces them | — |
| Capital | Allocates it, including swaps, bought inbound liquidity and splicing | — |
| Which channels need liquidity | Sets a target for each, and what it is worth paying to meet it | — |
| How to move it | — | Routes, amounts, timing and cost |
| Fees | Sets each channel's fee range from its economics | Proposes changes within the range to steer liquidity |
| Limits | Daily rebalance fee ceiling, how often a fee may change, reserve floor | Its own, when standalone |
| Public events | Publishes them, per [`event-model.md`](../event-model.md) | Never publishes |

Foundry decides where capital goes and what liquidity is worth. The engine finds the cheapest
way to get it there.

## Two modes

### Standalone

Jet runs on an LND node as it does today, with its own macaroon and its own limits, and
installing it is the operator's approval. That macaroon can send any payment, because LND's
permissions alone cannot limit a macaroon to circular payments. Jet's own limits are then the
only ones, so they should be deterministic code, not a model's judgment.

If Foundry runs on the same node, it observes Jet's rebalances from the node's payments,
accounts for them, and does nothing else. M2 and M3 run this way.

### Managed

Jet keeps its own connection to LND, and LND makes it ask Foundry first:

- **The caveat.** The macaroon Jet acts with carries a custom caveat. LND hands every request
  made with it to the middleware registered for the caveat, which is Foundry's Policy, before
  running it.
- **Policy decides.** It approves or rejects each request, and it sees the response, so it
  counts what each payment actually cost.
- **It fails closed.** If Policy is not registered, LND refuses the macaroon outright. If
  Policy does not answer in time, the call fails.
- **Never mandatory.** Policy is not registered with `rpcmiddleware.addmandatory`. That would
  block every RPC call while Foundry is down, the operator's own included, and break
  [invariant 7](../invariants.md#7-failure-isolation).
- **Reads go around it.** Jet reads with a second macaroon that is read-only and carries no
  caveat.
- **One code path.** Jet calls LND's own API in both modes. The macaroon the operator gives it
  decides which mode it runs in.

Managed mode needs `rpcmiddleware.enable=true` in LND, which is off by default.

### What Policy allows

An allowlist. Anything not on it is denied, including any call Policy does not recognize, so a
new LND call stays closed until Policy learns it.

- **Circular payments and probes.** The route ends at the node itself, uses channels in
  Foundry's targets, stays within the target amount, and its fee limit fits in what is left of
  the daily ceiling.
- **Fee changes.** Within the channel's fee range, and no more often than the limit allows. A
  fee that changes often floods gossip, gets throttled by other nodes, and fails some payments
  made against the old fee.
- **Never.** Opening or closing channels, on-chain sends, and paying any invoice that is not
  the node's own.

The daily ceiling is also what bounds fee farming, where a peer who can predict rebalances
provokes them to collect the fees ([`threat-model.md`](../threat-model.md)). Standalone, Jet's
own limit is the only bound.

Under supervision, from M4, a person approves Foundry's targets, and Policy checks every call
the engine makes against them. Nobody approves single payments; the middleware's timeout is
measured in seconds.

## The contract

Foundry owns the contract and versions it, with conformance tests that any engine can run. An
engine that passes them can be managed. The schema is not written yet; this is what it
carries.

**From Foundry to the engine:**

- **Targets.** For each channel, the direction and amount of liquidity wanted, and the most it
  is worth paying to move it.
- **Fee ranges.** For each channel, the band the engine may move its fee within.
- **Withdrawals.** When a target goes away, for instance because Foundry has decided to close
  the channel, the engine drops any work pending on it.

**From the engine to LND, through Policy:** payments and fee changes, each checked against the
targets. The engine learns Policy's verdict from the call itself.

**From the engine to Foundry, optionally:** forecasts and cost estimates, labeled derived. They
inform Foundry's targets; they never change a limit.

The engine uses the definitions in [`economics.md`](../economics.md), and Foundry measures
every rebalance against them, whichever engine made it.

## Models

Each project may build its own model: Jet's for short-term liquidity, Foundry's for topology,
channels and capital. They share definitions and evaluation methods, not weights. Neither
model gets authority:

- **Models propose; deterministic code decides.** No model holds a credential, approves an
  action or changes a limit, in either project.
- **In managed mode, Foundry's Policy is that code.** Standalone, Jet's own limits are.
- **Training data stays local**, as in
  [`ai-strategy.md`](../ai-strategy.md#training-data-and-privacy). An engine that learns from a
  node learns from that node, on that node.

## Jet's versions

- **Jet 1.x** runs standalone only, as it does today. Its behavior is one of the baselines M3
  measures.
- **Jet 2.0** is a rework built for managed mode. To run as part of a Foundry node it must meet
  invariants [1](../invariants.md#1-minimal-trusted-computing-base),
  [3](../invariants.md#3-network-isolation) and [4](../invariants.md#4-verifiable-software):
  pinned and hashed dependencies with no install scripts, no network path beyond the node it
  rebalances, and verified artifacts. Jet 1.6.0 does not meet them yet: its dependencies float,
  it runs a post-install script, two of its modules run install scripts for native code, and it
  ships a Telegram client. Alerts can stay, as a separate notifier that holds no credential.

## Without Jet

Foundry works without any engine. Rebalancing is then the operator's job, or another engine's,
and Foundry observes and accounts for it as it does for Jet standalone. Foundry's milestones
never wait on a Jet release: its tests run the contract against a scripted stand-in engine, as
rule 2 of [`roadmap.md`](../roadmap.md) requires.

## Where the work lives

- **Lightning Jet** owns the engine, its model, standalone mode, Jet 2.0, and its side of the
  contract.
- **Foundry** owns the contract, Policy's rules for it, the conformance tests and the stand-in
  engine.

Foundry sets a target. Jet decides how to meet it. Neither repository decides the other's half.
