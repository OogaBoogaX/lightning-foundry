# 0006. Contributions open, under both Apache-2.0 and the Unlicense

**Status:** superseded by [0007](0007-ooga-booga-license.md), 2026-09-30. Superseded the pause
on contributions in [0004](0004-provisional-license.md).

## Decision

Contributions are open while the license is provisional. Each one is made under both the
Apache License 2.0 and the Unlicense, confirmed in the pull request template. A contribution
that cannot be offered on both terms waits until the license settles.

## Alternatives

- **Keep the pause** until the team settles the license.
- **A contributor license agreement** granting the project the right to relicense.
- **A sign-off under the Developer Certificate of Origin**, which certifies the right to
  contribute but grants no right to relicense.

## Why

There is no code yet, and a pause shuts out design review, which is what the project needs
most right now. Offering each contribution under the Unlicense as well keeps every destination
open: Ooga Booga Land describes the Ooga Booga License as a dedication with the meaning of the
Unlicense, and public-domain code can go under any license without asking anyone again.
Apache-2.0 alongside it keeps each contributor's patent grant for as long as the project stays
Apache-2.0. A contributor license agreement would do the same job with more paperwork than a
project this size can justify.

## Consequences

- Every pull request confirms both licenses; the template asks.
- A contribution can be used under the Unlicense alone, without Apache-2.0's conditions, so it
  is more open than the project around it — which is where the project expects to go anyway.
- When the license settles, a new record says what contributions are made under from then on.
