# Migration-ready props

In `web+native` projects the props interface is **reused verbatim** in the native
counterpart (`packages/ui-native`) — only JSX and styling change during migration. Name
props so that migration is seamless. (See the `web-to-native` skill.)

## Migration-aware prop naming

| Web HTML convention | Use instead | Why |
| --- | --- | --- |
| `onClick` | `onPress` | React Native uses `onPress` |
| `children` for simple text | `label` prop | Native `Text` needs a string, not JSX children |
| `className` | keep as optional | Native ignores it — a web-only escape hatch |
| `onChange` on input | `onChangeText` | Native `TextInput` uses `onChangeText` |
| `placeholder` | `placeholder` | Same — no change |
| `value` | `value` | Same — no change |

**Exception:** composable containers/wrappers may use `children: ReactNode` — native
supports children too.

## Interface rules

- Props `interface` in a separate `Name.types.ts`, exported, **extends
  `VariantProps<typeof nameVariants>`**.
- Migration-ready: the same interface must work in native (no web-only types in the
  shared shape; keep `className` optional).

```ts
export interface ButtonProps extends VariantProps<typeof buttonVariants> {
  label: string;           // text content — not children
  onPress?: () => void;    // not onClick
  disabled?: boolean;
  loading?: boolean;
  leftIcon?: ReactNode;
  className?: string;      // web-only override — native ignores it
}
```

## Type rules

- No `any` — use `unknown` and narrow.
- No `as SomeType` casts to silence errors — fix the type.
- Discriminated unions for mutually-exclusive multi-state props.
- Shared domain types imported from `@repo/core/types` — never redefined locally.

```ts
// ✅ discriminated union — states are explicit
type ButtonState = { loading: true; disabled?: never } | { loading?: false; disabled?: boolean };

// ❌ ambiguous — can both be true?
interface Props { loading?: boolean; disabled?: boolean }
```
