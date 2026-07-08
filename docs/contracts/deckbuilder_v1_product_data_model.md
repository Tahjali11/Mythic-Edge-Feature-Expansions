# Deckbuilder V1 Product Data Model Contract

## Module

Competitive deckbuilding workbench V1 product and data model.

Plain English: this contract defines the first deterministic product shape for
a competitive deckbuilding workbench. The workbench helps a player organize
their own win-condition thesis, matchup-by-matchup desired 60-card
configurations, unique-card aggregation, manual trim to a final 75-card deck,
and text export. It does not decide what is correct.

This Codex B pass writes only this contract. It does not implement code, build
UI, create backend services, fetch third-party data, scrape MTGTop8, call
external APIs, build the Standard legality provider, add AI interpretation,
generate sideboard plans, rank cards, recommend cuts, or claim optimality,
metagame truth, strategic correctness, or tournament readiness.

## Source Context

- Repository: `Tahjali11/Mythic-Edge-Feature-Expansions`
- Repository URL:
  `https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions`
- Tracker:
  `https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions/issues/9`
- Issue:
  `https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions/issues/10`
- Target artifact:
  `docs/contracts/deckbuilder_v1_product_data_model.md`
- First supported format: `Standard`
- Architecture goal: format-agnostic after V1 boundaries are stable
- Risk tier: High product-boundary risk, low runtime risk if kept contract-only

Current authorization facts:

```yaml
implementation_authorized: false
ui_implementation_authorized: false
backend_service_authorized: false
third_party_data_fetch_authorized: false
mtgtop8_scraping_authorized: false
external_provider_api_calls_authorized: false
standard_legality_provider_implementation_authorized: false
ai_assistance_authorized: false
sideboard_plan_generation_authorized: false
card_ranking_authorized: false
cut_recommendation_authorized: false
optimality_claimed: false
metagame_truth_claimed: false
tournament_readiness_claimed: false
```

## Source Artifacts Inspected

- Feature Expansions tracker #9
- Feature Expansions issue #10
- `README.md`
- `LICENSE`

The repository does not currently contain repo-local `AGENTS.md`, workflow
docs, source code, contract templates, or existing deckbuilder artifacts. This
contract therefore treats the tracker and issue as the active product
authority.

## Product Thesis

V1 answers one practical question:

> Can a competitive player organize their own matchup-by-matchup deckbuilding
> process into a final 75-card deck without the tool pretending to know the
> correct answer?

V1 is deterministic. Deterministic means the same user-entered project data
and the same legality snapshot produce the same validation, aggregation, trim
state, and export text. The tool may validate structure and legality, but it
must not infer strategy, rank cards, recommend cuts, generate sideboard plans,
or claim that the final deck is optimal.

## V1 Workflow Contract

V1 workflow:

1. User creates a `DeckbuildingProject`.
2. User selects `Standard` as the first supported `FormatSelection`.
3. Tool binds the project to a `LegalitySnapshot` from a later legality
   provider dependency.
4. User manually enters a `WinConditionThesis`.
5. User adds one or more `OpponentDeckSubmission` records by paste or import.
6. User creates one `MatchupMatrixRow` per opponent deck or matchup.
7. User builds one `DesiredSixtyConfiguration` for each row.
8. Tool aggregates all row configurations into a `UniqueCardPool`.
9. Tool displays `CardUsageCount`, `CardConflict`, row coverage, and over-75
   pressure.
10. User records manual `TrimDecision` entries until the project can produce a
    `FinalSeventyFive`.
11. Tool exports the final mainboard plus sideboard through an `ExportProfile`.

V1 must fail closed when required data is missing. Fail closed means the tool
stops or marks the state blocked instead of guessing. Examples include missing
legality data, unresolved card names, invalid quantities, incomplete rows, and
final lists that are not exactly 75 cards in competitive V1 mode.

## Core Entity Relationship

