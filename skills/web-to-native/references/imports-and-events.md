# Imports, events, navigation, a11y

## Imports migration

```tsx
// ── REMOVE (web-only) ──────────────────────────────
import { cn } from '../../lib/utils';
import { cva, type VariantProps } from 'class-variance-authority';
import { Loader2, Search } from 'lucide-react';
import { useNavigate, useParams, Link } from 'react-router-dom';

// ── KEEP UNCHANGED (the whole point — @repo/core is shared) ──
import { useGetUser } from '@repo/core/hooks';
import { useAuthStore } from '@repo/core/store';
import { rnTokens } from '@repo/core/tokens/rn-styles';
import type { User } from '@repo/core/types';

// ── ADD (native) ───────────────────────────────────
import { View, Text, TouchableOpacity, TextInput, Image, ScrollView, FlatList, ActivityIndicator, StyleSheet } from 'react-native';
import { useNavigation, useRoute } from '@react-navigation/native'; // only if the component navigates
```

Note: icons are `ReactNode` slots — no icon library is imported by the component. The
web loading icon (`Loader2`) becomes `ActivityIndicator`. `SafeAreaView` and navigation
providers are app-level (`apps/native`), not part of a `ui-native` component.

## Events

```ts
onClick={onPress}   → onPress={onPress}
onChange (input)    → onChangeText
onFocus / onBlur    → onFocus / onBlur   (same names)
// hover / keyboard events → not applicable on touch
```

## Navigation (for links only)

```ts
useNavigate()          → useNavigation()
navigate('/profile')   → navigation.navigate('Profile')
navigate(-1)           → navigation.goBack()
useParams()            → useRoute().params
<Link to="/x">         → <TouchableOpacity onPress={() => navigation.navigate('X')}>
```

## Accessibility (ARIA → native)

```ts
aria-label="x"       → accessibilityLabel="x"
aria-hidden="true"   → importantForAccessibility="no-hide-descendants"
aria-busy={loading}  → accessibilityState={{ busy: loading }}
aria-disabled={x}    → accessibilityState={{ disabled: x }}
role="button"        → accessibilityRole="button"
```

Full native a11y guidance: `rn-component/references/rn-accessibility.md`.
