---
name: typescript-pro
description: Designs TypeScript types for this monorepo — feature-local vs shared placement, one shared local+remote entity contract, snake_case response/DTO interfaces, discriminated unions for state, type guards, and utility types. Use when modeling types, adding a response/DTO interface, deciding where a type lives, or enforcing type safety across packages.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.1.0"
  domain: language
  triggers: TypeScript, generics, type safety, conditional types, mapped types, tsconfig, type guards, discriminated unions
  role: specialist
  scope: implementation
  output-format: code
  related-skills: feature-slice, web-component
---

# TypeScript Pro

> Follows the project's TypeScript conventions in root `CLAUDE.md`. **Types placement:** a
> feature's own types live with its slice in `packages/core/features/<x>/<x>.types.ts`
> (exported via the feature barrel); the global `packages/core/src/types/` holds **only
> genuinely-shared** types (used by ≥2 features or app-wide, e.g. `User`, `ApiError`).
> `interface` for object shapes, `type` for unions; `unknown` not `any`; API response
> types (snake_case, mirroring the backend) defined before their hook.

## Core Workflow

1. **Analyze type architecture** - Review `tsconfig.base.json`, type coverage, build performance
2. **Design types (placement first)** - Put a feature's types in its slice
   (`<x>.types.ts`); reserve the global `types/` for shared. Define API response types
   before the hook; use one shared local+remote entity contract → `references/domain-types.md`
3. **Implement with type safety** - Write type guards, discriminated unions, conditional types; run `tsc --noEmit` to catch type errors before proceeding
4. **Optimize build** - Configure project references, incremental compilation, tree shaking; re-run `tsc --noEmit` to confirm zero errors after changes
5. **Test types** - Confirm type coverage with a tool like `type-coverage`; validate that all public APIs have explicit return types; iterate on steps 3–4 until all checks pass

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Our type patterns (placement, one local+remote contract, triads, discriminated unions) | `references/domain-types.md` | Designing a feature's or API's types |

## Code Examples

### Entity contract (one shape for local + remote)
```typescript
// features/order/order.types.ts — a feature's own types live with its slice.

/** `get_orders` row — the shared contract for the API response AND the local table.
 *  snake_case, mirrors the (Frappe) backend. The local table stores the same fields,
 *  so there is no separate Local* interface and no mapper. */
export interface Order {
  name: string;            // PK — same on the server and in the local table
  customer: string;
  total_amount: number;
  status?: string;         // optional — absent on some responses
}
// repo.listRemote() and repo.listLocal() both return Promise<Order[]>.
```

### Discriminated Unions & Type Guards
```typescript
type LoadingState = { status: "loading" };
type SuccessState = { status: "success"; data: string[] };
type ErrorState   = { status: "error";   error: Error };
type RequestState = LoadingState | SuccessState | ErrorState;

// Type predicate guard
function isSuccess(state: RequestState): state is SuccessState {
  return state.status === "success";
}

// Exhaustive switch with discriminated union
function renderState(state: RequestState): string {
  switch (state.status) {
    case "loading": return "Loading…";
    case "success": return state.data.join(", ");
    case "error":   return state.error.message;
    default: {
      const _exhaustive: never = state;
      throw new Error(`Unhandled state: ${_exhaustive}`);
    }
  }
}
```

### Utility types
Prefer the built-ins (`Partial`, `Pick`, `Omit`, `Record`); define small custom utilities
inline where genuinely needed.

### tsconfig
Extend the monorepo's shared `tsconfig.base.json` — never hand-roll a separate config.
Keep `strict: true` and all strict flags on.

## Constraints

### MUST DO
- Enable strict mode with all compiler flags (extend `tsconfig.base.json`)
- Keep a feature's types in its slice (`<x>.types.ts`); global `types/` only for shared
- `interface` for object shapes, `type` for unions/aliases
- Define API response types (snake_case, mirroring the backend) before the hook
- Use one contract for a feature's local + remote data (no divergent `Local*` shape or mapper)
- Use `satisfies` for validation; discriminated unions for state
- Use type predicates for narrowing; optimize for inference

### MUST NOT DO
- Use `any` — use `unknown` and narrow (unknown id → `string`)
- Dump a feature's types into the global `types/` file — keep them feature-local
- Redefine a shared type in an app package — import from `@repo/core`
- Disable strict null checks; mix type-only and value imports
- Use `as` to silence errors (only where genuinely necessary)
- Use enums (prefer `as const` objects)

## Output Templates

When implementing TypeScript features, provide:
1. Type definitions (interfaces, types, generics)
2. Implementation with type guards
3. tsconfig configuration if needed
4. Brief explanation of type design decisions

## Knowledge Reference

TypeScript 5.0+, generics, conditional types, mapped types, template literal types, discriminated unions, type guards, project references, incremental compilation, declaration files, const assertions, satisfies operator
