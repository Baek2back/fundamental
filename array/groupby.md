# groupBy

> 배열 내의 요소를 특정 프로퍼티를 기준으로 그룹화한다.

```typescript
function groupBy<T, Key extends PropertyKey>(
  array: readonly T[],
  getGroupId: (item: T) => Key
): Partial<Record<Key, T[]>> {
  return array.reduce((acc, item) => {
    const groupId = getGroupId(item);
    if (!acc[groupId]) acc[groupId] = [];
    acc[groupId].push(item);
    return acc;
  }, {} as Record<Key, T[]>);
}

groupBy([6.1, 4.2, 6.3], Math.floor);
// { '4': [4.2], '6': [6.1, 6.3] }
```
