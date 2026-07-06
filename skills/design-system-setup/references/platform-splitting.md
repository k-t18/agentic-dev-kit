# Phase 4 — Platform-aware token splitting

Before writing any file, split token categories that behave differently across web
and native. This is unique to the dual-target monorepo.

## 4.1 Shadow splitting (mandatory)

Web and native represent shadows entirely differently. Store both in `index.ts`:

```ts
// tokens/index.ts — store both representations under one semantic name
shadow: {
  sm: {
    web: '0 1px 2px 0 rgba(0,0,0,0.05)',
    native: {
      shadowColor: '#000000',
      shadowOffset: { width: 0, height: 1 },
      shadowOpacity: 0.05,
      shadowRadius: 2,
      elevation: 1,        // Android
    },
  },
  md: {
    web: '0 4px 6px -1px rgba(0,0,0,0.1)',
    native: {
      shadowColor: '#000000',
      shadowOffset: { width: 0, height: 4 },
      shadowOpacity: 0.1,
      shadowRadius: 6,
      elevation: 4,
    },
  },
  // …lg, etc.
}
```

`rn-styles.ts` consumes `.native` only; Tailwind consumes `.web` only.

## 4.2 Typography / font-loading split

Web fonts load via CSS `@font-face`/CDN or `next/font` in `/apps/web`. Native fonts are
linked via `react-native.config.js` and placed in `/apps/native/assets/fonts/`.

Store the family name as a string in `index.ts`:

```ts
typography: {
  family: {
    sans: 'YourSans',   // Web: CSS font-family / next/font. Native: fontFamily in StyleSheet.
    mono: 'YourMono',
  },
}
```

In `rn-styles.ts` the family value must exactly match the **linked font's PostScript
name** (iOS is case-sensitive). Document the required font files under
`/apps/native/assets/fonts/` in the Phase 8 output.

## 4.3 Spacing & radius (no split)

Plain numeric values — compatible with both Tailwind (numeric scale) and native
(`StyleSheet` accepts numbers directly):

```ts
spacing: { 1: 4, 2: 8, 3: 12, 4: 16, 6: 24, 8: 32, 12: 48, 16: 64 }
radius:  { sm: 4, md: 8, lg: 12, xl: 16, full: 9999 }
```

## 4.4 Percentage widths (native forbidden)

Never include `%` values in `rn-styles.ts`:

```ts
// Note: % widths are invalid in RN StyleSheet.
// Use Dimensions.get('window').width * 0.x or Flexbox instead.
```

## 4.5 CSS shorthand (native forbidden)

```ts
// ❌ WRONG for native
padding: '16px 24px'
border: '1px solid #ccc'
font: 'bold 16px/1.5 YourSans'

// ✅ CORRECT for native
paddingVertical: 16,
paddingHorizontal: 24,
borderWidth: 1,
borderColor: colors.border.default,
fontWeight: '700',
fontSize: 16,
lineHeight: 24,
fontFamily: 'YourSans',
```