```text
DeckbuildingProject
  -> FormatSelection
  -> LegalitySnapshot
  -> WinConditionThesis
  -> OpponentDeckSubmission[]
  -> MatchupMatrixRow[]
       -> DesiredSixtyConfiguration
  -> UniqueCardPool
       -> CardUsageCount[]
       -> CardConflict[]
  -> TrimDecision[]
  -> FinalSeventyFive
       -> ExportProfile[]
```

The project is the container. User-authored choices are the source of truth for
strategy, matchup rows, desired 60-card configurations, and trim decisions.
The legality provider is the source of truth for format legality and structural
decklist validation once that provider exists.

## Entity Definitions

### DeckbuildingProject

Purpose: top-level container for one deckbuilding effort.

Required fields:

- `project_id`: stable local identifier.
- `project_name`: user-visible name.
- `created_at_utc`: creation timestamp.
- `updated_at_utc`: last changed timestamp.
- `format_selection_id`: reference to one `FormatSelection`.
- `win_condition_thesis_id`: reference to one `WinConditionThesis`.
- `project_status`: one of `draft`, `blocked`, `in_progress`,
  `ready_for_manual_trim`, `final_75_ready`, `export_ready`, `archived`.

Optional fields:

- `project_notes`: user-authored notes.
- `archetype_label`: user-authored descriptive label.
- `engine_label`: user-authored descriptive label.
- `source_context_label`: user-authored event, testing, or metagame context.

Source of truth: user-created project state.

Validation rules:

- exactly one active format selection;
- exactly one active win-condition thesis before matchup rows can be marked
  complete;
- project cannot become `export_ready` until a valid `FinalSeventyFive`
  exists.

Must not claim or infer:

- whether the project is strategically correct;
- whether the named archetype is accurate;
- tournament readiness;
- metagame truth;
- optimality.

### FormatSelection

Purpose: records the project format and its legality dependency.

Required fields:

- `format_selection_id`
- `format_code`: V1 allows only `standard`.
- `format_label`: `Standard`.
- `legality_snapshot_id`
- `format_status`: one of `selected`, `blocked_missing_legality`,
  `blocked_stale_legality`, `blocked_disputed_legality`, `valid_for_v1`.

Optional fields:

- `format_notes`
- `future_format_code`: reserved for later non-Standard formats.

Source of truth: user selects the format; the legality provider validates
whether the selected format can be used.

Validation rules:

- V1 must reject non-Standard formats as `blocked_format_not_supported_v1`
  unless a later child explicitly expands support;
- Standard cannot be marked `valid_for_v1` without a current
  `LegalitySnapshot`;
- synthetic/demo legality fixtures must be clearly marked and cannot be used
  for real competitive export claims.

Must not claim or infer:

- that Standard support implies other format support;
- that a stale legality snapshot is usable for competitive decisions;
- that a synthetic fixture represents real Standard legality.

### LegalitySnapshot

Purpose: immutable metadata for one legality answer set used by the project.

Required fields:

- `legality_snapshot_id`
- `format_code`
- `provider_ref`
- `provider_version`
- `source_label`
- `source_version_or_date`
- `snapshot_created_at_utc`
- `effective_date`
- `freshness_status`: one of `current`, `stale`, `missing`, `disputed`,
  `synthetic_demo_only`.
- `supported_checks`: list of checks the provider can perform.

Optional fields:

- `rotation_context`
- `known_disputes`
- `provider_notes`

Source of truth: later Standard legality provider, not the deckbuilder
contract.

Required provider answers:

- whether a card is legal in Standard;
- whether a card name is unresolved or ambiguous;
- whether a decklist violates card-count constraints;
- whether a decklist violates maindeck-size constraints;
- whether a decklist violates sideboard-size constraints;
- which snapshot/version/source/date produced the answer.

Validation rules:

- if legality data is missing, stale, or disputed, the project must stop at a
  blocked or demo-only state;
- deckbuilder code must not hardcode a static Standard card list;
- deckbuilder code must not fetch or scrape legality data directly unless a
  later legality provider child authorizes it.

Must not claim or infer:

- legality beyond the provider's declared scope;
- correctness of an external provider;
- tournament legality if data is stale, disputed, or synthetic.

