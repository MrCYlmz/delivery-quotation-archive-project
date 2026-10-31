---
name: vue3
description: Vue 3 with TypeScript, Vue Router and Vite conventions.
---

## Versions

Use the versions in `package.json` and follow the official Vue style guide. Much of the Vue code models learned from is Vue 2, so these older forms keep coming back:

- `<script setup lang="ts">` with the Composition API, never the Options API (`data()`, `methods`). Mixing both styles splits the codebase, and Vapor mode supports only the Composition API.
- No Vuex and no Pinia: page state lives in the page's composable (playbook decisions).
- None of the APIs Vue 3 removed: filters, `$on`/`$off` event buses, `this.$set`, the `.sync` modifier.
- `defineModel` for two-way binding instead of a prop plus an update event, and `useTemplateRef` for template refs.

## Reactivity

- Prefer `ref`. Destructuring or reassigning a `reactive` object silently drops reactivity.
- Destructured props stay reactive, but pass them to `watch` as a getter: `watch(() => size, …)`.
- Derived values are `computed`; `watch` is only for side effects outside Vue.

## Components

- Below pages, components use `defineProps` and `defineEmits` only.
- Never mutate a prop. Emit an event or use `defineModel`, because the parent owns the value.
- Type props and emits with TypeScript generics, so a wrong prop fails the build instead of the page.
- Never put `v-if` on the same element as `v-for`. In Vue 3, `v-if` runs first and cannot see the loop variable.
- Style components with `<style scoped>`, so their styles cannot leak into others.
- Load route pages lazily with `() => import(…)`, so users download only the pages they open.

## Checks

- `npm run typecheck` passes, and from story 1 on, ESLint with `eslint-plugin-vue` and `vitest run` pass too.
