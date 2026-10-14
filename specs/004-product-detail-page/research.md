# Research: Product Detail Page

## Decision: Resolve product route identifiers through the Catalog service and distinguish not-found from service failure.

**Rationale**: Catalog owns product identity and availability. A missing product must not be misrepresented as a transient load failure, or vice versa.

**Alternatives considered**: Embedding full product records in the route or sourcing from a client cache alone was rejected because the cache can be stale and is not authoritative.

## Decision: Compose a versioned Catalog detail/recommendations API and Cart mutation API.

**Rationale**: Catalog owns product descriptions, prices, images, availability, and related-item selection; Cart owns quantities and session state. This keeps service responsibilities cohesive.

**Alternatives considered**: A single UI page writing directly to both service databases was rejected because it bypasses independent service contracts.

## Decision: Have Cart service revalidate current availability on every add and derive price from Catalog-owned data.

**Rationale**: A product can become unavailable between page load and click, and browser values cannot be trusted.

**Alternatives considered**: Relying on the availability state captured at page render was rejected because it becomes stale.

## Decision: Derive related products from active products in the current product's category, excluding the current product.

**Rationale**: This is the selected default relevance rule and yields predictable recommendations without an extra ranking dependency.

**Alternatives considered**: A new recommendation service was rejected as unnecessary for the specified MVP.

## Decision: Use an accessible image fallback and omit the related-products section if no eligible product is returned.

**Rationale**: Keeps purchase information available when assets or recommendations are missing, and follows the required empty behavior.

**Alternatives considered**: Empty cards or treating missing images as page failures were rejected because they reduce usability without affecting the product record.
