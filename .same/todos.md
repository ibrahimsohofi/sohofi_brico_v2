# Task: Search product by ID in sales_manager

## Goal
Allow searching products by numeric ID in the sales_manager search fields, in addition to name/SKU/barcode. Support the existing `@ID` convention.

## Plan
- [x] Backend `/api/integration/search` (products_manager/backend/server.js ~L976, active route): `@` strip, exact + partial ID match, exact-ID priority, relaxed min-length for numeric
- [x] Service `inventoryIntegration.js` searchProducts: strip `@`, allow numeric single-char queries
- [x] Backend duplicate search route (~L1186): same ID logic applied for consistency
- [x] `SalesForm.jsx`: `isSearchableQuery` guard, i18n placeholder, `#{id}` badge in suggestions
- [x] `CustomerWishlist.jsx`: `isSearchableQuery` guard, i18n placeholder, `#{id}` badge in suggestions
- [x] i18n placeholder key (`inventory.searchProductPlaceholder` in fr + ar, mentions ID)
- [x] Verify: `node --check` on both server.js files -> OK
- [x] Verify: biome lint on SalesForm/CustomerWishlist/Inventory/ProductForm -> 0 errors
- [x] Fix `isNaN` -> `Number.isNaN` + explicit radix in `Inventory.jsx` ID-search path (only lint error in the ID code)

## Verification results
- `node --check products_manager/backend/server.js` -> OK
- `node --check sales_manager/server/server.js` -> OK
- SQL placeholder/param counts audited: route ~L976 = 7/7, route ~L1186 = 12/12 -> balanced
- `/api/products/:id` returns `{success, product}`; `Inventory.jsx` reads `data.product` -> shape matches
- biome lint on the 4 task-relevant components -> clean (repo baseline still has 14 unrelated pre-existing errors elsewhere in `src`)

## Notes
- MySQL not installed here; can't run full stack. Verified via node --check + lint + code audit.
- Express uses first matching route -> `/api/integration/search` at ~L976 is active (L1186 duplicate is dead code, updated for consistency).
- `@ID` convention: `Inventory.jsx` uses `parseSearchInput` (server-side single-product fetch); `SalesForm`/`CustomerWishlist` use `isSearchableQuery` + `inventoryService.searchProducts`.
- `sales_manager/src/components/ProductForm.jsx` (named in the original task title) is a create/edit form with no search field -> nothing to change there.
- `normalizeProduct` spreads `...product`, so `id` survives normalization and is available for the `#{id}` badge.
- products table: id (INT PK), is_active, name, sku, barcode, selling_price, remaining_stock.
- Minor pre-existing oddity (left alone): `Inventory.jsx` `API_BASE_URL = " http://localhost:5000"` has a leading space; the URL parser strips it so it still works.
