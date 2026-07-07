---
name: react-native-expert
description: React Native discipline for this monorepo (bare RN CLI, not Expo) — navigation with React Navigation (tabs, stacks, drawers, deep linking), platform handling (SafeArea, keyboard, StatusBar, Android back, .ios/.android splits), FlatList/SectionList/FlashList performance, native modules, memory/subscriptions, and native view-local storage. Defers component building to rn-component, web→native ports to web-to-native, data/API to feature-slice, global state to zustand-slice, tokens to design-system-setup, and effect/memo rules to react-renderer. Use for navigation, scroll performance, platform-specific code, or native-module wiring.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "2.0.0"
  domain: frontend
  triggers: React Native, bare RN CLI, React Navigation, mobile app, iOS, Android, cross-platform, native module, FlatList, SafeArea, keyboard, deep linking
  role: specialist
  scope: implementation
  output-format: code
  related-skills: rn-component, web-to-native, feature-slice, design-system-setup, react-renderer, zustand-slice
---

# React Native Expert (discipline)

Cross-cutting React Native know-how for our **bare React Native CLI** native app:
**navigation, platform handling, list performance, native modules, memory, and native
view-local storage.** It does **not** build components or own data/state/tokens — those
have skills.

Read root `CLAUDE.md` first; this skill never overrides it. Native is **React 18**.

- Building a `packages/ui-native` component → `rn-component` (web twin → `web-component`).
- Porting a web component to native → `web-to-native`.
- Hooks / API / server data → `feature-slice`; global store → `zustand-slice`.
- Design tokens (`rnTokens`) → `design-system-setup`; effect/memo/perf rules → `react-renderer`.

> **Bare RN CLI, not Expo.** No `expo`, `expo-router`, or `@expo/*`. Navigation is
> **React Navigation**; styling is `StyleSheet` with `rnTokens` from
> `@repo/core/tokens/rn-styles` — never hardcoded hex/px.

## Core Workflow

1. **Setup** — React Navigation, TypeScript config → _verify the monorepo builds: `pnpm install` and confirm `metro.config.js` keeps its `watchFolders` (design-system-setup) before proceeding_
2. **Structure** — the native app is `apps/<native>` consuming `@repo/ui-native` + `@repo/core`
3. **Implement** — components from `rn-component`, wired with platform handling → _run on iOS simulator and Android emulator; check Metro output for errors before moving on_
4. **Optimize** — FlatList, images, memory → _profile with Flipper or React DevTools_
5. **Test** — Both platforms, real devices

### Error Recovery
- **Metro bundler errors** → reset cache with `npx react-native start --reset-cache`, then restart
- **iOS build fails** → check Xcode logs → resolve the native dependency / pods → `npx pod-install` → rebuild with `pnpm --filter <native> ios` (`npx react-native run-ios`)
- **Android build fails** → check `adb logcat` or Gradle output → resolve SDK/NDK version mismatch → rebuild with `pnpm --filter <native> android` (`npx react-native run-android`)
- **Native module not found** → install it, re-run `npx pod-install` (iOS), then rebuild the native layers

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Navigation | `references/navigation.md` | React Navigation — tabs, stacks, params, deep linking |
| Platform | `references/platform-handling.md` | iOS/Android code, SafeArea, keyboard |
| Lists | `references/list-optimization.md` | FlatList, performance, memo |
| Storage | `references/storage-hooks.md` | Native view-local prefs (durable data → offline-kit) |
| Structure | `references/project-structure.md` | Monorepo native-app layout, Metro |

## Constraints

### MUST DO
- Use FlatList/SectionList for lists (not ScrollView)
- Implement memo + useCallback for list items
- Handle SafeAreaView for notches
- Test on both iOS and Android real devices
- Use KeyboardAvoidingView for forms
- Handle Android back button in navigation
- Style with `rnTokens` from `@repo/core/tokens/rn-styles`; navigate with React Navigation
- Build components via `rn-component`; get server data via `@repo/core` hooks (`feature-slice`)

### MUST NOT DO
- Use ScrollView for large lists
- Use inline styles extensively (creates new objects)
- Hardcode dimensions or colors — use `rnTokens` (dimensions: `Dimensions`/flex, never `%`)
- Ignore memory leaks from subscriptions
- Skip platform-specific testing
- Use waitFor/setTimeout for animations (use Reanimated)
- Introduce Expo / `expo-router` / `@expo/*` — this is a bare RN CLI app
- Redefine component / data / state / token concerns owned by the other skills

