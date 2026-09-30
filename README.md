# Lightning Foundry

An open-source, AI-native, local-first platform for building, operating and continuously
optimizing autonomous Bitcoin Lightning routing nodes.

**Running a profitable routing node is a part-time job. Foundry's goal is to make it a
decision you review, not a shift you work** — on modest hardware, with verifiable software,
intelligence that runs on your own machine, and an interface simple enough that operating a
node does not require becoming a Lightning expert first.

**Profitability is the optimization objective, not a guaranteed outcome.** Foundry measures
real economic performance, learns from its decisions, and accounts for the full cost of
running a routing node — including the costs most dashboards leave out.

## Status

**Pre-alpha. Nothing here runs a node yet.**

This repository holds the design, written before the code so the boundaries are decided
rather than discovered: the vision, the event contract and its tests, the economic
definitions, the architecture and the invariants it keeps, the threat model, and the roadmap.
The runtime, the intelligence and the installer arrive in later milestones. Do not point
anything in this repository at a node holding funds you would mind losing, because there is
nothing here to point at yet.

## What Foundry is not

- **Not a Lightning implementation.** Bitcoin Core and LND provide the Bitcoin and
  Lightning infrastructure. Foundry operates and optimizes that infrastructure; it does not
  reimplement it.
- **Not a wallet.** Foundry never holds your seed and never needs it.
- **Not financial advice.** Routing is a business with real downside. A node can lose money
  through fees, capital lockup and force closes while behaving exactly as designed.
- **Not a service.** There is no cloud component, no account, and nothing phones home. The
  one thing that can leave the machine, the public event feed, is off until the operator
  turns it on.

## Documentation

Start with [`docs/vision.md`](docs/vision.md): who Foundry is for, and what it refuses to do.
Then, roughly in this order:

| Document | What it answers |
|---|---|
| [`docs/economics.md`](docs/economics.md) | What "profitable" means here, and how outcomes get attributed to decisions |
| [`docs/event-model.md`](docs/event-model.md) | What a node says about itself, what it never publishes, and how public events travel |
| [`docs/architecture.md`](docs/architecture.md) | The components, what each may hold, and why nothing reaches LND except through Policy |
| [`docs/invariants.md`](docs/invariants.md) | The eight principles every change is judged against |
| [`docs/threat-model.md`](docs/threat-model.md) | Who we defend against, and what actually stops them |
| [`docs/ai-strategy.md`](docs/ai-strategy.md) | The two AI systems, and why neither enforces anything |
| [`docs/roadmap.md`](docs/roadmap.md) | The milestones in order, and what would make us stop |
| [`docs/integrations/oogabooga.md`](docs/integrations/oogabooga.md) | What Ooga Booga Land's Lightning Factory may show, and what publishing rebalances costs |
| [`docs/integrations/obl-payments-poc.md`](docs/integrations/obl-payments-poc.md) | How OBL's payments and Factory proof of concept lines up with the milestones |
| [`docs/decisions/`](docs/decisions/) | Choices that are expensive to revisit, and why they were made |
| [`schemas/`](schemas/), [`examples/`](examples/) | The event contract as JSON Schema, and one channel's life in both streams |

Two of these are worth reading even if you skip the rest. `docs/economics.md` defines the
reward signal every automated decision is judged against, and `docs/event-model.md` explains
why there are two event schemas rather than one — publishing a node's liquidity state in real
time tells an adversary where to attack it.

## Tests

No dependencies and no build. Node 22 or newer:

```bash
node tests/contract.test.mjs
```

## Milestones

| | Milestone | Done when |
|---|---|---|
| **M1** | Contract and simulator | a consumer can build against the event stream with no node running |
| **M2** | Foundry Node, read-only | a real LND runs under Foundry and Foundry has instructed it to do nothing |
| **M3** | Measurement and baselines | an operator can answer "was this channel worth having?" from their own data |
| **M4** | Supervised action | Foundry proposes, the operator approves, and no deterministic limit is ever breached |
| **M5** | Routing intelligence | a model beats the M3 baselines on realized sats — or it doesn't, and we publish that |
| **M6** | Autonomy within a mandate | a node runs unattended inside limits its model cannot widen |

Read-only comes before acting, and measurement comes before intelligence, deliberately.
M4 exists as its own milestone because the first time Foundry itself moves money, even with
the operator approving each action, is the riskiest step in the project.

Two tracks run alongside rather than gating the sequence: the Ooga Booga Land visualization
and community loop, and conditional hardware research. Ordering, not dates —
see [`docs/roadmap.md`](docs/roadmap.md).

## Contributing

Start with [`CONTRIBUTING.md`](CONTRIBUTING.md) for setup and what a reviewable change looks
like, and [`AGENTS.md`](AGENTS.md) for the rules code is held to: the dependency rules, the
security boundaries, the testing expectations, and the requirement that every commit is
written with AI assistance and says which model did the work.

**Contributions are open** under The Ooga Booga License — see
[`CONTRIBUTING.md`](CONTRIBUTING.md#licensing-your-contribution). There is no code yet, so the
most useful contributions today are to the design itself.

Found a vulnerability? Report it privately — see [`SECURITY.md`](SECURITY.md).

## License

[The Ooga Booga License](LICENSE), a public-domain dedication. The previous Apache-2.0 and
Unlicense contribution policy is recorded in [decision 0007](docs/decisions/0007-ooga-booga-license.md).
