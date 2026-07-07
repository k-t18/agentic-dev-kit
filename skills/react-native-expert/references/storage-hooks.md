# Storage & Hooks

> **Where storage lives:**
> - **Durable / synced app data** (anything the app treats as the source of truth, offline
>   writes) → `@repo/core` / `@repo/offline-kit` (`OfflineDb`). **Not** a component storage hook.
> - **View-local prefs only** (theme toggle, last-used filter) → a native storage hook below.
>   Because these use native modules (MMKV/AsyncStorage), the hook lives in `packages/ui-native`,
>   **never in `@repo/core`** (core is platform-agnostic — same rule as `useLocalStorage` in
>   `react-renderer`).
> - **Global store slices** → `zustand-slice` (`@repo/core/state`); **session/auth tokens** →
>   `@repo/core` auth (SecureStore is wired there, not ad-hoc in a screen).

## AsyncStorage

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Basic operations
await AsyncStorage.setItem('user', JSON.stringify(user));
const user = JSON.parse(await AsyncStorage.getItem('user') || 'null');
await AsyncStorage.removeItem('user');
await AsyncStorage.clear();

// Multiple items
await AsyncStorage.multiSet([
  ['user', JSON.stringify(user)],
  ['settings', JSON.stringify(settings)],
]);

const values = await AsyncStorage.multiGet(['user', 'settings']);
```

## useStorage Hook

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useState, useEffect, useCallback } from 'react';

function useStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(initialValue);
  const [loading, setLoading] = useState(true);

  // Load on mount
  useEffect(() => {
    AsyncStorage.getItem(key)
      .then((item) => {
        if (item !== null) {
          setValue(JSON.parse(item));
        }
      })
      .finally(() => setLoading(false));
  }, [key]);

  // Persist changes
  const setStoredValue = useCallback(
    async (newValue: T | ((prev: T) => T)) => {
      const valueToStore =
        newValue instanceof Function ? newValue(value) : newValue;
      setValue(valueToStore);
      await AsyncStorage.setItem(key, JSON.stringify(valueToStore));
    },
    [key, value]
  );

  const removeValue = useCallback(async () => {
    setValue(initialValue);
    await AsyncStorage.removeItem(key);
  }, [key, initialValue]);

  return { value, setValue: setStoredValue, removeValue, loading };
}

// Usage
function Settings() {
  const { value: theme, setValue: setTheme, loading } = useStorage('theme', 'light');

  if (loading) return <Loading />;

  return (
    <Switch
      value={theme === 'dark'}
      onValueChange={(dark) => setTheme(dark ? 'dark' : 'light')}
    />
  );
}
```

## MMKV (Faster Alternative)

```typescript
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

// Synchronous operations
storage.set('user.name', 'John');
const name = storage.getString('user.name');

storage.set('user.age', 25);
const age = storage.getNumber('user.age');

storage.set('user.premium', true);
const isPremium = storage.getBoolean('user.premium');

storage.delete('user.name');
storage.clearAll();

// JSON data
storage.set('user', JSON.stringify(user));
const user = JSON.parse(storage.getString('user') || '{}');
```

## useMMKV Hook

```typescript
import { useMMKVString, useMMKVNumber, useMMKVBoolean } from 'react-native-mmkv';

function Settings() {
  const [theme, setTheme] = useMMKVString('theme');
  const [fontSize, setFontSize] = useMMKVNumber('fontSize');
  const [notifications, setNotifications] = useMMKVBoolean('notifications');

  return (
    <>
      <Switch
        value={theme === 'dark'}
        onValueChange={(dark) => setTheme(dark ? 'dark' : 'light')}
      />
      <Slider value={fontSize} onValueChange={setFontSize} />
      <Switch value={notifications} onValueChange={setNotifications} />
    </>
  );
}
```

## Zustand with MMKV → defer to `zustand-slice`

A persisted global store is a **`zustand-slice`** concern in `@repo/core/state`, not a bespoke
store defined in a screen. The MMKV persist adapter below is the *mechanism* `zustand-slice`
wires for the native target — follow that skill for the canonical store pattern (selectors,
persist, `@repo/core/state`); don't hand-roll a store here.

```typescript
// Mechanism only — the store itself is owned by zustand-slice (@repo/core/state)
import { createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

export const mmkvPersistStorage = createJSONStorage(() => ({
  getItem: (name: string) => storage.getString(name) ?? null,
  setItem: (name: string, value: string) => storage.set(name, value),
  removeItem: (name: string) => storage.delete(name),
}));
```

## Quick Reference

| Storage | Speed | Async | Use Case |
|---------|-------|-------|----------|
| AsyncStorage | Slow | Yes | Small data, simple apps |
| MMKV | Fast | No | Large data, frequent access |
| SecureStore | Medium | Yes | Sensitive data (tokens) |

| Hook | Returns |
|------|---------|
| `useStorage()` | { value, setValue, loading } |
| `useMMKVString()` | [value, setValue] |
| `useMMKVNumber()` | [value, setValue] |
| `useMMKVBoolean()` | [value, setValue] |
