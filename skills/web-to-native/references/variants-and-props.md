# Variants & props migration

## Props — mirror the web twin verbatim

Native has no `cva`/`VariantProps`, so re-express the **same** variant names and sizes
as **explicit unions**. Drop web-only props (`className`, `type`). Everything else
carries over unchanged. Divergence from the web variant set is drift — keep them parallel.

```tsx
// Web — packages/ui-web/src/components/Button/Button.types.ts
export interface ButtonProps extends VariantProps<typeof buttonVariants> {
  label: string; onPress?: () => void; disabled?: boolean; loading?: boolean;
  leftIcon?: ReactNode; type?: 'button' | 'submit' | 'reset'; className?: string;
}

// Native — inline in Button.tsx (or Button.types.ts if complex)
export interface ButtonProps {
  label: string;
  intent?: 'primary' | 'secondary' | 'ghost';   // ← the web cva `intent` union, by hand
  size?: 'sm' | 'md' | 'lg';                     // ← the web cva `size` union, by hand
  onPress?: () => void;
  disabled?: boolean;
  loading?: boolean;
  leftIcon?: ReactNode;                          // ReactNode slot carries over
  // dropped: className, type (web-only)
}
```

Read the web component's `cva` config to get the exact variant keys and values — those
become the explicit unions.

## Variants — cva → keyed StyleSheet

Convert the `cva` variant map into `StyleSheet` entries **keyed by variant value**, then
select them with a style array (this is the `rn-component` pattern).

```tsx
// Web — cva
const buttonVariants = cva('base', {
  variants: { intent: { primary: 'bg-primary text-white', ghost: 'bg-transparent text-primary' } },
  defaultVariants: { intent: 'primary' },
});
<button className={cn(buttonVariants({ intent }), className)} />

// Native — keyed StyleSheet + style array
<TouchableOpacity style={[styles.base, styles[intent], styles[`size_${size}`], isDisabled && styles.disabled]}>
  <Text style={[styles.label, styles[`label_${intent}`]]}>{label}</Text>
</TouchableOpacity>

const styles = StyleSheet.create({
  base:          { flexDirection: 'row', alignItems: 'center', justifyContent: 'center' },
  primary:       { backgroundColor: rnTokens.colors.primary },
  ghost:         { backgroundColor: rnTokens.colors.transparent },
  label_primary: { color: rnTokens.colors.white },
  label_ghost:   { color: rnTokens.colors.primary },
  disabled:      { opacity: 0.5 },
});
```

- Base classes → `styles.base`. Each variant value → its own keyed style.
- `defaultVariants` → default parameter values (`intent = 'primary'`).
- `compoundVariants` → a conditional style pushed into the array when the combo matches.
- For the class → StyleSheet **property** mapping (colors/spacing/typography/shadow/
  layout, and the no-RN-equivalent cases), use `rn-component/references/rn-style-map.md`.