## Code Examples

> These are **discipline** examples (perf / platform wiring). The list rows, forms, and chips
> shown here are real `packages/ui-native` components built per **`rn-component`** (every
> string in `<Text>`, keyed-StyleSheet variants, props mirroring the web twin, a11y). This
> skill owns the *list container, navigation, and platform behaviour* around them.

### Optimized FlatList with memo + useCallback

```tsx
import React, { memo, useCallback } from 'react';
import { FlatList, View, Text, StyleSheet } from 'react-native';
import { rnTokens } from '@repo/core/tokens/rn-styles';

type Item = { id: string; title: string };

// A trivial inline row for illustration; a real row is a ui-native component (rn-component).
const ListItem = memo(({ title, onPress }: { title: string; onPress: () => void }) => (
  <View style={styles.item}>
    <Text onPress={onPress}>{title}</Text>
  </View>
));

export function ItemList({ data, onSelect }: { data: Item[]; onSelect: (id: string) => void }) {
  const handlePress = useCallback((id: string) => onSelect(id), [onSelect]);

  const renderItem = useCallback(
    ({ item }: { item: Item }) => (
      <ListItem title={item.title} onPress={() => handlePress(item.id)} />
    ),
    [handlePress]
  );

  return (
    <FlatList
      data={data}
      keyExtractor={(item) => item.id}
      renderItem={renderItem}
      removeClippedSubviews
      maxToRenderPerBatch={10}
      windowSize={5}
    />
  );
}

const styles = StyleSheet.create({
  item: { padding: rnTokens.spacing[4], borderBottomWidth: StyleSheet.hairlineWidth },
});
```

### KeyboardAvoidingView Form

```tsx
import React from 'react';
import {
  KeyboardAvoidingView,
  Platform,
  ScrollView,
  StyleSheet,
  SafeAreaView,
} from 'react-native';
import { rnTokens } from '@repo/core/tokens/rn-styles';
import { Input } from '@repo/ui-native'; // the field is a ui-native component (rn-component)

export function LoginForm() {
  return (
    <SafeAreaView style={styles.safe}>
      <KeyboardAvoidingView
        style={styles.flex}
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      >
        <ScrollView contentContainerStyle={styles.content} keyboardShouldPersistTaps="handled">
          <Input placeholder="Email" autoCapitalize="none" />
          <Input placeholder="Password" secureTextEntry />
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  safe: { flex: 1 },
  flex: { flex: 1 },
  content: { padding: rnTokens.spacing[4], gap: rnTokens.spacing[3] },
});
```

### Platform-Specific Component

```tsx
import { StyleSheet, View, Text } from 'react-native';
import { rnTokens } from '@repo/core/tokens/rn-styles';

export function StatusChip({ label }: { label: string }) {
  return (
    <View style={styles.chip}>
      <Text style={styles.label}>{label}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  chip: {
    paddingHorizontal: rnTokens.spacing[3],
    paddingVertical: rnTokens.spacing[1],
    borderRadius: rnTokens.radius.full,
    backgroundColor: rnTokens.colors.primary,
    // Shadows are pre-split by design-system-setup ({ web, native }); spread the native token —
    // no hand-written Platform.select. Reserve Platform.select for genuinely divergent props.
    ...rnTokens.shadow.chip,
  },
  label: {
    color: rnTokens.colors.white,
    fontSize: rnTokens.typography.size.sm,
    fontWeight: rnTokens.typography.weight.semibold,
  },
});
```

## Output Format

When implementing React Native features, deliver:
1. **Component code** — TypeScript, with prop types defined
2. **Platform handling** — `Platform.select` or `.ios.tsx` / `.android.tsx` splits as needed
3. **Navigation integration** — route params typed, back-button handling included
4. **Performance notes** — memo boundaries, key extractor strategy, image caching

## Knowledge Reference

Bare React Native CLI (0.73+, React 18), React Navigation 7, Reanimated 3, Gesture Handler,
MMKV / AsyncStorage (native view-local prefs). Data → `@repo/core` React Query
(`feature-slice`); global state → `zustand-slice`; durable/offline data → `@repo/offline-kit`;
components → `rn-component`; tokens → `design-system-setup`.
