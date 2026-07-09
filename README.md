# Mythic-Edge-Feature-Expansions

Planning repo for future Mythic Edge feature ideas.

This repo is where feature ideas are parked, organized, compared, and promoted.
It is not the default implementation repo for parser behavior, analytics
behavior, private Fable internals, security remediation, or production UI
changes.

## Organization

Feature work is organized into:

- feature-family trackers;
- source/inspiration trackers;
- shared dependency trackers;
- child issues;
- promotion packets into the owning implementation repo.

The repo taxonomy is documented in
[`docs/feature_expansion_taxonomy.md`](docs/feature_expansion_taxonomy.md).

Before a feature family is promoted to implementation, it must satisfy the
required child issue floor:

1. Product thesis and V1 scope.
2. Required parser facts / data-source matrix.
3. Privacy and fair-play risk review.
4. Data model / contract.
5. UI or workflow sketch.
6. Implementation destination decision.
7. MVP implementation issue in the owning repo.
8. Validation and non-claim checklist.

Trackers in this repo are planning authority only. They do not authorize
implementation, parser behavior changes, production behavior, external source
usage, cloud/shared-data behavior, live action recommendation, or readiness
claims without a later scoped owner-approved workflow.
