# Contributing

Read [`AGENTS.md`](AGENTS.md) first — it holds the rules code is judged against. This document
covers the practical parts: getting set up, what to run, and what a reviewable change looks
like.

## Licensing your contribution

Foundry uses [The Ooga Booga License](LICENSE), the same public-domain dedication as the other
OogaBoogaX projects. By contributing, you agree to release your contribution under that
license; the pull request template asks you to confirm this. The provisional Apache-2.0 and
Unlicense policy for earlier contributions is recorded in
[decision 0006](docs/decisions/0006-open-contributions.md), and the switch is recorded in
[decision 0007](docs/decisions/0007-ooga-booga-license.md). Earlier grants remain available
for versions released under them.

This applies to everyone, maintainers included, and changes only by a later decision record.

There is no code yet, so the most useful contributions today are to the design: issues,
questions, and pull requests against the docs, schemas and tests.

## Setup

Nothing to install. There is no build, no package manager, and no dependencies.

```bash
git clone https://github.com/OogaBoogaX/lightning-foundry.git
cd lightning-foundry
node tests/contract.test.mjs
```

Node 22 or newer. That is the whole toolchain.

**There is no manifest on purpose.** The implementation language is undecided — see the open
decisions in [`docs/architecture.md`](docs/architecture.md) — and adding a `package.json`
would settle it silently.

## What to run

Run the tests covering what you touched, and say in the PR what you ran and what happened.
Report failures with their output. Never claim a suite passed that you did not run.

Deterministic tests only. Anything involving time, randomness or network conditions must be
injectable, so a check measures your change rather than the machine it ran on.

## What a change looks like

Small. One idea per pull request. A change that needs three paragraphs to explain what it is
doing probably wants to be three changes.

The description should say **what you verified**, not only what you wrote. "Added X" tells a
reviewer nothing about whether it works.

Every commit carries a `Co-Authored-By` trailer naming the model that helped write it — this
project requires AI assistance rather than merely permitting it. See
[`AGENTS.md`](AGENTS.md#attribution).

## Changes that need more review

Some areas get a second pair of eyes regardless of size:

- **Security policy, limits and permissions** — anything constraining what Foundry may do
- **Dependencies** — every addition needs a written justification, a pinned version and a hash
- **Wallet, macaroon or key handling**
- **Economic execution** — anything that moves money, with or without a human approving it
- **The public event schema and its delivery** — anything published outside the operator's
  machine, and the path it travels
- **The economic definitions** — they are the reward signal everything else optimizes

If your change touches one of these, say so in the description. It speeds review rather than
slowing it.

## Economic claims

If a PR reports an economic result, use the terms in
[`docs/economics.md`](docs/economics.md) or define new ones explicitly. State the baseline,
the period, and which costs are included.

Simulated results are demonstrations, never evidence that a strategy is profitable on a real
node.

## Decisions

Choices that are expensive to revisit — the implementation language, the license for model
weights, the shape of the public feed — belong in [`docs/decisions/`](docs/decisions/) as a
short record: what was decided, what the alternatives were, and why. Written at the time,
while the reasoning is still in someone's head.

A decision record is not a design document. Half a page is usually right.

## Reporting a vulnerability

Privately, never in a public issue. See [`SECURITY.md`](SECURITY.md).
