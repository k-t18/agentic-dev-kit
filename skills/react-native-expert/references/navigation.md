# Navigation (React Navigation)

Bare RN CLI uses **React Navigation** (not Expo Router — there is no file-based `app/`
directory). Navigators are declared in code and typed with param lists. Screens render
`@repo/ui-native` components and read data via `@repo/core` hooks (`feature-slice`).

## Typed param lists

Type every navigator so `navigate`/params are checked.

```typescript
// navigation/types.ts
export type RootStackParamList = {
  Tabs: undefined;
  Auth: undefined;
  Details: { id: string; title?: string }; // was details/[id]
};

export type TabParamList = {
  Home: undefined;
  Profile: undefined;
};

// Makes navigation.navigate() globally type-safe
declare global {
  namespace ReactNavigation {
    interface RootParamList extends RootStackParamList {}
  }
}
```

## Root navigator

```typescript
// navigation/RootNavigator.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import type { RootStackParamList } from './types';
import { navigationTheme } from './theme'; // from token theme-modes, not RN Dark/DefaultTheme

const Stack = createNativeStackNavigator<RootStackParamList>();

export function RootNavigator() {
  return (
    <NavigationContainer theme={navigationTheme} linking={linking}>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        <Stack.Screen name="Tabs" component={TabNavigator} />
        <Stack.Screen name="Auth" component={AuthNavigator} />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={{ presentation: 'modal' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

> Theme comes from the design-system token **theme modes** (`design-system-setup`) mapped to a
> React Navigation theme object — do not import RN's `DarkTheme`/`DefaultTheme` directly.

## Tab navigator

```typescript
// navigation/TabNavigator.tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import Icon from 'react-native-vector-icons/Ionicons'; // bare CLI icon lib (not @expo/vector-icons)
import { rnTokens } from '@repo/core/tokens/rn-styles';
import type { TabParamList } from './types';

const Tab = createBottomTabNavigator<TabParamList>();

export function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{ tabBarActiveTintColor: rnTokens.colors.primary, headerShown: true }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => <Icon name="home" color={color} size={size} />,
        }}
      />
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size }) => <Icon name="person" color={color} size={size} />,
        }}
      />
    </Tab.Navigator>
  );
}
```

## Programmatic navigation + reading params

```typescript
import { useNavigation, useRoute, type RouteProp } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import type { RootStackParamList } from './types';

// Navigate
function useAppNav() {
  const navigation = useNavigation<NativeStackNavigationProp<RootStackParamList>>();
  navigation.navigate('Details', { id: '123', title: 'Item' }); // push to stack
  navigation.replace('Tabs');   // replace current
  navigation.goBack();          // go back
  navigation.canGoBack();       // check if can go back
  navigation.popToTop();        // dismiss the whole stack
}

// Reading params
function DetailsScreen() {
  const { params } = useRoute<RouteProp<RootStackParamList, 'Details'>>();
  return <Text>Details for {params.id}</Text>;
}
```

## Protected routes

Choose the navigator at the top based on the session from `@repo/core` — the equivalent of
Expo Router's group redirects.

```typescript
// navigation/RootNavigator.tsx
import { useSession } from '@repo/core'; // auth/session hook (feature-slice)

export function RootNavigator() {
  const { user, isLoading } = useSession();
  if (isLoading) return <LoadingScreen />;

  return (
    <NavigationContainer theme={navigationTheme} linking={linking}>
      {user ? (
        <Stack.Navigator screenOptions={{ headerShown: false }}>
          <Stack.Screen name="Tabs" component={TabNavigator} />
          <Stack.Screen name="Details" component={DetailsScreen} />
        </Stack.Navigator>
      ) : (
        <AuthNavigator />
      )}
    </NavigationContainer>
  );
}
```

## Deep linking

```typescript
// navigation/linking.ts
import type { LinkingOptions } from '@react-navigation/native';
import type { RootStackParamList } from './types';

export const linking: LinkingOptions<RootStackParamList> = {
  prefixes: ['myapp://', 'https://app.example.com'],
  config: {
    screens: {
      Tabs: { screens: { Home: 'home', Profile: 'profile' } },
      Details: 'details/:id', // handles myapp://details/123
    },
  },
};
```

> Register the `myapp` URL scheme natively (iOS `Info.plist` `CFBundleURLTypes`, Android
> manifest `intent-filter`) — bare CLI has no `app.json` `scheme`.

## Android hardware back

React Navigation handles back automatically; override only for guard logic (see
`platform-handling.md` → `BackHandler`).

## Quick Reference

| Navigator | Factory |
|-----------|---------|
| Stack | `createNativeStackNavigator()` |
| Tabs | `createBottomTabNavigator()` |
| Drawer | `createDrawerNavigator()` |

| navigation method | Behavior |
|-------------------|----------|
| `navigate(name, params)` | Go to a screen (reuse if mounted) |
| `push(name, params)` | Always add a new stack entry |
| `replace(name)` | Replace current screen |
| `goBack()` | Go back |
| `popToTop()` | Dismiss the stack to its first screen |
