# AGENTS.md

Guidelines for AI agents and human collaborators working on Lightning Foundry. Read this
before changing anything. The standard: production-quality code with a minimal trusted
computing base, correct, measured, auditable, and verified before it lands.

Foundry manages real money on other people's nodes. Treat every change as though it will
run unattended against a funded routing node, because eventually it will.

## What this is

Foundry builds, operates and optimizes autonomous Bitcoin Lightning routing nodes. It does
**not** implement the Lightning protocol: Bitcoin Core and LND provide the Bitcoin and
Lightning infrastructure. Foundry provides everything needed to operate that infrastructure
and make it economically accountable.

Six functional areas, one system: node operation, intelligence, automation, security,
factory (the events a visualization is built on), education.

## Ground rules

- Read a file before editing it. Edit what is on disk, not what you assume.
- **Every dependency must justify its existence.** A new dependency needs a written reason
  in the PR, a pinned version and hash, and no install scripts. Prefer the standard library.
  Prefer copying twenty lines over adding a package.
- **The core makes no arbitrary outbound connections.** Talking to a local Bitcoin Core or
  LND is the job. Reaching the internet is not. Optional integrations are isolated, opt-in,
  and never on the path that manages funds.
- **No telemetry, ever.** Operational data stays on the operator's machine.
- **Never commit secrets or key material.** No seeds, macaroons, certificates, wallet files,
  export keys or `.env` contents, and no real node pubkeys or channel points — not in code,
  tests, fixtures, examples, or commit messages. Fixtures use values that are obviously
  synthetic, like the `02aaaa…` pubkey in `examples/`. Test scripts must not embed absolute
  paths, user names or machine names.
- **Never test against real funds.** Fixtures, simulators, regtest and signet only. A change
  that cannot be exercised without mainnet money is not ready.
- Smallest change that works. No refactors, reformatting or renames the task does not require.
- Distinguish **observed** from **derived** throughout. A fact read from LND and a number
  Foundry inferred are different things and must stay labeled as different things.
- Keep the AI out of enforcement. A model may propose, recommend or explain. A deterministic
  check decides. Security policy, capital limits and dependency rules are never enforced by
  a model's judgment.

## Contributing

Foundry uses fork and pull request. Branch from `main` in your own fork, open a PR against
`OogaBoogaX/lightningfoundry`, and describe what you verified. Contributions are made under
[The Ooga Booga License](LICENSE); see
[`CONTRIBUTING.md`](CONTRIBUTING.md#licensing-your-contribution).

Changes touching security policy, limits or permissions, dependencies, wallet, macaroon or key
handling, economic execution with or without a human approving it, the economic definitions,
or the public event schema and its delivery require additional review.

## Testing

New behavior needs a check. Run the tests covering what you touched before finishing, and
say in the PR what you ran and what the result was. Do not claim a suite passed that you did
not run; report failures with their output.

Tests are deterministic. Anything involving time, randomness or network conditions must be
injectable, so a check measures behavior rather than the host machine.

## Economics

Profitability is the optimization objective, not a guaranteed outcome. Any claim that a
strategy is profitable must say against which baseline, over what period, and net of which
costs. See `docs/economics.md` — that document defines the terms, and a PR that reports
economic results uses those terms or defines new ones explicitly.

Simulated results are demonstrations. They are never evidence that a strategy is profitable
on a real node, and PRs must not present them as such.

## Attribution

Foundry is AI-native, and that is a requirement rather than a description: **every commit is
written with AI assistance.** Which assistant is yours to choose — Claude, ChatGPT, or
anything else that does the job.

Because every commit is assisted, **every commit carries a `Co-Authored-By` trailer** naming
the model that did the work, using the vendor's own noreply address where one exists:

```
Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
```

A commit without a trailer is incomplete, the same as a commit without a message.

Name the specific model rather than a generic placeholder. A reviewer judging how a change
was produced learns nothing from "an AI", and knowing which model wrote a thing is useful
later, when a pattern of mistakes turns out to be a pattern. More than one trailer is fine
when more than one was used.

A pull request whose code was substantially AI-generated says so in its description and
names the tool. A footer line is the conventional form:

🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Before you finish

1. Tests covering your change pass, and you have said which ones.
2. No new dependency without a written justification, a pinned version and a hash.
3. No secrets, key material, absolute paths or machine names anywhere in the diff.
4. Observed and derived values are still distinguishable.
5. Nothing in the core reaches the network that did not before.
6. Your change is the smallest that does the job, and it reads like the code around it.
