# Threat model

Who we are defending against, what they get if they succeed, and what actually stops them.

This document exists so the [invariants](invariants.md) can be judged. An invariant that
defends against no identified adversary is decoration; an adversary with no invariant against
them is an accepted risk that should be accepted deliberately.

Status: v0.1, written before the runtime exists. Expect it to change as real components land.

## What is worth taking

| Asset | Why it matters |
|---|---|
| On-chain funds and channel balances | Directly spendable |
| Seed and wallet keys | Total loss |
| LND macaroons | Can close channels, spend fees, move funds within limits |
| Export key and per-node credential | Can publish forged events under the node's name; cannot move funds or read balances |
| Liquidity state | Enables jamming and probing; commercially useful to competitors |
| Channel topology and peer set | Deanonymizes the operator; informs targeting |
| Economic history | Reveals strategy; reveals when the node is weak |
| The operator's trust in the numbers | If Foundry's accounting can be silently wrong, every decision built on it is wrong |

That last one is not conventional, and it is the one Foundry is most likely to get wrong.

## Adversaries

### 1. The jamming or probing adversary

**Wants:** to discover channel balances, then exhaust or grief them.

**Gets:** with balance knowledge, they can jam the depleted direction cheaply, lock up capital
in in-flight HTLCs, and make the node useless for routing without ever stealing anything.

**Mitigation:** Foundry publishes no liquidity state. The public event schema has no field
capable of expressing a balance, no per-channel identity, bucketed timing and bucketed
amounts — see [`event-model.md`](event-model.md). This is the adversary the two-schema split
exists for.

**The one exception is deliberate.** Rebalance events are published, and a rebalance means some
channel ran low: never which, never when within the hour, but it happened. What that hands
this adversary is set out risk by risk in
[`integrations/oogabooga.md`](integrations/oogabooga.md).

**Residual risk:** probing works against any node regardless of what it publishes. Foundry
reduces what it hands over for free; it does not make a node unprobeable.

### 2. The malicious or compromised channel peer

**Wants:** to force unfavorable closes, grief with stuck HTLCs, or exploit a fee policy.

**Gets:** force closes cost on-chain fees and lock funds behind a timelock. A peer who can
predict Foundry's automated responses can farm them — provoke a rebalance, collect the fee.

**Mitigation:** hard budget caps that are not policy suggestions (invariant 7). A peer that
can trigger unlimited rebalancing has found a money pump; a daily fee ceiling turns that into
a bounded annoyance. The ceiling binds the rebalancing engine too: in managed mode Policy
counts every fee Lightning Jet pays against it, and standalone, Jet's own limit is the only
bound.

**Open question:** automated responses are predictable by construction, and predictability is
exploitable. Worth deliberate thought before M4, when Foundry first acts, rather than a claim
now.

### 3. The supply chain

**Wants:** to run their code on a machine holding Lightning funds.

**Gets:** everything. This is the highest-severity, lowest-effort attack on any node software,
and the reason invariant 1 is first.

**Mitigation:** minimal dependencies; pinned versions and hashes; no install scripts; offline
reproducible builds; runtime network isolation enforced by the sandbox rather than by trusting
the package. See the Dependency Guardian in [`invariants.md`](invariants.md), including an
explicit statement of what it cannot guarantee.

A rebalancing engine running beside the node is part of this surface even when Foundry does
not manage it. Managed, it must meet invariant 1 like Foundry's own code; see
[`integrations/lightning-jet.md`](integrations/lightning-jet.md).

### 4. The update channel

**Wants:** to ship a malicious version to people who already trust Foundry.

**Gets:** the same as the supply chain, at a time of their choosing, to every operator at once.

**Mitigation:** releases are signed; installation verifies signatures and hashes before
execution; upgrades are staged and validated.

**Stated honestly:** an updater must not promise rollback it cannot deliver. Once a component
has touched irreversible wallet or channel state, "roll back" is not a thing that exists, and
software that claims otherwise teaches operators a false sense of safety.

### 5. Physical or local access

**Wants:** the seed, or the macaroons.

**Gets:** the funds, eventually.

**Mitigation:** largely outside Foundry's control and stated as such. Foundry never handles
the seed and never needs it. Macaroons are scoped, so what sits on the box is less than full
authority.

### 6. The public feed reader

**Wants:** to learn about the node from what it broadcasts.

**Gets:** everything the node publishes for the Lightning Factory in Ooga Booga Land — which is
the point of designing that surface adversarially.

**Mitigation:** the public schema is structurally incapable of carrying identity, balance or
exact timing, and the [contract tests](../tests/contract.test.mjs) assert this against the
schema itself, so the property survives future edits by people who have not read this
document.

**Residual risk:** aggregate activity still leaks *something*. A node visibly busy at certain
hours reveals a pattern. Bucketing narrows this; it does not eliminate it. Rebalance events
leak more, by decision, as adversary 1 describes. Publishing is opt-in for exactly these
reasons.

### 7. Foundry itself

**Wants:** nothing. It is buggy, not hostile — which makes it the most likely adversary on
this list to actually cost someone money.

**Gets:** whatever its credentials allow. A rebalancing loop with a bad cost model, in
Foundry's targets or in the engine's choices, can burn real sats indefinitely while every
component behaves exactly as written.

**Mitigation:** deterministic limits that are not advisory; read-only by default; a model can
never grant itself authority (invariant 6); every economic decision recorded and evaluated
against a baseline (invariant 8).

**This is the adversary the project is most likely to underestimate,** because it does not
feel like an attack. An autonomous system optimizing a slightly wrong objective is
indistinguishable from an attacker with a small budget and a lot of patience.

### 8. The delivery path

**Wants:** to publish as the node in the Factory, to read or alter its events on the way, or to
reach the node through the connection that carries them.

**Gets:** with the export key and credential, forged events under the node's name — invented
activity, invented failures, a factory that lies to its visitors. Reading events in transit
gains little, since they are public on arrival; altering them is forgery by another route.

**Mitigation:** the node pushes, listens on nothing, and reads nothing back but an
acknowledgement, so there is no connection to reach it through. Every batch is signed with a
dedicated export key that is never the Lightning key, which is what makes a forgery fail.
Around that: TLS 1.3, with the endpoint's key pinned wherever the receiver controls it; a
per-node credential; `seq` and `id` to reject replays and expose gaps; and batches on a fixed
schedule so their timing says nothing. The export holds no macaroon, so compromising it forges
public events and does nothing else. See the Delivery section of
[`event-model.md`](event-model.md).

**Residual risk:** the connection itself tells Ooga Booga Land, and anyone watching the
network, where the node is. A CDN in front of the endpoint terminates TLS, so it sees every
batch as well; the batches are public on arrival, and it cannot sign one. An operator who hides
the node behind Tor must send the export the same way, or the feed gives away the address the
node conceals. Rotating and revoking the export key and the credential are not specified yet;
they belong with the export implementation.

## Explicit non-goals

Stated so nobody mistakes silence for coverage:

- **Anonymity.** Foundry does not hide that you run a node, and running a public routing node
  is a public act.
- **Protecting against a compromised LND or Bitcoin Core.** Those are the trust anchors. If
  they are hostile, nothing above them helps.
- **Guaranteeing profit.** Not a security property, and not a promise. See
  [`economics.md`](economics.md).
- **Defending funds from the operator's own instructions.** If an operator raises a limit,
  Foundry obeys. It should make the consequence legible first.
