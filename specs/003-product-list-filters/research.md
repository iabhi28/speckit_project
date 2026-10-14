# Research: Product List Filters

## Decision: Keep filter and sort selection in URL query parameters and treat parsed URL state as the canonical browser selection.

**Rationale**: Directly satisfies share, refresh, and browser history behavior and avoids a second client-only state source.

**Alternatives considered**: Component-only state was rejected because it is lost on reload and cannot be shared or restored through history.

## Decision: Use a single typed parser/serializer for category, subcategory, minimum/maximum price, availability, and sort values.

**Rationale**: A centralized parser can reject or drop malformed/stale values, ensure dependent subcategory rules, and produce deterministic URLs.

**Alternatives considered**: Ad hoc query-string reads in each component were rejected due to inconsistent validation and history handling.

## Decision: Perform filtering, counting, and stable sorting through the Catalog service contract.

**Rationale**: Catalog owns authoritative product availability, category membership, and price; server-side query semantics keep counts and result pages consistent.

**Alternatives considered**: Filtering only the current client page was rejected because result count and ordering would be incorrect for paged result sets.

## Decision: Combine distinct filter dimensions with AND semantics, permit one category/subcategory selection, and clear subcategory when category changes or is removed.

**Rationale**: These are the confirmed feature rules and prevent incompatible category state.

**Alternatives considered**: Multi-select categories and OR semantics were rejected because they contradict the feature assumptions.

## Decision: Use a responsive filter drawer on narrow screens and a persistent filter region on larger screens.

**Rationale**: Preserves product results as primary mobile content while keeping all controls keyboard reachable.

**Alternatives considered**: Permanently visible narrow-screen sidebar was rejected because it crowds results and can create horizontal overflow.