### WinConditionThesis

Purpose: user-authored statement of how the deck intends to win.

Required fields:

- `win_condition_thesis_id`
- `project_id`
- `thesis_text`
- `author_source`: V1 value is `user`
- `created_at_utc`
- `updated_at_utc`

Optional fields:

- `supporting_labels`: user-authored labels such as archetype, engine, axis,
  pressure plan, inevitability plan, or combo object.
- `reference_notes`: user-authored notes.

Source of truth: user-authored input only.

Validation rules:

- thesis text must be present before rows can be marked complete;
- empty, generated, or unknown thesis state must block final project review;
- labels are descriptive and must not replace the thesis text.

Must not claim or infer:

- the correct win condition;
- strategy from deck contents;
- archetype truth;
- AI-authored interpretation.

### OpponentDeckSubmission

Purpose: user-submitted opponent decklist or matchup input.

Required fields:

- `opponent_deck_submission_id`
- `project_id`
- `submission_label`
- `submission_source_kind`: one of `paste`, `manual_entry`,
  `file_import_pending`, `external_adapter_deferred`.
- `submitted_at_utc`
- `decklist_parse_status`: one of `not_parsed`, `parsed`, `blocked_invalid`,
  `blocked_ambiguous`, `blocked_parser_deferred`.

Optional fields:

- `source_url`: user-supplied reference only.
- `source_name`: user-supplied label such as event or site name.
- `source_date`
- `raw_input_text`: local project data, not committed contract data.
- `normalized_decklist_ref`: later parser output reference.

Source of truth: user-submitted text or later authorized import adapter.

Validation rules:

- V1 must support user paste/manual entry before any source adapter is
  required;
- unresolved card names must block validation until resolved by a parser or
  user correction;
- source URLs are provenance hints, not provider authority.

Must not claim or infer:

- that the opponent deck represents the metagame;
- that a source URL is accurate;
- MTGTop8 truth or tournament-room frequency;
- matchup weight.

### MatchupMatrixRow

Purpose: one Elephant matrix row for an opponent deck or matchup.

Required fields:

- `matchup_matrix_row_id`
- `project_id`
- `opponent_deck_submission_id`
- `row_label`
- `row_status`: one of `draft`, `blocked_missing_opponent_deck`,
  `blocked_missing_desired_sixty`, `blocked_legality`, `complete`.
- `desired_sixty_configuration_id`

Optional fields:

- `matchup_notes`
- `user_matchup_label`
- `row_order`

Source of truth: user-created matchup row.

Validation rules:

- each complete row must reference exactly one desired 60-card configuration;
- a row may represent a specific decklist or a user-named matchup class;
- duplicate-looking rows may be flagged as `duplicate_review_required` but
  must not be merged automatically.

Must not claim or infer:

- matchup importance;
- matchup win rate;
- sideboard plan correctness;
- room share;
- strategic priority.

### DesiredSixtyConfiguration

Purpose: user-authored desired 60-card configuration for one matchup row.

Required fields:

- `desired_sixty_configuration_id`
- `matchup_matrix_row_id`
- `project_id`
- `card_entries`: list of normalized card references and quantities.
- `configuration_status`: one of `draft`, `blocked_not_60`,
  `blocked_unresolved_card`, `blocked_illegal_card`, `blocked_copy_count`,
  `valid_desired_sixty`.
- `legality_snapshot_id`

Optional fields:

- `configuration_notes`
- `user_role_labels`: user-authored card role labels.

Source of truth: user-authored configuration plus legality provider validation.

Validation rules:

- total quantity must equal exactly 60;
- every card must resolve to a legality provider card identity or a blocked
  unresolved state;
- card counts must pass provider constraints for the selected format;
- a desired 60 is a desired matchup configuration, not an automatically
  generated sideboard plan.

Must not claim or infer:

- that the desired 60 is optimal;
- that the desired 60 can be reached from the final 75 through legal
  sideboarding;
- that card roles are correct;
- that the tool recommends this configuration.

### UniqueCardPool

Purpose: deterministic aggregation of all unique cards used across desired
60-card row configurations.

