# Roadmap

Ordering, not dates. This is volunteer work on software that moves money; a schedule would
be a guess dressed as a commitment, and the temptation to hit a date is exactly how autonomous
financial software ships early.

Two rules shape the list:

1. **Every milestone ends with something someone can use.** No milestone exists only to enable
   the next one.
2. **Every acceptance criterion is checkable inside this repository.** A milestone that can
   only be completed by another team's merge is a dependency, not a milestone.

## The spine

### M1 — Contract and simulator

Establish what a Foundry node says about itself, before building anything that says it.

- Event schemas, internal and public ([`event-model.md`](event-model.md))
- Economic definitions ([`economics.md`](economics.md))
- Invariants and threat model
- A deterministic simulator producing reproducible event streams for the scenarios that
  matter: channel lifecycle, healthy routing, liquidity imbalance, rebalancing, and an
  unprofitable channel
- Signed, push-only delivery of the public stream, exercised end to end by the simulator
  against a reference receiver
- The consumer's side of the contract, written for Ooga Booga Land's Lightning Factory
  ([`lightning-factory.md`](lightning-factory.md))
- Contract tests

**Done when:** a consumer can build against the event stream without a Lightning node
existing, and the same scenario and seed produce the same stream every run.

*A visualization built on this contract lives in Ooga Booga Land and ships on its own
schedule. It is not a gate on M1.*

### M2 — Foundry Node, read-only

A minimal, verifiable, installable node that **observes and touches nothing**.

- Bitcoin Core and LND, with versions and hashes published and verified at install
- Foundry Core emitting real events against the M1 contract, and publishing the public ones
  when the operator opts in
- Lightning Jet running standalone and operator-driven, with Foundry observing its rebalances
  ([decision 0009](decisions/0009-rebalancing-in-lightning-jet.md))
- Deterministic dependency and policy validation
- A small local interface
- A local assistant that can explain what the node is doing and change nothing

**Done when:** a contributor installs Foundry on supported hardware, runs a real LND, sees
real events, and Foundry has never sent an instruction to the node.

Read-only first is deliberate. It puts the whole stack — install, verification, event
emission, visualization — under test while the blast radius is zero.

### M3 — Measurement and baselines

Make [`economics.md`](economics.md) real, and build the thing every later claim is measured
against.

- Per-channel cost accounting: revenue, rebalance cost, lifecycle cost, capital committed
- The three measures, reported and reconciled against the node's actual balance
- Holdout and staggered-rollout infrastructure, so attribution is possible later
- **Deterministic baseline strategies** — simple fee rules, threshold rebalancing, Jet's
  existing behavior — run by the operator and Jet as they are today, with Foundry recording
  their performance over real operation

**Done when:** an operator can answer "was this channel worth having?" with numbers derived
from their own node, and the baselines have published results on real data.

This milestone is missing from most projects like this, and it is the one that makes the rest
honest. Without baselines there is nothing for a model to beat, and "the AI improved things"
becomes unfalsifiable.

### M4 — Supervised action

The first time Foundry moves money. It gets its own milestone because it is the single
riskiest transition in the project, and burying it inside a larger one is how it goes wrong.

- Foundry proposes; the operator approves; Foundry executes
- Scoped LND macaroons: the component that reads is not the component that acts
- A managed rebalancing engine, Lightning Jet or another: the operator approves Foundry's
  targets, and Policy rules on every call the engine makes to reach them, recording the target
  each call served
- Deterministic limits enforced in code, not policy: daily rebalance fee ceiling, channel
  close ceiling, reserve floor, how often a channel's fee may change
- A kill switch that returns the node to operator control immediately
- Every proposal, decision and outcome recorded against M3's accounting

**Done when:** a node runs under supervision for a sustained period with zero limit
violations, and every action taken can be traced to the proposal that caused it.

### M5 — Routing intelligence

Learned models for peer classification, demand forecasting, channel recommendation and
capital allocation, evaluated against M3's baselines.

**Done when:** a model beats the deterministic baseline on **realized sats net of full
costs**, over a pre-registered evaluation window, with holdouts — or when it doesn't, and we
publish that.

A negative result here is a real contribution. A routing node sees only its own forwards, in
a non-stationary environment, with delayed and confounded rewards and no observable
counterfactuals. Those conditions favor simple heuristics, and "we tried, the heuristic won,
here is the data" is more useful to the ecosystem than a model that quietly underperforms one.

### M6 — Autonomy within a mandate

The full loop: peer discovery, channel allocation, fee policy, rebalancing through a managed
engine, channel retirement, capital redeployment — inside a deterministic mandate the model
cannot widen.

**Done when:** a node operates unattended within its mandate for a sustained period, its
economic outcome is attributable rather than merely recorded, and the operator can explain
every decision it made.

## Parallel tracks

These do not gate the spine and do not wait for it.

### Ooga Booga Land and the community loop

OBL is Foundry's reference deployment and its front door. The OBL node becomes a real
Foundry-operated node with real economic activity, which gives Foundry a live system to
operate rather than a synthetic demo. The Lightning Factory cave turns the public event
stream into something a person can watch and understand. OBL's proof of concept, mapped onto
these milestones, is in [`integrations/obl-payments-poc.md`](integrations/obl-payments-poc.md).

The loop that matters: someone meets Lightning through a game, watches a gorilla build a
channel, learns why a rebalance happened, finds Foundry, contributes, runs a node — and may
eventually connect that node back to the ecosystem.

Later, additional operators' nodes can appear in the Factory as separate rooms, each showing
only what its operator chose to publish.

**The boundary is permanent:** the cave consumes exported events. It never holds credentials,
never controls LND, and receives nothing the public schema cannot express. Foundry stays
useful with no cave at all, and OBL stays a separate project with its own schedule.

### Hardware — conditional research

Benchmark affordable commodity hardware across the real workload: Bitcoin Core, LND, Foundry
and local inference together.

**The gate:** build a dedicated appliance only if the measurements show commodity hardware is
genuinely inadequate. The default outcome is a published benchmark and a recommended
configuration, which is a useful result and much cheaper than hardware.

This sits last for a reason. Designing hardware for a workload that does not exist yet
produces hardware for an imagined workload.

## What would make us stop, or change course

Stated now, while it is cheap to be honest:

- **The baselines win.** If M5 cannot beat M3's deterministic strategies on real economics,
  Foundry ships the baselines, says so publicly, and the intelligence work becomes research
  rather than product.
- **The economics don't work.** If complete profitability is reliably negative once capital is
  accounted for, that is a finding about Lightning routing, not a failure of the software, and
  it should be published as clearly as a success would be.
- **The security model can't hold.** If deterministic limits cannot actually constrain the
  autonomous loop, M6 does not ship. Autonomy is not worth a weakened mandate.
