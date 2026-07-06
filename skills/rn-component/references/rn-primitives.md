# RN primitives, text, lists, states

## Element usage

| Purpose | Primitive |
| --- | --- |
| Layout container | `View` |
| Any text | `Text` (all text must be wrapped) |
| Pressable | `TouchableOpacity` (or `Pressable` for press states) |
| Text input | `TextInput` (`onChangeText`, not `onChange`) |
| Image | `Image` — **explicit `width` + `height` required** |
| Long/dynamic list | `FlatList` (never `.map()` inside a `ScrollView`) |
| Short static list | `View` + mapped `View` children |
| Scroll area | `ScrollView` |
| Loading spinner | `ActivityIndicator` |

## The text rule (critical)

Every string must live inside `<Text>`. A bare string in a `View` **crashes** RN.

```tsx
// ❌ crashes
<View>Hello</View>
// ✅
<View><Text>Hello</Text></View>
```

## Icons

Icons are passed in as `ReactNode` slots — the component renders whatever it's given;
it does not import an icon library.

```tsx
export interface CardProps { leftIcon?: ReactNode; }
// …
{leftIcon ?? null}
```

The loading indicator is `ActivityIndicator`, not an icon.

## Images & lists

```tsx
<Image source={{ uri }} style={{ width: 40, height: 40 }} />   // explicit dimensions

<FlatList
  data={items}
  keyExtractor={(it) => it.id}
  renderItem={({ item }) => <Row item={item} />}
/>
```

## States

- **Loading** — `ActivityIndicator` + `disabled` interaction + `accessibilityState={{ busy: true }}`.
- **Disabled** — conditional style + `disabled` prop on the touchable.
- **Error** (inputs) — error-colored border + a `<Text>` message; wire a11y (see
  `rn-accessibility.md`).
- **Empty / populated** (data) — render an empty `View`+`Text` block when the list is
  empty; the async loading/error/empty decision comes from the `@repo/core` hook
  (`isLoading`/`isError`/`!data`) exactly as on web — the hook is unchanged.

```tsx
if (isLoading) return <LoadingSkeleton />;
if (isError)   return <ErrorState message={error.message} />;
if (!data)     return <EmptyState />;
return <PopulatedView data={data} />;
```