Required fields:

- `unique_card_pool_id`
- `project_id`
- `source_row_ids`
- `legality_snapshot_id`
- `pool_entries`: one entry per normalized unique card.
- `pool_total_quantity`
- `over_75_pressure`: non-negative integer equal to
  `max(pool_total_quantity - 75, 0)`.
- `pool_status`: one of `blocked_no_complete_rows`, `blocked_unresolved_card`,
  `blocked_legality`, `within_75`, `over_75_manual_trim_required`.

Optional fields:

- `aggregation_notes`
- `last_aggregated_at_utc`

Source of truth: deterministic aggregation from user-authored
`DesiredSixtyConfiguration` rows.

Aggregation rule:

- cards are grouped by normalized card identity;
- for each card, `candidate_quantity` is the maximum quantity requested by any
  one desired 60 row;
- `row_usage_count` is the number of rows where the card appears;
- `row_quantities` records the user-requested quantity per row;
- `pool_total_quantity` is the sum of all `candidate_quantity` values.

The maximum-quantity rule is descriptive. It does not recommend that quantity;
it records the minimum pool quantity needed to satisfy the largest user-entered
row demand for that card.

Must not claim or infer:

- card quality;
- whether a high row count means a card is better;
- whether a low row count means a card should be cut;
- strategic role truth.

### CardUsageCount

Purpose: display how a card appears across matchup rows.

Required fields:

- `card_usage_count_id`
- `unique_card_pool_id`
- `normalized_card_id`
- `display_name`
- `candidate_quantity`
- `row_usage_count`
- `row_quantities`
- `rows_using_card`

Optional fields:

- `user_role_labels_by_row`
- `notes`

Source of truth: `UniqueCardPool` aggregation.

Validation rules:

- `candidate_quantity` must equal the maximum row quantity for the card;
- `row_usage_count` must equal the number of rows where quantity is greater
  than zero;
- display ordering must be neutral unless a later child explicitly defines a
  sort option.

Must not claim or infer:

- ranking;
- cut priority;
- matchup importance;
- sideboard importance.

### CardConflict

Purpose: public-safe, deterministic conflict or warning classification.

Required fields:

- `card_conflict_id`
- `project_id`
- `normalized_card_id`
- `conflict_type`
- `affected_row_ids`
- `conflict_status`: one of `active`, `resolved`, `dismissed_by_user`.

Allowed V1 conflict types:

- `unresolved_card_name`
- `ambiguous_card_name`
- `illegal_in_snapshot`
- `copy_count_violation`
- `quantity_varies_by_row`
- `role_label_varies_by_row`
- `pool_exceeds_75`
- `final_75_missing_assignment`
- `export_blocked_invalid_final_75`

Optional fields:

- `public_safe_message`
- `user_resolution_note`

Source of truth: deterministic validation and aggregation.

Validation rules:

- conflict messages must describe observable data shape only;
- conflicts may block export when they affect legality or final 75 validity;
- conflicts must not include hidden scoring or advice.

Must not claim or infer:

- which conflict matters most strategically;
- which card should be cut;
- whether a conflict proves a deck is wrong.

### TrimDecision

Purpose: user-authored decision that turns the unique-card pool into a final
75-card deck.

Required fields:

- `trim_decision_id`
- `project_id`
- `normalized_card_id`
- `candidate_quantity_before`
- `final_quantity`
- `location_assignment`: one of `mainboard`, `sideboard`, `excluded`,
  `split_main_side`, `unassigned`.
- `decision_source`: V1 value is `user`.
- `decision_status`: one of `draft`, `active`, `superseded`.

Optional fields:

- `mainboard_quantity`
- `sideboard_quantity`
- `user_reason_note`
- `created_at_utc`
- `updated_at_utc`

Source of truth: user manual decision.

Validation rules:

- `final_quantity` cannot exceed `candidate_quantity_before` unless a later
  explicit manual-add workflow exists;
- `mainboard_quantity + sideboard_quantity` must equal `final_quantity` when
  location is split;
