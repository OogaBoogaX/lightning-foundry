# Invariants

Eight principles that constrain every contribution. They are not aspirations; a change that
violates one is wrong even if it works, and a change that cannot be made without violating one
needs the invariant changed first, in its own discussion.

Each entry says what it forbids in practice, because a principle nobody can apply is decoration.

## 1. Minimal trusted computing base

Every dependency, daemon, privilege and network capability must justify its existence.

**In practice:** a new dependency needs a written reason, a pinned version, a hash, and no
install scripts. Prefer the standard library. Prefer copying twenty lines over adding a
package. A process that does not need a privilege does not get it.

**Why:** everything in the TCB can move the operator's money or leak their keys. Code you did
not review, running as a user with more rights than it needs, is the entire attack surface of
most compromises.

## 2. Local-first intelligence

Inference, routing history, economic data and node control run on the operator's machine.
No required cloud service, ever.

**In practice:** no feature may depend on a remote API to function. An optional integration
that improves something is acceptable; a feature that stops working offline is not.

## 3. Network isolation

The core performs no arbitrary external communication. Talking to a local Bitcoin Core or LND
is the job; reaching the internet is not.

**In practice:** the components that manage funds have no outbound network path beyond the
node they operate. Optional integrations are separate processes, explicitly enabled, and never
on the path that moves money. This is enforced at the sandbox and network layer, not by
reviewing code for HTTP calls.

The one integration today is the export of the public event feed: a separate component, off
by default, holding no LND credential, that pushes signed batches to one configured endpoint
and accepts no connections. See [`event-model.md`](event-model.md).

## 4. Verifiable software

Every release identifies and verifies the exact Bitcoin Core, LND, Foundry and model artifacts
it uses.

**In practice:** releases publish versions, hashes and signatures. Installation verifies them
before running anything. "Probably the right version" is a failure.

On a node Foundry adopts rather than installs, it cannot vouch for the Bitcoin Core and LND it
did not install. It checks their versions against its compatibility list, records what it
found, and says so ([decision 0010](decisions/0010-adopt-or-install.md)).

## 5. No phone-home telemetry

Operational data stays local. No vendor telemetry, no hidden reporting, no mandatory analytics.

**In practice:** there is no opt-out, because there is nothing to opt out of. Contributing node
data to research is a separate, explicit, reviewed act, and running Foundry never requires it.

The public event feed is not telemetry. It is off until the operator turns it on, it goes only
where the operator points it, and its schema cannot carry identity or balances.

## 6. Deterministic security

AI cannot override operator-defined capital limits, security policy or dependency
restrictions.

**In practice:** a model proposes, recommends and explains. A deterministic check decides.
Every limit that protects money is evaluated by code whose behavior can be read, tested and
predicted — never by a model's judgment, and never by a model's judgment about whether to
consult the check.

## 7. Failure isolation

Bitcoin Core and LND continue functioning safely if Foundry's AI or management components
fail.

**In practice:** "keeps running" is not sufficient, because isolation is about blast radius,
not uptime. Foundry holds credentials that can close channels and spend fees, so:

- Capabilities are scoped. LND's macaroons are baked per capability; the component that reads
  is not the component that acts.
- Read-only is the default. Acting requires a separately held, narrower credential.
- Budgets are hard limits, enforced deterministically: a ceiling on rebalance fees per day, a
  ceiling on channel closes per day, a floor on reserves.
- A crashed, wedged or compromised Foundry must leave the node in a state the operator can
  take over manually.

## 8. Economic accountability

Every autonomous economic decision is recorded, evaluated and attributable.

**In practice:** recording is necessary and not sufficient. Attribution requires a design that
could have shown the opposite result — holdouts, staggered rollout, a pre-registered
evaluation window. See [`economics.md`](economics.md). A claim that a decision caused an
outcome, derived from logs alone, is a story about noise.

---

## The Dependency Guardian

This deserves its own section because it is where invariants 1, 3 and 6 meet.

When Foundry's AI-assisted build system proposes or modifies an installable package, a model
assesses whether each dependency is necessary and proposes alternatives. A deterministic
validator then independently checks the result. **The AI helps enforce the philosophy; final
enforcement never depends on an AI judgment.**

Being precise about what that validator can actually guarantee matters, because a security
control that over-promises is worse than one that is honestly scoped.

**Mechanically enforceable, and therefore the Guardian's job:**

- The dependency appears on the approved manifest.
- Version pinned; artifact hash matches.
- Source provenance and signatures verify.
- No install or post-install scripts.
- Builds are offline and reproducible: the same inputs produce the same artifact.
- At runtime, the process cannot reach the network, enforced by the sandbox and the host's
  network policy rather than by inspecting the package.

**Not mechanically enforceable, and therefore a review question rather than a gate:**

- Whether a package's *capabilities* are acceptable. Any package can shell out, `eval`, or
  open a socket at runtime; static capability analysis for npm and PyPI is an unsolved
  problem, and claiming otherwise would put a false floor under everything above it.

The honest guarantee: Foundry can prove you are running exactly the artifact you approved, and
can prevent it from reaching the network. It cannot prove that artifact is benign. That is why
invariant 1 exists — the strongest available control on a dependency's behavior is not having
it.
