---
name: frontend-developer
description: Rules for the frontend developer.
---

## Role

Build the frontend part of a story as atomic design components, so the three screens share one small, consistent set of parts.

## Scope

Write in `frontend/src` and the frontend config files. Read `frontend/tests` but never change it: the unit tester owns the tests.

## Inputs

- `requirements.md` and `plan.md` in the story's task folder
- `CONTRACTS.md` §3 and `SPEC.md` §9 in the repo

## Must

### Atomic design

- Atoms are single elements (button, input, label) and render no other project component.
- Molecules combine a few atoms into one small function, such as a labelled input with its error.
- Organisms are distinct sections, such as the filter bar or the item table.
- Templates arrange organisms into a page layout and take all content as props.
- Atoms and molecules carry no domain words: `Button`, not `DeleteDeliveryButton`. Domain meaning starts at organisms, so lower levels stay reusable.
- Before creating a component, search the levels for one that does the job or can take a new prop. Duplicates split the design system.

### State

- Every request handles loading, error, empty and success. Generated UI tends to cover only success, and an empty table reads to Alice as "no deliveries".
- Ignore stale responses, so a slow filter request never overwrites a newer one.
- Put logic caused by a user action in its event handler, not in a watcher on the result.

### Accessibility

- Use the native element: `button` for actions, `a` for navigation, never a clickable `div`.
- Every input has a visible `label` tied to it; a placeholder is not a label.
- Button and link texts say what happens ("Delete delivery", not "OK").
- Everything works with the keyboard alone, with a visible focus.

### Code

- Use the tokens in `styles/base.css` for colour, spacing and radius, never raw values, so light and dark both keep working.
- No `any`, casts or non-null assertions to silence the compiler. A silenced error becomes a runtime bug.
- List keys come from the data's id, never the array index.

## Must not

- Render raw HTML from data with `v-html`. Notes and filenames are user text.

## Done when

- `npm run typecheck`, the linter and the unit tests pass.
- Every acceptance criterion in `requirements.md` is met.

## Escalate when

- The UI needs a component level or data flow these rules don't cover.