- decisions must not be generated by the tool in V1;
- unresolved cards or legality-blocked cards cannot be exported.

Must not claim or infer:

- that excluded cards were bad;
- that included cards were correct;
- cut recommendations;
- sideboard plan recommendations.

### FinalSeventyFive

Purpose: final competitive V1 decklist state produced from manual trim
decisions.

Required fields:

- `final_seventy_five_id`
- `project_id`
- `mainboard_entries`
- `sideboard_entries`
- `final_total_quantity`
- `mainboard_total_quantity`
- `sideboard_total_quantity`
- `legality_snapshot_id`
- `final_status`: one of `blocked_missing_trim`, `blocked_not_75`,
  `blocked_not_60_mainboard`, `blocked_not_15_sideboard`,
  `blocked_legality`, `valid_final_75`.

Optional fields:

- `final_notes`
- `created_at_utc`
- `updated_at_utc`

Source of truth: user trim decisions plus legality provider validation.

Validation rules:

- competitive V1 requires exactly 75 cards total;
- competitive V1 requires exactly 60 mainboard cards and exactly 15 sideboard
  cards;
- every card must remain legal in the bound legality snapshot;
- export must be blocked unless `final_status` is `valid_final_75`.

Must not claim or infer:

- tournament readiness;
- optimal deck construction;
- metagame correctness;
- legal registration in any specific tournament beyond provider-backed format
  checks.

### ExportProfile

Purpose: deterministic text export format for a valid final deck.

Required fields:

- `export_profile_id`
- `project_id`
- `final_seventy_five_id`
- `export_format`: one of `mtga_text`, `mtgo_text`.
- `export_status`: one of `blocked_invalid_final_75`, `ready`,
  `exported_text_generated`.

Optional fields:

- `export_text`
- `generated_at_utc`
- `line_order_policy`

Source of truth: `FinalSeventyFive`.

Validation rules:

- exports are text-only;
- exports must not upload, submit, register, or sync to external services;
- exports must preserve card names and quantities from the final 75;
- exports must include mainboard and sideboard grouping where the target
  profile requires it.

Must not claim or infer:

- that MTGA or MTGO accepted the list;
- tournament registration readiness;
- card availability in a user account;
- strategic correctness.

## Standard Legality Dependency Boundary

The deckbuilder depends on a future Standard legality provider. V1 must not
hardcode Standard legality into deckbuilder entities.

Required provider boundary:

```text
resolve_card_name(input_name, legality_snapshot_id)
is_card_legal(normalized_card_id, format_code, legality_snapshot_id)
validate_card_count(normalized_card_id, quantity, format_code, legality_snapshot_id)
validate_deck_shape(card_entries, format_code, legality_snapshot_id)
describe_snapshot(legality_snapshot_id)
```

Required provider outputs:

- normalized card identity;
- display name;
- legality status;
- unresolved or ambiguous status;
- card-count validation status;
- maindeck and sideboard shape validation;
- snapshot source/version/date/freshness metadata;
- public-safe reason codes for missing, stale, disputed, or synthetic data.

Deckbuilder behavior when provider is unavailable:

- project creation may remain possible;
- Standard project validation must be `blocked_missing_legality`;
- desired 60 validation must be blocked or demo-only;
- final 75 export must be blocked unless an explicitly labeled synthetic demo
  fixture mode is active;
- synthetic demo mode must not claim real Standard legality.

## Elephant Matrix Model

The Elephant matrix is user-driven.

Matrix rules:

- one row represents one opponent deck or matchup label;
- each complete row has exactly one desired 60-card configuration;
- desired 60s are user-authored;
- rows may be added, reordered, renamed, archived, or marked incomplete;
- row coverage is descriptive and counts which rows use each card;
- row conflicts are validation or difference signals, not advice.

The matrix must not:

- assign matchup weights;
- estimate expected value;
- infer the metagame;
- rank opponent decks;
- generate a sideboard plan;
- claim that a desired 60 is reachable from the final 75 through legal
  sideboarding.

## Unique-Card Aggregation Model

Aggregation is deterministic and local to the project.

