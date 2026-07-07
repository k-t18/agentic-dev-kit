---
name: zustand-slice
description: Scaffold a packages/core/src/state/<name>Store.ts Zustand slice, or append a field/action to an existing one. Runs the zustand-slice skill.
argument-hint: "[new store <name>, or 'add <field/action> to <store>']"
---

# /zustand-slice

**Arguments:** $ARGUMENTS

Runs the **`zustand-slice`** skill (global client-state stores in `@repo/core/state`). Never
overwrites existing stores — appends only. `@repo/core` stays platform-agnostic.

## What to do

1. **Parse `$ARGUMENTS`:**
   - `new store <name>` (or a request to create a store) → **scaffold mode**: generate
     `state/<name>Store.ts` + barrel export per `references/store-anatomy.md`.
   - `add <field/action> to <store>` (or a described change) → **extend mode**: **append** the
     field + action (+ optional selector) to the existing `state/<name>Store.ts`. If a field is
     ambiguous (type, default, whether to persist), **ask before generating**.
   - Empty → ask whether to scaffold a store or extend one, and for the input.

2. **Generate per the skill:**
   - **Scaffold** → the state `interface` (state + colocated actions), the `create()` store
     (wrap in `persist` with `createJSONStorage(() => getStateStorage())` only if it should
     survive reloads), named selector hooks, and the `state/index.ts` barrel export.
   - **Extend** → append the new member(s) to the existing interface + store + (optional)
     selector. Leave existing members untouched.

3. **Enforce the boundaries** — global **client** state only (server data → `feature-slice`,
   local/Context → `react-renderer`); `@repo/core` platform-agnostic (persist via injected
   `getStateStorage()`, never `localStorage`/MMKV/`react-native`); narrow selectors; immutable
   updates; no `any`; named exports + barrel.

4. **Print the summary** of files created/appended (note any skipped because they existed), and
   remind that a **new persisted store** needs `configureStateStorage()` wired at app startup
   (web localStorage / native MMKV) — see `references/persist-and-storage.md`.
