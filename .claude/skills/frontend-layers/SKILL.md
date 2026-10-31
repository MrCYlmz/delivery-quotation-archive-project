---
name: frontend-layers
description: Layer rules for the frontend module of dew-delivery-quotation-manager.
---

## Layers

Imports go top to bottom only:

1. `src/router.ts`: one lazy route per page.
2. `src/pages`: one per route, with its view model `use<Page>()` in `src/composables`.
3. `src/components/templates`, then `organisms`, `molecules`, `atoms`.
4. `src/api` (`client.ts`, `types.ts`) and the shared modules `format.ts`, `countries.ts`, `filters.ts`.

Until the atomic design story (story 2) lands, pages live in `src/views` and components sit flat in `src/components`. New components still go into the level folders.

## What each layer may do

- Pages: fill a template with state from their view model. No `fetch`, no direct API calls in the template code.
- View models (`use<Page>()`): hold page state and are the only code that calls `api/client.ts`, so loading, errors and stale responses are handled in one place per page. Shared lookups such as `useSuggestions` are called from a view model.
- Components: props in, events out. Organisms may hold local UI state, never server data. Building a URL with `pdfContentUrl` or `excelDownloadUrl` is allowed, because it fetches nothing.
- `api/client.ts`: the only code that calls `fetch`. Every function matches an endpoint in CONTRACTS.md §3, and every non-2xx response throws `ApiError`.
- `api/types.ts`: wire types in `snake_case`, exactly as in CONTRACTS.md §3. The frontend never recomputes what the backend returns, such as `item_sums` or `totals_edited`.
- `format.ts`: all display formatting of numbers, money, dates and bytes. `countries.ts`: the bundled ISO 3166-1 list and code-to-name lookup; codes on the wire, names on screen.
- Styles: tokens from `styles/base.css`, with light and dark both defined there.
