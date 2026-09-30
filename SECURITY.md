# Security policy

Foundry operates software that holds and moves real money. We would much rather hear about a
problem from you than from an operator who lost funds.

## Before anything else: do not risk money you mind losing

Foundry is pre-alpha. The autonomous strategies it will eventually run are **experiments**,
and experiments fail in ways their authors did not predict.

- Do not run experimental autonomous strategies against a node holding significant funds.
- Do not treat simulated results as evidence that a strategy is profitable on a real node.
  They are demonstrations. See [`docs/economics.md`](docs/economics.md).
- Use regtest, signet, or an amount you would be relaxed about losing entirely.

A routing node can lose money while every component behaves exactly as designed — through
fees, capital lockup, and force closes. That is not a bug, and no amount of correct software
prevents it.

## Reporting a vulnerability

**Report privately. Do not open a public issue for a security problem.**

Use GitHub's private vulnerability reporting:
[report a vulnerability](https://github.com/OogaBoogaX/lightningfoundry/security/advisories/new),
or Security → Report a vulnerability on the repository page. It opens a private thread visible
only to maintainers.

Helpful things to include, as far as you have them:

- What an attacker gains, not only what misbehaves.
- The smallest reproduction you can manage, and the versions involved.
- Whether it is already public or being exploited.
- How you would like to be credited, or that you would prefer not to be.

Please do not include seeds, macaroons, private keys, node pubkeys or channel points in a
report. If a reproduction seems to need them, say so and we will find another way.

## What we will do

This is a small project without a funded security team, so here is what is actually
realistic rather than a number that sounds reassuring:

- We aim to acknowledge a report within **three days**.
- We aim to give an initial assessment within **two weeks**.
- We will tell you plainly if a fix will take longer, or if we have decided not to fix
  something and why.

We ask for a reasonable window to ship a fix before public disclosure, and we will agree the
timing with you rather than impose it. If a problem is being actively exploited, disclosure
speed matters more than our schedule.

## Scope

**In scope:** anything in this repository. Especially: handling of keys and macaroons,
privilege and capability boundaries, the dependency and release verification path, the public
event feed leaking operational data, the delivery path that carries it (forging a node's
events, or reading or altering them in transit), and any way the deterministic limits
protecting capital can be bypassed — including by Foundry's own AI components.

**Out of scope,** because they are upstream projects with their own processes: vulnerabilities
in Bitcoin Core, LND or other dependencies. Please report those to their maintainers. If the
issue is that *Foundry uses them unsafely*, that is in scope and we want to hear it.

Also out of scope: the fact that routing nodes can lose money, and attacks requiring physical
access to the operator's hardware. Both are documented in
[`docs/threat-model.md`](docs/threat-model.md).

## Supported versions

None yet. There are no releases, and nothing in this repository should be run against funds.
This section becomes meaningful at M2.
