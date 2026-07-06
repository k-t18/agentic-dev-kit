# Phase 1 — Monorepo & project audit

Before generating anything, inspect the current state of the repo.

## 1.1 Verify monorepo structure

Confirm these exist (create stubs if missing):

```
/packages/core/src/tokens/index.ts
/packages/core/src/tokens/rn-styles.ts
/apps/web/tailwind.config.ts
/apps/web/next.config.js
/apps/native/metro.config.js
```

## 1.2 Check the Tailwind relationship

Open `/apps/web/tailwind.config.ts`. Determine if it:
- ✅ Imports from `@repo/core/tokens` (correct — consumer)
- ❌ Defines token values inline (incorrect — tokens belong in `@repo/core`)

Also confirm `content` globs cover the app and the web UI package, e.g.:
```ts
content: ['./app/**/*.{ts,tsx}', './src/**/*.{ts,tsx}', '../../packages/ui-web/src/**/*.{ts,tsx}']
```
If inverted (defines values), flag for remediation in Phase 6.

## 1.3 Check Metro config

Open `/apps/native/metro.config.js`. Confirm `watchFolders` includes the monorepo
root; if missing, native will fail to resolve `@repo/core` imports.

```js
// Required in metro.config.js
watchFolders: [path.resolve(__dirname, '../..')],
resolver: {
  nodeModulesPaths: [
    path.resolve(__dirname, 'node_modules'),
    path.resolve(__dirname, '../../node_modules'),
  ],
}
```

## 1.4 Audit existing tokens for drift

Scan `/packages/ui-web` and `/packages/ui-native` for:
- Raw hex values (`#xxxxxx`)
- Raw pixel values (`px-[16px]`, `paddingHorizontal: 16`)
- Hardcoded font sizes, weights, or families
- Tailwind arbitrary values (`bg-[#3B82F6]`)
- Inline styles on web components (`style={{ color: 'red' }}`)

Document all drift instances for the Phase 8 report.

## 1.5 Audit output

Produce a summary:
- Monorepo structure: intact / missing files
- Tailwind relationship: correct (consumer) / inverted (defines values)
- Metro config: correct / missing watchFolders
- Token drift: count of violations found
- Recommended action: fresh extraction / update / audit-only
