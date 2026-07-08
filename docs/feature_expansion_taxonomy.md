# Feature Expansion Taxonomy

## Purpose

This document defines how `Mythic-Edge-Feature-Expansions` organizes future
feature ideas. The repo is a planning and promotion repo, not the default place
where features are implemented.

## Issue Classes

### Raw Idea

A raw idea is a note, comment, or intake item that has not yet earned a durable
tracker. It should be parked until it has enough shape to classify.

### Feature Family Tracker

A feature family tracker is a durable product capability area likely to need
multiple child issues.

Current feature-family trackers:

- #2: match overlay and live match telemetry;
- #3: deck tracker, draw odds, sideboard state, and cards-bottomed views;
- #4: opponent scouting and observed-card context;
- #5: analytics drilldowns, match history, and heatmaps;
- #6: economy, inventory, and progression views;
- #7: collection browser and deck exploration views;
- #8: MTGO source-parser support;
- #9: competitive deckbuilding workbench.

### Source / Inspiration Tracker

A source or inspiration tracker captures lessons from another public tool,
project, workflow, or design reference. It should route candidate ideas into
feature-family or shared-dependency trackers before implementation.

Current source/inspiration trackers:

- #1: MTG Arena Tool feature inspiration and parity ideas;
- #12: Manasight inspiration and parser/corpus automation lessons;
- #13: 17Lands limited analytics inspiration and parity ideas;
- #14: Untapped-style deck, matchup, and ranked-improvement inspiration;
- #15: Hollowmark action and draft-surface inspiration;
- #16: public MTGA companion benchmark watch.

### Shared Dependency Tracker

A shared dependency tracker owns reusable prerequisites that multiple feature
families may need.

Current shared dependency trackers:

- #17: feature expansion taxonomy, issue templates, and promotion workflow;
- #18: rolling format legality provider and deck rules validation;
- #19: decklist import, normalization, and export adapters;
- #20: card identity, catalog, and metadata boundaries;
- #21: feature privacy, fair-play, and live-play policy review;
- #22: external data/source adapter policy and public-data boundaries;
- #23: feature promotion packets and implementation-repo routing.

### Child Issue

A child issue is one scoped planning, contract, prototype, review, or routing
step under a tracker. Child issues should be small enough to finish through the
Mythic Edge workflow without changing the parent tracker scope.

### Promotion Packet

A promotion packet is the final planning artifact before creating implementation
work in another repo. It should name the destination repo, remaining blockers,
validation requirements, and non-claims.

## Required Child Issue Floor

Before a feature-family tracker can be promoted to implementation, it must have
at least these child issue lanes:

1. Product thesis and V1 scope.
2. Required parser facts / data-source matrix.
3. Privacy and fair-play risk review.
4. Data model / contract.
5. UI or workflow sketch.
6. Implementation destination decision.
7. MVP implementation issue in the owning repo.
8. Validation and non-claim checklist.

If a lane is not applicable, the owner must explicitly record why and identify
the replacement evidence required before promotion.

## Routing Rules

- Parser facts or source ingestion route to the parser/source owning repo.
- Local analytics/statistics route to `Mythic-Edge-Analytics` unless private
  scoring internals make `Mythic-Edge-Fable-Engine` the owner.
- Security, privacy, and policy remediation route to `Mythic-Edge-Security`.
- Automation prompts and reusable automation artifacts route to
  `Mythic-Edge-Automation-Artifacts`.
- Product/UI implementation routes to the future UI/local-app repo if created,
  or to the owner-selected implementation repo.
- This repo keeps the idea lineage and promotion packet after implementation
  work moves elsewhere.

## Non-Claims

An open tracker in this repo does not authorize:

- implementation;
- parser behavior changes;
- production UI behavior;
- provider calls;
- cloud/shared-data/commercial behavior;
- live action recommendation;
- hidden-information inference;
- copying external code or fixtures;
- merge, deploy, release, security, privacy, or policy readiness claims.

