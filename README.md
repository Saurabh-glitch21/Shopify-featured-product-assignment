# Shopify Developer Assignment

This project is built on Shopify's Dawn theme and customizes the collection page so featured products appear first on the initial load while preserving infinite scroll and normal Shopify behavior for sorting and filtering.

## Approach

### 1. How featured and non-featured products were loaded and separated

Instead of trying to discover featured products only by scanning the entire collection in Liquid, I used a collection metafield called `custom.featured_products` with the type `List of products`.

On the default collection view, the theme reads the first 15 products from that metafield and renders them at the top of the grid. After that, it loops through the normal collection products and fills the remaining visible slots with non-featured products until the first 20 products are shown.

To make sure featured products are not rendered again in the normal part of the first grid, their ids are collected first and checked before any standard collection product is rendered.

### 2. How infinite scroll was implemented

Infinite scroll is handled with JavaScript using an `IntersectionObserver`.

A small sentinel element is placed below the product grid. Once that element enters the viewport, the script requests the next page of the collection using Shopify section rendering. Instead of loading a full page, it requests the collection section HTML for the next page and extracts only the product items from that response.

This keeps the implementation close to how Shopify and Dawn already handle collection updates, which made it easier to preserve compatibility with existing sorting and filtering behavior.

### 3. How duplicate products were prevented

Duplicate prevention is handled on the client side.

When the page first loads, the script creates a `Set` containing the ids of all products already rendered in the grid. Every time the next page is fetched, each returned product is checked before being appended.

A product is skipped if:

- it is already in the rendered product id set
- it belongs to the featured product list and featured mode is active

This avoids duplicates both on the initial transition from the pinned layout and on later pages where Shopify may still return products that were already shown earlier.

### 4. How the solution scales for large collections

The main reason I moved featured products into a collection metafield was scalability.

Liquid pagination is limited and does not make it practical to reliably scan a large collection and reorder products globally by tag. On a larger collection, some featured products may sit outside the first Liquid pagination window, which makes a tag-only solution unreliable.

By explicitly storing featured products in a collection metafield, the theme can always access the intended featured items directly, regardless of how large the collection is. The rest of the collection continues to load through normal paginated requests in batches of 20, which keeps the front-end work lightweight and predictable.

### 5. How filtering and sorting were handled

Sorting and filtering are intentionally left to Shopify's native collection behavior.

The featured pinning logic only runs on the default collection state. If a sort option or filter is active, the custom pinning logic is disabled and the collection is rendered in Shopify's normal order.

This keeps the behavior consistent with what a merchant or customer would expect when using:

- Best Selling
- Price Low to High
- Price High to Low
- collection filters

Infinite scroll still works in those cases, but it simply continues loading products in Shopify's default sorted or filtered order instead of applying any manual pinning.

### 6. Liquid limitations and how they were handled

The main Liquid limitation in this task is that Liquid cannot reliably reorder a large collection by tag across the full collection dataset. It only has access to the current paginated slice of products, so a tag-based "pull all featured to the top" approach becomes unreliable once the collection grows.

To solve that, I did not depend on Liquid to discover featured products across the whole collection. Instead, I used a collection metafield as the source of truth for the featured products. That gave the theme a reliable way to render the correct featured products first, while leaving the rest of the collection loading to Shopify's native pagination.

In other words, Liquid is used for the initial layout, and JavaScript handles the ongoing infinite scroll and duplicate control.

## Files changed

- `sections/main-collection-product-grid.liquid`
- `sections/collection-infinite-scroll.liquid`
- `snippets/collection-product-grid-item.liquid`
- `assets/featured-collection.js`
- `assets/facets.js`
- `templates/collection.json`

## Setup

1. Create a collection metafield:
   - `custom.featured_products`
   - Type: `List of products`

2. Select exactly 15 featured products for the target collection.

3. Set the collection page size to 20 products.

## Testing checklist

- Confirm the first 20 visible products are 15 featured + 5 normal
- Scroll and verify featured products do not repeat
- Confirm no duplicate products appear
- Test sorting options
- Test filtering
- Confirm sorting and filtering disable featured pinning
