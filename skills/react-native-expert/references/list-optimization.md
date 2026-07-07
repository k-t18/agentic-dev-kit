# List Optimization

> This reference owns **list performance** (windowing, `getItemLayout`, FlashList, batching).
> The row component (`ListItem`) is a `packages/ui-native` component built per `rn-component`
> (every string in `<Text>`, keyed styles), and every `styles.*` value comes from `rnTokens`
> (`@repo/core/tokens/rn-styles`) — import `rnTokens` where these examples use styling.

## Optimized FlatList

```typescript
import { FlatList, ListRenderItem } from 'react-native';
import { memo, useCallback } from 'react';

interface Item {
  id: string;
  title: string;
  subtitle: string;
}

// Memoized list item
const ListItem = memo(function ListItem({
  item,
  onPress
}: {
  item: Item;
  onPress: (id: string) => void;
}) {
  return (
    <Pressable onPress={() => onPress(item.id)} style={styles.item}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.subtitle}>{item.subtitle}</Text>
    </Pressable>
  );
});

function OptimizedList({ data, onSelect }: { data: Item[]; onSelect: (id: string) => void }) {
  // Memoize callbacks (onSelect typically navigates or calls a feature-slice hook)
  const handlePress = useCallback((id: string) => onSelect(id), [onSelect]);

  const renderItem: ListRenderItem<Item> = useCallback(
    ({ item }) => <ListItem item={item} onPress={handlePress} />,
    [handlePress]
  );

  const keyExtractor = useCallback((item: Item) => item.id, []);

  // Fixed height for getItemLayout
  const getItemLayout = useCallback(
    (_: any, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      // Performance props
      removeClippedSubviews
      maxToRenderPerBatch={10}
      windowSize={5}
      initialNumToRender={10}
      updateCellsBatchingPeriod={50}
    />
  );
}

const ITEM_HEIGHT = 72;
```

## SectionList

```typescript
import { SectionList } from 'react-native';

interface Section {
  title: string;
  data: Item[];
}

function GroupedList({ sections }: { sections: Section[] }) {
  const renderSectionHeader = useCallback(
    ({ section }: { section: Section }) => (
      <View style={styles.sectionHeader}>
        <Text style={styles.sectionTitle}>{section.title}</Text>
      </View>
    ),
    []
  );

  return (
    <SectionList
      sections={sections}
      renderItem={renderItem}
      renderSectionHeader={renderSectionHeader}
      keyExtractor={keyExtractor}
      stickySectionHeadersEnabled
    />
  );
}
```

## Pull to Refresh

```typescript
function RefreshableList({ data, onRefresh }: Props) {
  const [refreshing, setRefreshing] = useState(false);

  const handleRefresh = useCallback(async () => {
    setRefreshing(true);
    await onRefresh();
    setRefreshing(false);
  }, [onRefresh]);

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      refreshControl={
        <RefreshControl
          refreshing={refreshing}
          onRefresh={handleRefresh}
          tintColor={rnTokens.colors.primary}
        />
      }
    />
  );
}
```

## Infinite Scroll

Pagination is **server state** — don't hand-roll `useState` + `fetch`. Use a React Query
`useInfiniteQuery` hook from `@repo/core` (`feature-slice`); this screen only wires the list
to it. The same hook is shared with the web twin.

```typescript
import { useItemsInfinite } from '@repo/core/features/items'; // feature-slice useInfiniteQuery hook

function InfiniteList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useItemsInfinite();
  const items = data?.pages.flatMap((p) => p.items) ?? [];

  const loadMore = useCallback(() => {
    if (hasNextPage && !isFetchingNextPage) fetchNextPage();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  const renderFooter = useCallback(
    () => (isFetchingNextPage ? <ActivityIndicator style={styles.loader} /> : null),
    [isFetchingNextPage]
  );

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={renderFooter}
    />
  );
}
```

## FlashList (Alternative)

```typescript
import { FlashList } from '@shopify/flash-list';

function FastList({ data }: { data: Item[] }) {
  return (
    <FlashList
      data={data}
      renderItem={renderItem}
      estimatedItemSize={72}
      keyExtractor={keyExtractor}
    />
  );
}
```

## Quick Reference

| Prop | Purpose |
|------|---------|
| `removeClippedSubviews` | Unmount off-screen items |
| `maxToRenderPerBatch` | Items per render batch |
| `windowSize` | Render window multiplier |
| `initialNumToRender` | Initial items to render |
| `getItemLayout` | Skip measurement (fixed height) |

| Optimization | When |
|--------------|------|
| `memo()` | All list items |
| `useCallback` | renderItem, keyExtractor |
| `getItemLayout` | Fixed height items |
| `FlashList` | Very large lists |
