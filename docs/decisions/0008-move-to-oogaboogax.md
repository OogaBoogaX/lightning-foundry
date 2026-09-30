# 0008. The repository lives at OogaBoogaX/lightningfoundry

**Status:** accepted, 2026-09-30

## Decision

Foundry's repository moved from `drneski/lightning-foundry` to the OogaBoogaX organization by
transfer, then was renamed `lightningfoundry` to match the organization's repository names,
which are lowercase and have no hyphens. The project is still called Lightning Foundry.

## Alternatives

- **Stay on the founder's account.**
- **Move, but keep the hyphenated name** `lightning-foundry`.

## Why

The team adopted Foundry, and OogaBoogaX is where its projects live, including Ooga Booga Land,
which Foundry integrates with. An organization lets maintainers share the repository's
permissions rather than depend on one account. The rename keeps it consistent with the
organization's other repositories.

## Consequences

- The docs name `OogaBoogaX/lightningfoundry` wherever they point at the repository: the
  vulnerability report form, the contribution instructions and the pull request template.
- GitHub redirects both old names, `drneski/lightning-foundry` and
  `OogaBoogaX/lightning-foundry`, but only until a repository takes either name again. Nothing
  should depend on them, and existing clones should point `origin` at the new address.
- The schemas' `$id`s use the domain `lightning-foundry.org`, which the move does not settle.
  Where they should point is a separate question.
