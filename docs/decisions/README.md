# Decisions

Choices that are expensive to revisit, written down when they are made, while the reasoning is
still in someone's head. A decision record is not a design document; half a page is usually
right.

## Format

One file per decision, numbered in order: `NNNN-short-title.md`.

- **Status** — proposed, accepted, or superseded by a later record, which links back.
- **Decision** — what was decided, in a sentence or two.
- **Alternatives** — what else was on the table.
- **Why** — the reasoning that won.
- **Consequences** — what it makes easier, harder or impossible.

Records are not edited after acceptance, except to mark them superseded. Changing a decision
means writing a new record.

## Records

| | Decision | Status |
|---|---|---|
| [0001](0001-two-event-schemas.md) | Two event schemas, not one with a redaction filter | accepted |
| [0002](0002-publish-rebalance-events.md) | Rebalance events are published, under constraints | accepted |
| [0003](0003-random-public-ids.md) | Public event ids are random UUIDv4 | accepted |
| [0004](0004-provisional-license.md) | Apache-2.0 as a placeholder license | superseded by 0007 |
| [0005](0005-ai-assisted-commits.md) | Every commit is written with AI assistance and names the model | accepted |
| [0006](0006-open-contributions.md) | Contributions open, under both Apache-2.0 and the Unlicense | superseded by 0007 |
| [0007](0007-ooga-booga-license.md) | Adopt The Ooga Booga License | accepted |

## Waiting for a record

- The implementation language — see [`architecture.md`](../architecture.md#open-decisions).
- The license for model weights — see [`ai-strategy.md`](../ai-strategy.md#model-artifacts).
