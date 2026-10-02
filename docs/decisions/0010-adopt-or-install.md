# 0010. Foundry adopts compatible nodes as well as installing its own

**Status:** proposed, 2026-10-02

## Decision

Foundry runs either way: installed as a package with Bitcoin Core and LND, or adopted onto a
compatible node that is already running. Compatibility is a published list of the Bitcoin
Core and LND versions, and the settings, that Foundry needs.

## Alternatives

- **Install only.** Every node is one Foundry built, and invariant 4 holds in full.
- **Adopt only.** Foundry never ships Bitcoin Core or LND, and a newcomer has to assemble a
  node before Foundry can help.

## Why

Many of the operators Foundry is for already run a node, and asking them to rebuild it to try
Foundry is asking them not to. Adoption opens a path that starts where they are: an LND node,
then Lightning Jet running standalone, then Foundry managing the node and Jet with it. The
packaged install keeps a minimal, verified stack for anyone starting fresh.

## Consequences

- On an adopted node Foundry did not install Bitcoin Core or LND, so
  [invariant 4](../invariants.md#4-verifiable-software) covers only what Foundry ships. Foundry
  checks the node's versions against the compatibility list, records what it found, and says
  so.
- An adopted node may run other services beside LND. They stay outside Foundry's trusted
  computing base, and outside its guarantees.
- The compatibility list includes what managed rebalancing needs, such as LND's RPC middleware
  ([decision 0009](0009-rebalancing-in-lightning-jet.md)).
