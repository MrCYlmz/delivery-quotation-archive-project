# Playbook: delivery-quotation-archive

## Idea

A private archive of past deliveries, so a new delivery quote can be based on comparable past ones instead of a search through old email. Running deliveries are tracked there too, by status on a board, from the first request to a week after delivery. Each record holds route, transport method, dates, cost, cargo totals parsed from the factory packing list, attached PDFs and free-text notes, and the list can be filtered down to comparables in seconds. Requirements and domain: `SPEC.md` in the repo.

## Users and usage

One user, Alice, who quotes delivery costs for European companies shipping from Asian factories. No login, no roles, no sharing. Runs locally on her Windows laptop; development happens on Linux. Alice uses v1 with real data, so every change must keep her existing records intact.

## Architecture

One repo, three modules:

- `parser/`: plain Java library, no Spring. Reads the packing list sheet of a factory Excel with Apache POI. Exposes `PackingListParser`.
- `backend/`: Spring Boot app. Layers `web` (controllers, DTOs, exception handler) → `service` → `repo` (hand-written `NamedParameterJdbcTemplate` DAOs) and `storage` (files on disk under generated names). Uses the parser as a Maven dependency. SQLite with Flyway migrations.
- `frontend/`: Vue 3 SPA with five screens: board (start page `/`), list (`/deliveries`), form, detail, estimate. Calls the backend over `/api` through `api/client.ts`. Built into the backend's static resources, so one jar serves everything same-origin. Atomic design: `src/pages` (one per route) fill a `components/templates` template from a `use<Page>()` view model in `src/composables`; components in `atoms|molecules|organisms|templates`.

## Technologies

- Java 21, Maven (no wrapper), Spring Boot 3.4.1, spring-jdbc, Flyway 10.20.1, SQLite (sqlite-jdbc 3.47.1.0), Apache POI 5.4.1, JUnit 5.
- Vue 3.5, TypeScript 5.7 (strict), Vue Router 4, Vite 6, vue-tsc. Vitest 3 (jsdom) and Vue Test Utils for tests in `frontend/tests`; ESLint flat config with eslint-plugin-vue `flat/recommended` and typescript-eslint `recommended`, 0 warnings, no Prettier. No Pinia.
- Hand-written CSS with design tokens in `frontend/src/styles/base.css`, light and dark. No UI framework.
- UI text in English, written directly in templates; no translation layer. Numbers, money and dates go through `format.ts`. Dates are calendar dates, formatted in UTC so they show the same day everywhere.
- Node 22.22.2 downloaded by `frontend-maven-plugin` during `mvn package` (jsdom 30 needs ≥ 22.22.2). After a Node or npm pin change, run `mvn clean` once: old npm in `backend/target/node` breaks `npm ci`.

## Repos and git host

- `/home/learn/Desktop/Repos/dew-delivery-quotation-manager`, GitHub `MrCYlmz/dew-delivery-quotation-manager`, default branch `main`.
- One branch per story, `story/<id>-<slug>`, and a GitHub pull request into `main`. The reviewer approves; the user merges.
- CI (`.github/workflows/ci.yml`) runs `mvn -B clean verify` on ubuntu for every PR into `main` and every push to `main`; a PR's first run misses the caches until `main` has saved them. `mvn verify` runs typecheck, lint and the frontend tests before the frontend build and fails on them; `-Dskip.frontend=true` skips all npm steps.

## Deployment

- One runnable jar from `mvn clean package`. All config through `DEW_*` environment variables with defaults.
- The app's own settings (`dew.*`) bind to the validated record `config/DewProperties`; Spring's own settings (port, datasource, multipart) stay in YAML. A blank or missing `dew.storage-dir` stops startup with a message naming `DEW_STORAGE_DIR`.
- Windows installer (per-user, no admin) and portable zip built by jpackage in `.github/workflows/windows-exe.yml` on a tag `vX.Y.Z` (major ≥ 1) or manual run.
- Both the installer and the zip keep being produced; Alice may use either.
- Packaged app stores data in `%LOCALAPPDATA%\DewDeliveryArchive` via the `windows` Spring profile and opens the browser on startup.

## API conventions

- The contract is `CONTRACTS.md` §3 in the repo, not an OpenAPI document. It changes only through an approved story.
- Base path `/api`, `snake_case` JSON matching SPEC.md §7, ISO dates `yyyy-MM-dd`, numbers as JSON numbers, countries as ISO 3166-1 alpha-2 codes.
- Errors are `{"error", "detail", "fields"}`, not problem JSON: 400 validation, 404 unknown id, 422 parse failure, 500 otherwise.
- No pagination on the list endpoint: one user, a few hundred records.

## Decisions

- SQLite, not Postgres: keeps desktop, container and hosted packaging open; revisit on concurrent writers (SPEC SD-3).
- Hand-written `NamedParameterJdbcTemplate`, not Spring Data JDBC repositories: no SQLite dialect, and aggregate saves would re-id PDFs (SPEC SD-5).
- `foreign_keys=on` on the JDBC url: SQLite enforces foreign keys per connection (SPEC SD-6).
- Frontend built into the jar by `frontend-maven-plugin`: one build on Linux and on the Windows runner (SPEC SD-7).
- The parser's acceptance test (41 items, 12 pallets, 13.07 m³, 3122 kg on both fixtures) must stay green; a change that breaks it is wrong (SPEC §12).
- Explicit `TransactionTemplate` in services, not `@Transactional`: file deletions must run outside the transaction.
- The service layer returns web DTOs and accepts `MultipartFile`; repos and domain never see DTOs. One service, so a separate mapping layer would add nothing.
- English-only UI without a translation layer: one user.
- Page state lives in a `use<Page>()` composable; no Pinia until state is shared across pages.
- Frontend tests go through the screens by URL with `api/client.ts` mocked and assert what the user sees, so they survive restructuring.
- `vue/singleline-html-element-content-newline` and `vue/multiline-html-element-content-newline` stay off: their fix adds visible whitespace.
- View models are the only API callers and drop any response, success or error, to a request a newer one has superseded (a request counter per kind of request).
- Fields point to their error or hint with `aria-describedby`; titled sections are regions labelled by their heading. Tests query through these hooks, never through markup structure.
- After a 422 Excel parse failure the file is dropped; save uploads no Excel.
- Frontend order: S-1 (tests and lint) and S-2 (atomic design) done.
- Estimate: the app picks comparables and drafts a prompt Alice pastes into Claude herself; the app calculates no price and calls no external service (SPEC F-19 to F-23, §3).
- `cost_eur` is Alice's buy cost, not her sell price (SPEC OQ-11).
- "Today" comes from the injected `Clock` bean (`config/ClockConfig`); backend tests replace it with a fixed date in `BackendTest`.
- A lookup triggered by typing in a text or number field waits for a 300 ms pause; choices from a list look up at once.
- UI drafts are made by frontend-developer as a static HTML file in the task folder until the team has a UX designer.
- One offer price per delivery, the accepted one, next to the real price (`cost_eur`), which the UI labels "Real price". Every offer received and the carrier are later stories (SPEC OQ-12).
- A Flyway migration that rebuilds a table (SQLite cannot drop NOT NULL) runs outside the transaction (`.sql.conf` with `executeInTransaction=false`), switches foreign keys off so the drop does not cascade, and carries over `sqlite_sequence` and the indexes. A test seeds rows under the previous schema and proves they survive.
- A native `<dialog>` can be closed by the browser regardless of `cancel` handling; the app treats any close it did not start as cancel, and a request still running reports its outcome on the page.
