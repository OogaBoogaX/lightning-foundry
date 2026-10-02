# 0009. Circular rebalancing lives in Lightning Jet

**Status:** proposed, 2026-10-02

## Decision

Foundry does not build circular rebalancing.
[Lightning Jet](https://github.com/drneski/lightning-jet), an independent project, does it, and
Foundry manages Jet, or any engine that keeps the same contract. Foundry keeps the decisions
about capital: which channels exist, which of them need liquidity, what it is worth paying to
move it, and each channel's fee range. The engine decides how to move the liquidity.

Jet runs in one of two modes:

- **Standalone.** Jet acts on its own macaroon and its own limits, as it does today, and
  Foundry only observes.
- **Managed.** Jet keeps its own connection to LND, but the macaroon it acts with carries a
  custom caveat, so LND hands every request made with it to Foundry's Policy before running it,
  through LND's RPC middleware. Without Policy, LND refuses the macaroon.

## Alternatives

- **Build rebalancing into Foundry.** It would duplicate a working engine, for no gain in what
  Foundry exists to add.
- **Absorb Jet's code into this repository.** Jet's schedule would become Foundry's, and its
  dependencies would enter Foundry's trusted computing base as they are.
- **Jet proposes and Foundry pays.** Every payment would stay inside Foundry, but Jet would
  need a second code path for managed mode, and its probing and retries would run through
  Foundry.
- **Jet holds a payment macaroon and asks before acting.** Asking would be a courtesy: nothing
  would stop a payment Policy never saw.

## Why

Jet already does this, and its existing behavior is the baseline M3 measures against. Moving
liquidity is execution; deciding what the liquidity is worth is the judgment Foundry exists for
(see [`vision.md`](../vision.md)). LND's middleware keeps the one rule in
[`architecture.md`](../architecture.md) structural rather than polite: in managed mode no
change Jet asks for reaches LND without Policy, because LND enforces it, and Jet runs the same
code in both modes. A contract any engine can keep stops a team project from depending on one
engine and one maintainer.

## Consequences

- LND credentials live with three components, not two, and the third's can act only call by
  call, as Policy allows.
- Managed mode needs `rpcmiddleware.enable=true` in LND, and Policy must understand every LND
  call it allows; it denies any call it does not recognize. Policy is never registered as
  mandatory middleware, which would block the operator's own calls whenever Foundry is down.
- Jet 1.x runs standalone only. Jet 2.0, a rework built for managed mode, must meet invariants
  1, 3 and 4 before it runs as part of a Foundry node.
- Foundry works without any engine, and its milestones are tested against a scripted stand-in,
  never against Jet's releases.
- Rebalances an engine makes on its own are observed from the node's payments and published
  under the same rules as any other. Engines never publish.
- The internal schema's `trigger` on `rebalance.started` (`operator`, `policy`, `jet`, `model`)
  predates this decision: it names one engine, and it lets Policy start a rebalance itself. It
  is revisited with the contract's schema.
- [`integrations/lightning-jet.md`](../integrations/lightning-jet.md) sets out the contract,
  the two modes and what Policy allows. [`architecture.md`](../architecture.md),
  [`ai-strategy.md`](../ai-strategy.md), [`roadmap.md`](../roadmap.md),
  [`event-model.md`](../event-model.md), [`threat-model.md`](../threat-model.md) and
  [`vision.md`](../vision.md) change to match.
