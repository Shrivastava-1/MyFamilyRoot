# Bharat Filter Engine

A configurable Ruby filtering engine that supports string, array, boolean, numeric, date-range, presence, and nested association filters, along with field-based search, multi-level associations, and dynamic filter values.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Changelog](#changelog)
  - [0.2.0 — October 24, 2026](#020---2026-10-24)
  - [0.1.2 — August 19, 2026](#012---2026-08-19)
  - [0.1.0 — August 19, 2026](#010---2026-08-19)

## Overview

Bharat Filter Engine provides a configuration-driven approach to filtering and searching Active Record records. It supports multiple filter types, nested associations, numeric ranges, dynamic dropdown values, and flexible search conditions.

## Features

- String, array, and boolean filters
- Integer and float filters
- Numeric range and date-range filters
- Negative filtering with `negate: true`
- Presence and blank-value filtering
- Nested association filters
- Multi-level nested search
- AND / OR search conditions
- Config-driven search using `filter_type: :search`
- Dynamic filter values for dropdowns and autocomplete
- Partial, case-insensitive matching for dynamic filter values
- Legacy filter configuration support
- Input validation and improved error handling

## Changelog

All notable changes to this project are documented in this file.

### [0.2.0] - 2026-10-24

#### Added

- **Negation support with `negate: true`**

  Added the `negate: true` rule option for string, array, boolean, and exact-match integer/float filters. It inverts the filtering condition, using `!=` or `NOT IN` where appropriate.

  Negation cannot be combined with `range_type:`, `daterange`, `search`, or `presence` filters. Invalid combinations raise `InvalidConfigurationError`.

- **Numeric range filtering with `range_type: :between`**

  Added `range_type: :between` for integer and float filters. It supports `{ from:, to: }` and `[from, to]` input formats, similar to date-range filtering.

- **Presence filtering with `filter_type: :presence`**

  Added filtering based on whether a database column contains a value or is blank.

  - Present values match records where the column is `NOT NULL` and not an empty string.
  - Blank values match records based on a falsy parameter value.
  - Supports nested and associated columns.

- **Enhanced dynamic filter values**

  `BharatFilterEngine.filter_values` and `DynamicFilterValues` now accept optional `term:` and `limit:` arguments.

  - `term:` performs partial, case-insensitive matching against distinct values.
  - `limit:` restricts the number of returned values.
  - Useful for typeahead and autocomplete dropdowns.

- **Documented configuration-driven search**

  Added documentation and a dedicated `Engine.apply` spec for `filter_type: :search`, which was previously available only through direct `SearchBuilder` instantiation.

#### Fixed

- **Invalid nested associations now raise errors**

  A nested filter rule with an unresolved `association:` now raises `BharatFilterEngine::InvalidAssociationError` instead of silently returning an unfiltered scope.

  Since association names come from developer configuration rather than request parameters, an unresolved association indicates a configuration error.

- **Invalid search column configurations now raise errors**

  `SearchBuilder` now raises `InvalidConfigurationError` when `allowed_columns:` is non-empty but none of its entries resolve against the model.

  Previously, searches could silently return the entire unfiltered scope.

- **Rule configuration validation**

  `RuleNormalizer` now raises `InvalidConfigurationError` for the following invalid configurations:

  - Unknown `filter_type`.
  - A `type: :nested` rule missing `association:`.
  - A `filter_type: :search` rule with missing or empty `dbcolumns:`.
  - Any other rule missing `dbcolumn:`.
  - Unknown `range_type`.
  - `range_type:` used with a non-numeric filter.
  - `type: :nested` combined with `filter_type: :search`.

  These cases previously resulted in silent no-ops, potentially causing filters to return unfiltered results.

- **SQL `LIKE` wildcard escaping**

  Literal `%` and `_` characters in search terms are now escaped before being used in `LIKE` or `ILIKE` patterns.

  For example, searching for `sale_2026` now treats the underscore as a literal character instead of allowing it to match any single character.

  This applies to simple and field-based searches in `SearchBuilder`, as well as the `term:` option in `DynamicFilterValues`.

- **Cross-database ID text searching**

  Replaced PostgreSQL-specific `column::text` casting with portable `CAST(column AS TEXT)` syntax in `SearchBuilder` ID text searches.

  This fixes SQL syntax errors on SQLite and improves compatibility with other database systems, including MySQL.

- **Invalid numeric input handling**

  Integer and float filters, including all supported range types, no longer silently convert non-numeric input such as `"abc"` to `0`.

  - Invalid numeric values are ignored, and the corresponding filter is skipped.
  - For `range_type: :between`, an invalid bound is discarded while the other valid bound is still applied.

### [0.1.2] - 2026-08-19

#### Fixed

- **Field-based search with invalid AND-group fields**

  Fixed `SearchBuilder#field_based_search` so an `AND` group containing a field not included in `allowed_columns` is treated as unmatchable rather than silently ignored.

  Previously, dropping the invalid condition could cause the search to return every record.

- **Unexpected test fixture creation**

  Changed the top-level `client`, `lead`, and `sale` fixtures in `spec/engine_spec.rb` from `let!` to lazy `let`.

  Previously, eager fixture creation inserted records before every example, including examples that created their own sales records. These extra records interfered with filter results and caused unrelated records to appear in `contain_exactly(...)` assertions.

- **Incorrect test class references**

  Corrected specs that invoked `described_class.apply` or `described_class.new(scope: ..., allowed_columns: ...)` while resolving `described_class` to the wrong class (`Engine` or `RuleNormalizer`).

  The affected examples were moved to their appropriate spec files, including `search_builder_spec.rb`, `dynamic_filter_values_spec.rb`, and `engine_spec.rb`, so they test the intended classes and behavior.

#### Changed

- **Extracted shared filter condition logic**

  Refactored `Engine` by extracting repeated table-qualified `where` branching from array, boolean, string, and exact numeric filters into a shared `apply_condition` helper.

- **Reused `AssociationResolver` instances**

  Updated `DynamicFilterValues` and `SearchBuilder` to reuse a single `AssociationResolver` instance instead of creating a new instance for each call or scope.

### [0.1.0] - 2026-08-19

#### Added

- Initial Bharat Filter Engine implementation.
- String filters.
- Array filters.
- Boolean filters.
- Integer filters.
- Float filters.
- Numeric range filters.
- Date-range filters.
- Nested association filters.
- Multi-level nested search.
- AND / OR search conditions.
- Dynamic filter values.
- Legacy filter configuration support.

---

## Versioning

This project follows a versioned release history. Each release documents its notable additions, changes, and fixes to help developers understand how the engine evolves.
