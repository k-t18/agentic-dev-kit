---
name: feature-slice
description: Scaffold a packages/core/features/<x> vertical slice, or add an API endpoint from a pasted curl/JSON/field-list. Runs the feature-slice skill.
argument-hint: "[paste curl/JSON/field-list, or 'new slice <name>']"
---

# /feature-slice

**Arguments:** $ARGUMENTS

Runs the **`feature-slice`** skill (core layering invariant, Frappe-shaped). Never overwrites
existing files — appends only. `@repo/core` data layer only.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A curl / Frappe URL / raw JSON / field-list / plain-English endpoint description →
     **add-endpoint mode**. Follow `feature-slice` → `references/parse-input.md`: extract
     domain/method/params/response, print the parse summary, and (if a field is ambiguous)
     **ask before generating**.
   - `new slice <name>` (or a request to scaffold a whole feature) → **scaffold mode**:
     generate `features/<name>/` per `references/slice-anatomy.md`.
   - Empty → ask whether to scaffold a slice or add an endpoint, and for the input.

2. **Generate per the skill:**
   - **Add-endpoint** → append a type (`types/index.ts`) + `endpoints.ts` entry
     (`buildEndpoint`) + `data/remote.ts` method + `repo.ts` method + `hooks.ts` hook
     (+ barrel). Create the slice folder first if missing.
   - **Scaffold** → the full slice (`data/remote.ts` + `data/local.ts` + `repo.ts` +
     `hooks.ts` + `usecases.ts`/`outbox.ts` if it writes + `index.ts`).

3. **Enforce the layering invariant** — `getOfflineDb`/SQL/HTTP only in
   `data/**`/`usecases.ts`/`outbox.ts`; hooks are glue only; `GET`→`useApiQuery`,
   writes→`useApiMutation` (online) or `usecases`+`outbox` (offline).

4. **Print the summary** of files created/appended (note any skipped because they existed).
