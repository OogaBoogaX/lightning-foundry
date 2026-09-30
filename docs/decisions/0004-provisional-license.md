# 0004. Apache-2.0 as a placeholder license

**Status:** superseded by [0007](0007-ooga-booga-license.md), 2026-09-30. The pause on
contributions was superseded earlier by [0006](0006-open-contributions.md).

## Decision

The repository is licensed Apache-2.0 until the Ooga Booga team has reviewed it; the expected
destination is the Ooga Booga License. External contributions are not accepted until the
license settles.

## Alternatives

- **Adopt the Ooga Booga License now**, ahead of the team's review.
- **No license until the review**, which reserves all rights.

## Why

The Ooga Booga License is expected to change substantially in review, so adopting it now would
mean adopting a draft. An unlicensed repository reserves all rights, which suits an open-source
project worse than a placeholder does. Apache-2.0 carries an explicit patent grant and a
well-tested disclaimer of liability, the safer default for software that will eventually move
real money. Pausing contributions keeps the change cheap, because relicensing needs the consent
of everyone who has contributed.

## Consequences

- README, CONTRIBUTING and AGENTS say contributions are paused, and why.
- The placeholder is a real license. Apache-2.0's grants are irrevocable, so anything published
  under it stays available under it after the switch.
- The license for model weights is a separate question, recorded separately — see
  [`ai-strategy.md`](../ai-strategy.md#model-artifacts).
