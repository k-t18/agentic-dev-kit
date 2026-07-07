---
name: rn-component
description: Use when creating or editing a native UI component in packages/ui-native (bare React Native CLI + StyleSheet). Build with RN primitives (every string inside <Text>), style with StyleSheet.create using rnTokens from @repo/core/tokens/rn-styles, express variants as keyed StyleSheet styles (no cva), and mirror the web twin's props verbatim as explicit unions (minus web-only props like className). Loading uses ActivityIndicator; a11y uses accessibilityRole/Label/State. Invoke for building a ui-native button, input, card, list, or any packages/ui-native component.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: ui-native, native component, React Native, StyleSheet, rnTokens, packages/ui-native, RN component, TouchableOpacity, native button
  role: specialist
  scope: implementation
  output-format: code
  related-skills: web-component, web-to-native, design-system-setup
---

# RN Component (packages/ui-native)

Builds a single native UI component in `packages/ui-native` (bare React Native CLI), styled exclusively with `rnTokens`, mirroring its web twin's props.

## Role Definition

Expert React Native (bare RN CLI, not Expo) component engineer building single, token-driven components in `packages/ui-native` with `StyleSheet` and `rnTokens` from `@repo/core/tokens/rn-styles`. Each component **mirrors its web twin** in `packages/ui-web` verbatim — same variant names/sizes re-expressed as explicit unions — so the two platforms stay parallel; builds with RN primitives (every string inside `<Text>`), keyed `StyleSheet` variants (no `cva`), and `accessibility*` props. Read root `CLAUDE.md` first; this skill never overrides it. Tokens come from `design-system-setup` — this skill _consumes_ `rn-styles.ts`; for converting an existing web component to native, use `web-to-native` (it references this skill's conventions).

## When to Use This Skill

- Building or editing a component in `packages/ui-native`.

**Not for:** web components (`web-component`), migrating a web component to native (`web-to-native`), or hooks/API/stores/types (`@repo/core`).

## Core Workflow

1. **Mirror the web twin's props verbatim.** Read `packages/ui-web/src/components/<Name>/<Name>.types.ts` and its `cva` config. Re-express the same variant names and sizes as **explicit unions** (native has no `cva`/`VariantProps`). Drop web-only props (`className`, `type`). Same names/values as web — divergence is drift.
2. **Build with RN primitives.** `View`/`Text`/`TouchableOpacity`/`TextInput`/`Image`/`FlatList`, etc. **Every string must be inside `<Text>`** — a bare string crashes RN. → `references/rn-primitives.md`
3. **Style with `StyleSheet.create`** at the bottom of the file; every value from `rnTokens`. Express variants as **keyed styles** (`styles[intent]`, `styles['label_'+intent]`) selected via a style array. → `references/rn-style-map.md`
4. **States:** `ActivityIndicator` for loading; `disabled` via style + `disabled` prop; error/empty per component type. → `references/rn-primitives.md`
5. **Accessibility:** `accessibilityRole`, `accessibilityLabel` (icon-only), `accessibilityState={{ busy, disabled }}`. → `references/rn-accessibility.md`
6. **Structure:** `Readonly<Props>` + `Name.displayName` + named export; folder `<Name>/<Name>.tsx` + `index.ts`; add `export * from './components/<Name>';` to `packages/ui-native/src/index.ts`. (Complex props → a `Name.types.ts`.)
7. **Validate:** `pnpm --filter @repo/ui-native exec tsc --noEmit` is clean; grep for raw hex/px, `%` widths, and CSS shorthand — there must be none.

## Technical Guidelines

### Dependencies

`react-native` (primitives + `StyleSheet`) and `rnTokens` from `@repo/core/tokens/rn-styles`. **No `cva`, no `cn`, no `className`.** Icons are passed in as `ReactNode` slots (`leftIcon`/`rightIcon`) — this skill prescribes no icon library.

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Tailwind intent → StyleSheet property + rnTokens | `references/rn-style-map.md` | Translating a style |
| RN primitives, text rule, lists, states | `references/rn-primitives.md` | Choosing elements / handling states |
| `accessibility*` props | `references/rn-accessibility.md` | Wiring a11y |

### Canonical Pattern

```tsx
// packages/ui-native/src/components/Button/Button.tsx
import { ActivityIndicator, StyleSheet, Text, TouchableOpacity } from 'react-native';
import type { ReactNode } from 'react';
import { rnTokens } from '@repo/core/tokens/rn-styles';

// Mirrors the web twin's props (minus web-only className/type); variants as explicit unions.
export interface ButtonProps {
  label: string;
  intent?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  onPress?: () => void;
  disabled?: boolean;
  loading?: boolean;
  leftIcon?: ReactNode;
}

export function Button({
  label,
  intent = 'primary',
  size = 'md',
  onPress,
  disabled = false,
  loading = false,
  leftIcon,
}: Readonly<ButtonProps>) {
  const isDisabled = disabled || loading;
  const spinnerColor = intent === 'primary' ? rnTokens.colors.white : rnTokens.colors.primary;
  return (
    <TouchableOpacity
      onPress={onPress}
      disabled={isDisabled}
      activeOpacity={0.8}
      accessibilityRole="button"
      accessibilityState={{ busy: loading, disabled: isDisabled }}
      style={[styles.base, styles[`size_${size}`], styles[intent], isDisabled && styles.disabled]}
    >
      {loading ? (
        <ActivityIndicator size="small" color={spinnerColor} style={styles.spinner} />
      ) : (
        leftIcon ?? null
      )}
      <Text style={[styles.label, styles[`label_${intent}`]]}>{label}</Text>
    </TouchableOpacity>
  );
}

Button.displayName = 'Button';

const styles = StyleSheet.create({
  base: { flexDirection: 'row', alignItems: 'center', justifyContent: 'center', borderRadius: rnTokens.radius.md },
  size_sm: { paddingHorizontal: rnTokens.spacing[3], paddingVertical: rnTokens.spacing[1] },
  size_md: { paddingHorizontal: rnTokens.spacing[4], paddingVertical: rnTokens.spacing[2] },
  size_lg: { paddingHorizontal: rnTokens.spacing[6], paddingVertical: rnTokens.spacing[3] },
  primary: { backgroundColor: rnTokens.colors.primary },
  secondary: { backgroundColor: rnTokens.colors.surfaceCanvas, borderWidth: 1, borderColor: rnTokens.colors.primary },
  ghost: { backgroundColor: rnTokens.colors.transparent },
  disabled: { opacity: 0.5 },
  label: { fontSize: rnTokens.typography.size.md, fontWeight: rnTokens.typography.weight.medium },
  label_primary: { color: rnTokens.colors.white },
  label_secondary: { color: rnTokens.colors.primary },
  label_ghost: { color: rnTokens.colors.primary },
  spinner: { marginRight: rnTokens.spacing[2] },
});
```

```ts
// packages/ui-native/src/components/Button/index.ts
export { Button } from './Button';
export type { ButtonProps } from './Button';
```

```ts
// packages/ui-native/src/index.ts  — package barrel
export * from './components/Button';
```

## Constraints

### MUST DO

- Mirror the web twin's props **verbatim** — same variant names/sizes as explicit unions (no `VariantProps`), minus web-only props (`className`, `type`).
- Build with RN primitives only; put **every string inside `<Text>`** (bare strings crash RN).
- Call `StyleSheet.create` at the file bottom; source **every value from `rnTokens`**.
- Express variants as **keyed styles** selected via a style array (no `cva`).
- Use `ActivityIndicator` for loading and `accessibility*` props for a11y.
- Use a named export + `Readonly<Props>` + `displayName`; add a folder `index.ts` + package barrel.

### MUST NOT DO

- Use `className`, `cn`, `cva`, or Tailwind.
- Use raw hex/px, `%` widths (use `Dimensions`/Flexbox), or CSS shorthand (expand it).
- Import `react-native` into `@repo/core`, or modify `@repo/core`.

## Output Templates

When building a `ui-native` component, provide:

1. The component folder — `Name.tsx` + `index.ts` — plus the package-barrel export line.
2. Props mirroring the web twin verbatim as explicit unions (minus web-only props).
3. `StyleSheet.create` at the file bottom, every value from `rnTokens`, variants as keyed styles.
4. Loading via `ActivityIndicator` and `accessibilityRole`/`Label`/`State` wiring.

## Knowledge Reference

Bare React Native CLI, StyleSheet, rnTokens, @repo/core/tokens/rn-styles, RN primitives, View, Text, TouchableOpacity, TextInput, FlatList, ActivityIndicator, keyed StyleSheet variants, explicit unions, accessibilityRole, accessibilityLabel, accessibilityState, packages/ui-native, web twin parity, Readonly props, displayName