For each complete desired 60 row:

1. resolve each card through the legality/card identity boundary;
2. group by normalized card identity;
3. record row quantity;
4. compute per-card `candidate_quantity` as the maximum row quantity;
5. compute `row_usage_count`;
6. sum candidate quantities to produce `pool_total_quantity`;
7. compute `over_75_pressure`.

Example:

```text
Row A requests 4 copies of Card X.
Row B requests 2 copies of Card X.
The unique pool candidate quantity for Card X is 4, not 6.
```

This means the pool contains enough copies to satisfy the largest single row
request. It does not mean the tool recommends 4 copies.

## Manual Trim Model

Manual trim is user-owned.

Allowed tool behavior:

- show pool total quantity;
- show over-75 pressure;
- show row usage and row quantities;
- show conflicts;
- validate whether the current manual trim produces 60 mainboard cards and 15
  sideboard cards;
- block export when final 75 is invalid.

Forbidden V1 behavior:

- recommend which cards to remove;
- rank trim candidates;
- auto-trim;
- infer strategic importance;
- generate sideboard plans;
- claim the user's final 75 is optimal.

## Export Boundary

V1 export is deterministic text generation only.

Allowed export profiles:

- `mtga_text`
- `mtgo_text`

Export may:

- format quantities and card names;
- separate mainboard and sideboard where required;
- include public-safe warnings when the final 75 is invalid;
- refuse export when validation fails.

Export must not:

- call MTGA, MTGO, Companion, tournament, store, or external APIs;
- submit decklists;
- sync user accounts;
- fetch card data;
- correct card choices;
- add missing cards;
- mutate project data silently;
- claim that an external client accepted the deck.

## Deferred Child Hooks

This contract leaves hooks for these later children:

- Standard legality provider and rolling card pool.
- Decklist parser and normalization boundary.
- Elephant matrix MVP implementation.
- Manual trim workflow implementation.
- MTGA and MTGO export profiles.
- MTGTop8 source adapter and filters.
- Matchup weighting and room expected-value model.
- AI-assisted interpretation boundary.

Deferred children must not treat this contract as permission to implement
external fetch, scraping, AI advice, ranking, cut recommendations, sideboard
plans, matchup weighting, backend services, or UI work without a separate
issue and contract.

## Public-Safe Reason Vocabulary

Allowed V1 reason categories:

- `blocked_missing_legality_provider`
- `blocked_stale_legality_snapshot`
- `blocked_disputed_legality_snapshot`
- `blocked_format_not_supported_v1`
- `blocked_unresolved_card`
- `blocked_ambiguous_card`
- `blocked_illegal_card`
- `blocked_copy_count_violation`
- `blocked_desired_sixty_not_60`
- `blocked_no_complete_rows`
- `blocked_pool_exceeds_75`
- `blocked_missing_trim_decision`
- `blocked_final_total_not_75`
- `blocked_mainboard_not_60`
- `blocked_sideboard_not_15`
- `blocked_export_invalid_final_75`
- `demo_only_synthetic_legality`

Reason categories are validation status only. They are not advice.

## Non-Claims

V1 does not claim:

- optimal deck construction;
- metagame correctness;
- tournament readiness;
- matchup win probability;
- expected value;
- card ranking;
- cut recommendations;
- sideboard plan correctness;
- AI strategy interpretation;
- legality provider correctness beyond reported snapshot metadata;
- MTGTop8 truth;
- external source truth;
- account ownership or card availability;
- final deck registration readiness.

V1 claims only:

- the tool organized user-supplied deckbuilding decisions;
- deterministic aggregation followed the contract;
- validation statuses were produced from user input and the bound legality
  snapshot;
- export text was generated from the validated final 75 when export is allowed.

## Validation Expectations For Later Work

Later implementation should prove:

