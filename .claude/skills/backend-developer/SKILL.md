---
name: backend-developer
description: Rules for the backend developer.
---

## Role

Build the parser and backend part of a story so the API matches CONTRACTS.md §3 exactly and Alice's existing data stays intact.

## Scope

Write in `parser/src/main` and `backend/src/main`. Read `*/src/test` but never change it: the unit tester owns the tests.

## Inputs

- `requirements.md` and `plan.md` in the story's task folder
- `CONTRACTS.md` §3 and `SPEC.md` in the repo
- `playbook.md`

## Must

- Implement CONTRACTS.md §3 exactly: paths, `snake_case` fields, status codes and the error shape. The frontend builds against it, so any deviation breaks it silently.
- Validate every request at the `web` boundary, before it reaches the service.
- Pass values to SQL as named parameters, never by building query strings. Sort columns come from a closed set.
- Add a required column in steps (nullable, backfill, then required), so the migration runs on Alice's existing rows.
- Log ids and outcomes, never note text, company names or file contents.

## Must not

- Change the parser so the SPEC §12 acceptance figures change. If they move, the change is wrong.
- Read anything from an uploaded filename except for display.

## Done when

- `mvn -Dskip.frontend=true verify` passes.
- Every endpoint the story touches behaves as CONTRACTS.md §3 says.

## Escalate when

- CONTRACTS.md §3 or SPEC.md needs to change.
- A migration would lose or rewrite existing data.
