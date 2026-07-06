# Tailwind intent → StyleSheet + rnTokens

Native has no Tailwind. Express the same design intent in `StyleSheet.create` using
`rnTokens` from `@repo/core/tokens/rn-styles`. Every value comes from a token.

## Colors

```ts
// bg-primary          → backgroundColor: rnTokens.colors.primary
// bg-surface-canvas   → backgroundColor: rnTokens.colors.surfaceCanvas
// bg-surface-subtle   → backgroundColor: rnTokens.colors.surfaceSubtle
// text-white          → color: rnTokens.colors.white
// text-text-primary   → color: rnTokens.colors.textPrimary
// text-text-secondary → color: rnTokens.colors.textSecondary
// border-border-default → borderColor: rnTokens.colors.borderDefault
// text-error / success → color: rnTokens.colors.error / rnTokens.colors.success
```

## Spacing (grid index, not px)

```ts
// p-4  → padding: rnTokens.spacing[4]
// px-4 → paddingHorizontal: rnTokens.spacing[4]
// py-2 → paddingVertical: rnTokens.spacing[2]
// mt-4 → marginTop: rnTokens.spacing[4]
// gap-2 → gap: rnTokens.spacing[2]        // RN 0.71+
// pattern: p-{n}/m-{n} → rnTokens.spacing[n]
```

## Typography

```ts
// text-sm/md    → fontSize: rnTokens.typography.size.sm / .md
// font-medium   → fontWeight: rnTokens.typography.weight.medium
// font-bold     → fontWeight: rnTokens.typography.weight.bold
// font-sans     → fontFamily: rnTokens.typography.family.sans
// text-center   → textAlign: 'center'
// uppercase     → textTransform: 'uppercase'
// truncate      → numberOfLines={1} ellipsizeMode="tail"  (props on <Text>, not a style)
```

## Border · radius · shadow

```ts
// border      → borderWidth: 1
// border-2    → borderWidth: 2
// border-t    → borderTopWidth: 1
// rounded-md  → borderRadius: rnTokens.radius.md
// rounded-full→ borderRadius: rnTokens.radius.full
// shadow-sm   → ...rnTokens.shadow.sm    // spread — includes iOS shadow* + Android elevation
```

## Layout

```ts
// flex-row        → flexDirection: 'row'
// items-center    → alignItems: 'center'
// justify-center  → justifyContent: 'center'
// justify-between → justifyContent: 'space-between'
// flex-1          → flex: 1
// w-full          → alignSelf: 'stretch'      // preferred over width: '100%'
// w-10 / h-10     → width: 40 / height: 40    // 1 unit = 4px
// overflow-hidden → overflow: 'hidden'
// absolute        → position: 'absolute'
// inset-0         → top: 0, right: 0, bottom: 0, left: 0
// opacity-50      → opacity: 0.5
// hidden          → conditional render (preferred) or display: 'none'
```

## No RN equivalent — handle explicitly

```ts
// hover:*         → Pressable style={({ pressed }) => [base, pressed && styles.pressed]}
// focus-visible:* → onFocus/onBlur state + conditional style (inputs)
// disabled:*      → style={[styles.base, isDisabled && styles.disabled]}
// transition-*    → Animated API or react-native-reanimated
// % widths        → Dimensions.get('window').width * 0.x  OR  Flexbox
// CSS shorthand   → always expand: padding:'16 8' → paddingVertical, paddingHorizontal
```