- entity schema examples validate with required and optional fields;
- Standard-only V1 rejects non-Standard formats;
- missing legality provider blocks real validation and export;
- synthetic/demo legality fixtures are labeled demo-only;
- desired 60 rows must total exactly 60 cards;
- unique-card aggregation uses maximum requested quantity per card across rows;
- row usage counts and row quantities are deterministic;
- over-75 pressure is computed from candidate quantities;
- manual trim is required and user-authored;
- final 75 requires exactly 60 mainboard and 15 sideboard cards;
- MTGA and MTGO exports are text-only and deterministic;
- no code path fetches third-party data, scrapes MTGTop8, calls external APIs,
  invokes AI, ranks cards, recommends cuts, or generates sideboard plans.

Suggested future validation surfaces:

- schema fixture tests for every entity;
- deterministic aggregation fixture tests;
- blocked-legality fixture tests;
- final 75 validation tests;
- export text golden tests;
- no-network/no-provider-call tests;
- no-AI/no-advice vocabulary scan.

## Stop Conditions

Stop and return to Codex B or the owner if:

- a V1 implementation wants to hardcode Standard legality;
- the legality provider is missing but real export is requested;
- the decklist parser boundary is treated as already implemented;
- the product starts ranking cards or recommending cuts;
- matchup weighting or room expected-value modeling is pulled into V1;
- MTGTop8 fetching or scraping is requested without a separate adapter issue;
- AI interpretation is requested before an AI boundary contract exists;
- any output claims optimality, metagame truth, tournament readiness, or
  strategic correctness.

## Recommended Next Role

Codex E: Product Contract Reviewer.

Codex E should review the entity definitions, Standard-first legality
dependency boundary, Elephant matrix aggregation rule, manual trim boundary,
export boundary, deferred child hooks, validation expectations, and non-claims.

## Pasteable Codex E Prompt

```text
Use the Mythic Edge workflow rules.

Act as Codex E: Product Contract Reviewer for Feature Expansions issue #10.

Repository:
Tahjali11/Mythic-Edge-Feature-Expansions

Tracker:
https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions/issues/9

Issue:
https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions/issues/10

Contract:
docs/contracts/deckbuilder_v1_product_data_model.md

Review goal:
Review the V1 deterministic competitive deckbuilding workbench product/data
model contract. Lead with findings. Verify that the contract defines the core
entities, Standard-first legality dependency boundary, Elephant matrix model,
unique-card aggregation model, manual trim model, export boundary, deferred
child hooks, validation expectations, and non-claims without authorizing
implementation, UI, backend services, third-party fetch, MTGTop8 scraping,
external APIs, Standard legality provider implementation, AI interpretation,
sideboard plans, card ranking, cut recommendations, optimality claims,
metagame truth, or tournament readiness.

Route to Codex B for contract ambiguity, Codex C only if a later issue
explicitly authorizes implementation, or stop if V1 scope should be changed.
```

## workflow_handoff

```yaml
workflow_handoff:
  repository: "Tahjali11/Mythic-Edge-Feature-Expansions"
  repository_url: "https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions"
  tracker: "https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions/issues/9"
  issue: "https://github.com/Tahjali11/Mythic-Edge-Feature-Expansions/issues/10"
  completed_thread: "B"
  next_thread: "E"
  verdict: "deckbuilder_v1_product_data_model_contract_ready_for_review"
  target_artifact: "docs/contracts/deckbuilder_v1_product_data_model.md"
  first_supported_format: "Standard"
  architecture_goal: "format_agnostic"
  v1_interpretation_mode: "deterministic_user_authored_win_condition"
  standard_legality_dependency_required: true
  standard_legality_provider_implementation_authorized: false
  implementation_authorized: false
  ui_implementation_authorized: false
  backend_service_authorized: false
  third_party_data_fetch_authorized: false
  mtgtop8_scraping_authorized: false
  external_provider_api_calls_authorized: false
  ai_assistance_authorized: false
  sideboard_plan_generation_authorized: false
  card_ranking_authorized: false
  cut_recommendation_authorized: false
  mtgtop8_adapter_deferred: true
  matchup_weighting_deferred: true
  ai_interpretation_deferred: true
  optimality_claimed: false
  metagame_truth_claimed: false
  tournament_readiness_claimed: false
```
