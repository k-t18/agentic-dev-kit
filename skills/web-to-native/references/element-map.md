# HTML element → RN primitive

| Web | Native | Notes |
| --- | --- | --- |
| `div` | `View` | |
| `span` (layout) | `View` | |
| `span` / `p` / `h1`–`h6` / `label` | `Text` | all text nodes need `<Text>` |
| `button` | `TouchableOpacity` + `Text` | |
| `input[text]` | `TextInput` | `onChangeText`, not `onChange` |
| `input[password]` | `TextInput secureTextEntry` | |
| `textarea` | `TextInput multiline` | |
| `img` | `Image` | explicit `width` + `height` required |
| `ul`/`ol` + map (long/dynamic) | `FlatList` | never `.map()` inside a `ScrollView` |
| `ul`/`ol` + map (short static) | `View` + mapped `View` children | |
| scroll area | `ScrollView` | |
| `a` / `<Link>` | `TouchableOpacity` + `navigation.navigate` | see imports-and-events |
| `form` | `View` | no form element in RN |
| `hr` | `View` with `height: 1` + `backgroundColor` | |

## The text rule (critical)

Every string must be inside `<Text>`. A bare string in a `View` **crashes** RN.

```tsx
// ❌ crashes
<View>Hello</View>
// ✅
<View><Text>Hello</Text></View>
```

Watch for interpolated strings, conditional text, and `{' '}` spacers — all must be
inside a `<Text>`.
