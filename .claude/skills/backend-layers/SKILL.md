---
name: backend-layers
description: Layer rules for the parser and backend modules of dew-delivery-quotation-manager.
---

## Layers

Calls go top to bottom only:

1. `backend/.../web`: controllers, `web/dto` records, `ApiExceptionHandler`.
2. `backend/.../service`: use cases, transactions, totals reconciliation.
3. `backend/.../repo` (SQL DAOs) and `backend/.../storage` (files on disk); `parser/` as a library.
4. `backend/.../domain`: records shared by service and repo.

`config` wires beans and settings and holds no logic. `error` holds the exceptions the handler maps.

## What each layer may do

- `parser/`: plain Java and POI only, no Spring, no knowledge of the backend. Its only API is `PackingListParser`, `PackingListResult`, `PackingListItem` and `PackingListParseException`, so the backend never depends on parsing internals.
- `web`: binds and validates the request, calls one service method, returns its result. No SQL, no file access, no business rules, so every rule lives in one testable place.
- `web/dto`: records shaped exactly like CONTRACTS.md §3, with `snake_case` names and static `from` factories that map domain records. The contract is pinned by `ContractShapeTest`.
- `ApiExceptionHandler`: the only place exceptions become HTTP responses, in the `{"error","detail","fields"}` shape.
- `service`: may use domain records, repos, storage, the parser and web DTOs. Opens transactions with its `TransactionTemplate`; deletes files only after the transaction commits, so a rollback never leaves rows pointing at missing files.
- `repo`: SQL through `NamedParameterJdbcTemplate` with named parameters. Returns domain records, never DTOs. Portable SQL only: no `RETURNING`, `ILIKE`, JSON or array types (SPEC §10.4).
- `storage`: reads and writes files under `DEW_STORAGE_DIR` with generated names and `java.nio.file.Path`. Never derives a path from an uploaded filename.
- `domain`: records and enums without Spring or SQL.
- Schema: Flyway migrations in `backend/src/main/resources/db/migration`, never edited after merge because Alice's database already ran them. Use create-copy-drop-rename where SQLite's `ALTER TABLE` falls short.
