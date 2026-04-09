# Shopify Developer Assignment

This project is based on the Dawn theme and customizes the collection page to pin featured products at the top on the initial load while keeping infinite scroll, sorting, and filtering working as expected.

## Overview

On the default collection view:

- 15 featured products are shown first
- the first visible batch contains 20 products total
- the remaining 5 slots are filled with normal products
- later scroll loads only non-featured products
- duplicate products are prevented

When sorting or filtering is applied, the featured pinning logic is disabled and Shopify's normal collection behavior is used.

## Featured product setup

Featured products are managed through a collection metafield instead of relying only on tags across the entire collection.

Metafield used:

- `custom.featured_products`
- Type: `List of products`

This makes the behavior more predictable, especially for larger collections.

## How it works

### Initial load

On the default collection page, the theme:

1. reads the featured products from the collection metafield
2. renders those products first
3. fills the rest of the first 20 visible slots with non-featured products

### Infinite scroll

Infinite scroll is handled with JavaScript and Shopify section rendering.

When the user reaches the bottom of the grid, the next collection page is requested and checked before products are appended. Featured products are skipped after the first load, and already-rendered product ids are ignored to prevent duplicates.

### Sorting and filtering

When sorting or filtering is active:

- featured pinning is turned off
- Shopify's normal ordering is preserved
- infinite scroll continues using the default collection output

## Duplicate prevention

A JavaScript `Set` is used to track product ids already shown on the page.

Before appending a product from a later request, the script checks whether:

- it has already been rendered
- it is one of the featured products already pinned at the top

If either condition is true, that product is skipped.

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

3. Make sure the collection page is set to 20 products per page.

## Expected behavior

### Default collection view

- first 20 products = 15 featured + 5 normal
- later loads contain only non-featured products
- no duplicate products appear

### Sorted or filtered views

- featured pinning is disabled
- Shopify default behavior is preserved

## Testing checklist

- Verify the correct collection is being used
- Verify 15 featured products are selected
- Confirm the first 20 visible products are 15 featured + 5 normal
- Scroll and confirm featured products do not repeat
- Confirm no duplicate products appear
- Test Best Selling
- Test Price Low to High
- Test Price High to Low
- Test filtering
- Confirm sort/filter disables featured pinning
