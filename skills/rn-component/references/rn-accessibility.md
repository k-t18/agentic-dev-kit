# Native accessibility

React Native uses `accessibility*` props (not ARIA). Map the web twin's a11y intent to
the native equivalent.

## ARIA → native mapping

```ts
// aria-label="x"        → accessibilityLabel="x"
// aria-hidden="true"    → importantForAccessibility="no-hide-descendants"
// aria-busy={loading}   → accessibilityState={{ busy: loading }}
// aria-disabled={x}     → accessibilityState={{ disabled: x }}
// aria-selected={x}     → accessibilityState={{ selected: x }}
// role="button"         → accessibilityRole="button"
// role="alert"          → accessibilityLiveRegion="assertive" (Android) + announce
// <label htmlFor>       → accessibilityLabel on the TextInput
```

## Rules

- Every touchable has an `accessibilityRole` (`button`, `link`, `checkbox`, …).
- Icon-only touchables **must** set `accessibilityLabel` (there's no visible text).
- Reflect state with `accessibilityState={{ busy, disabled, selected, checked }}`.
- Decorative elements: `importantForAccessibility="no"`.
- **Touch targets ≥ 44×44 pt.** If the visual is smaller, expand via `hitSlop`.
- Never convey state by color alone — pair with text or an icon (same as web).

```tsx
<TouchableOpacity
  onPress={onPress}
  disabled={isDisabled}
  accessibilityRole="button"
  accessibilityLabel={iconOnly ? label : undefined}
  accessibilityState={{ busy: loading, disabled: isDisabled }}
  hitSlop={{ top: 8, bottom: 8, left: 8, right: 8 }}
/>
```
