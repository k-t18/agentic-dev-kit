# Platform Handling

## Styling: tokens first (shadows/fonts are already platform-split)

Shadows and fonts are pre-split per platform by `design-system-setup` (the shadow token is
`{ web, native }`; the native object carries iOS `shadow*` and Android `elevation`). So spread
the token — **don't hand-write `Platform.select` for shadows or hardcode font names**.

```typescript
import { StyleSheet } from 'react-native';
import { rnTokens } from '@repo/core/tokens/rn-styles';

const styles = StyleSheet.create({
  card: {
    padding: rnTokens.spacing[4],
    borderRadius: rnTokens.radius.md,
    backgroundColor: rnTokens.colors.surfaceCanvas,
    ...rnTokens.shadow.card, // pre-split iOS shadow / Android elevation
  },
  text: {
    fontFamily: rnTokens.typography.family.base, // token font, not 'Helvetica Neue'/'Roboto'
  },
});
```

## Platform.select — for genuinely divergent props/behaviour

Reserve `Platform.select` / `Platform.OS` for real per-platform differences (behaviour,
props, offsets) — not styling values that tokens already handle.

```typescript
import { Platform } from 'react-native';

const behavior = Platform.select({ ios: 'padding', android: 'height' } as const);
```

## Platform.OS

```typescript
import { Platform } from 'react-native';

function MyComponent() {
  const isIOS = Platform.OS === 'ios';
  const isAndroid = Platform.OS === 'android';

  return (
    <View>
      {isIOS && <IOSOnlyComponent />}
      <Text>{isAndroid ? 'Android' : 'iOS'}</Text>
    </View>
  );
}
```

## Platform-Specific Files

```
components/
├── Button.tsx           # Shared logic
├── Button.ios.tsx       # iOS-specific
└── Button.android.tsx   # Android-specific
```

```typescript
// Import resolves to correct platform file
import Button from './components/Button';
```

## SafeAreaView

```typescript
import { SafeAreaView, StyleSheet } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

// Method 1: SafeAreaView component
function Screen() {
  return (
    <SafeAreaView style={styles.container}>
      <Content />
    </SafeAreaView>
  );
}

// Method 2: useSafeAreaInsets hook (more control)
function CustomHeader() {
  const insets = useSafeAreaInsets();

  return (
    <View style={[styles.header, { paddingTop: insets.top }]}>
      <Text>Header</Text>
    </View>
  );
}

// Method 3: SafeAreaProvider context
import { SafeAreaProvider } from 'react-native-safe-area-context';

function App() {
  return (
    <SafeAreaProvider>
      <Navigation />
    </SafeAreaProvider>
  );
}
```

## KeyboardAvoidingView

```typescript
import { KeyboardAvoidingView, Platform } from 'react-native';

function FormScreen() {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={{ flex: 1 }}
      keyboardVerticalOffset={Platform.select({ ios: 88, android: 0 })}
    >
      <ScrollView>
        <TextInput placeholder="Name" />
        <TextInput placeholder="Email" />
      </ScrollView>
    </KeyboardAvoidingView>
  );
}
```

## StatusBar

```typescript
import { StatusBar, Platform } from 'react-native';
import { rnTokens } from '@repo/core/tokens/rn-styles';

function Screen() {
  return (
    <>
      <StatusBar
        barStyle={Platform.OS === 'ios' ? 'dark-content' : 'light-content'}
        backgroundColor={Platform.OS === 'android' ? rnTokens.colors.black : undefined}
      />
      <Content />
    </>
  );
}
```

## Android Back Button

```typescript
import { useEffect } from 'react';
import { BackHandler, Platform } from 'react-native';

function useBackHandler(handler: () => boolean) {
  useEffect(() => {
    if (Platform.OS !== 'android') return;

    const subscription = BackHandler.addEventListener(
      'hardwareBackPress',
      handler
    );

    return () => subscription.remove();
  }, [handler]);
}

// Usage
function Screen() {
  useBackHandler(() => {
    if (hasUnsavedChanges) {
      showDiscardAlert();
      return true; // Prevent default back
    }
    return false; // Allow default back
  });
}
```

## Quick Reference

| API | Purpose |
|-----|---------|
| `Platform.OS` | Get platform ('ios' / 'android') |
| `Platform.select()` | Platform-specific values |
| `Platform.Version` | OS version number |
| `.ios.tsx` / `.android.tsx` | Platform-specific files |

| Component | Purpose |
|-----------|---------|
| `SafeAreaView` | Avoid notch/home indicator |
| `KeyboardAvoidingView` | Keyboard handling |
| `StatusBar` | Status bar styling |
| `BackHandler` | Android back button |
