# Project Structure

The native app is a **bare React Native CLI** app inside the Turborepo monorepo. It is a
*consumer*: UI comes from `@repo/ui-native`, and hooks/API/stores/types/tokens from
`@repo/core`. It does **not** define its own component library, API client, stores, or color
constants — those live in packages.

## Monorepo layout (native app)

```
apps/
└── native/                     # bare RN CLI app (consumer)
    ├── src/
    │   ├── navigation/         # React Navigation: RootNavigator, TabNavigator, types, linking
    │   ├── screens/            # screen components (compose @repo/ui-native + @repo/core hooks)
    │   └── App.tsx
    ├── android/                # native project (Gradle)
    ├── ios/                    # native project (CocoaPods)
    ├── index.js                # AppRegistry entry
    ├── metro.config.js         # MUST keep monorepo watchFolders
    ├── babel.config.js
    ├── tsconfig.json
    └── package.json

packages/
├── ui-native/                  # native components (rn-component) — no local components/ui here
└── core/                       # hooks, API (apiClient), stores, types, tokens/rn-styles
```

> No local `components/ui`, `services/api.ts`, `stores/`, or `constants/colors.ts` in the app —
> components → `@repo/ui-native`, data/API/stores/types → `@repo/core`, colors → tokens.

## metro.config.js (monorepo — watchFolders required)

```javascript
const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');
const path = require('path');

const projectRoot = __dirname;
const workspaceRoot = path.resolve(projectRoot, '../..');

/** @type {import('@react-native/metro-config').MetroConfig} */
const config = {
  // Watch the whole monorepo so Metro resolves @repo/* packages (design-system-setup pitfall)
  watchFolders: [workspaceRoot],
  resolver: {
    nodeModulesPaths: [
      path.resolve(projectRoot, 'node_modules'),
      path.resolve(workspaceRoot, 'node_modules'),
    ],
  },
};

module.exports = mergeConfig(getDefaultConfig(projectRoot), config);
```

> Removing `watchFolders` silently breaks Metro's resolution of `@repo/*` — never delete it.

## babel.config.js

```javascript
module.exports = {
  presets: ['module:@react-native/babel-preset'], // NOT babel-preset-expo
  plugins: [
    'react-native-reanimated/plugin', // must be last
  ],
};
```

## tsconfig.json

```json
{
  "extends": "@react-native/typescript-config",
  "compilerOptions": {
    "strict": true,
    "baseUrl": "."
  }
}
```

> `@repo/*` package resolution is handled by the workspace (pnpm) + Metro, not per-app `@/*`
> path aliases.

## Essential dependencies (no Expo)

```json
{
  "dependencies": {
    "react-native": "0.73.x",
    "@react-navigation/native": "^7.0.0",
    "@react-navigation/native-stack": "^7.0.0",
    "@react-navigation/bottom-tabs": "^7.0.0",
    "react-native-safe-area-context": "^4.8.0",
    "react-native-screens": "^3.29.0",
    "react-native-reanimated": "^3.6.0",
    "react-native-gesture-handler": "^2.14.0",
    "react-native-vector-icons": "^10.0.0",
    "react-native-mmkv": "^2.11.0",
    "@repo/ui-native": "workspace:*",
    "@repo/core": "workspace:*"
  }
}
```

> Server data (`@tanstack/react-query`) and global state (`zustand`) are dependencies of
> `@repo/core`, not the app — the app consumes the hooks/stores, it doesn't wire them.

## Quick Reference

| Location | Purpose |
|----------|---------|
| `apps/native/src/navigation/` | React Navigation navigators + types |
| `apps/native/src/screens/` | Screens (compose ui-native + core hooks) |
| `@repo/ui-native` | Native components (`rn-component`) |
| `@repo/core` | Hooks, API, stores, types, tokens |
| `metro.config.js` | Monorepo Metro (keep `watchFolders`) |
