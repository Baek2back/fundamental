# countBy

### Array.prototype.reduce()

```typescript
const colors = ['red', 'blue', 'green'];

colors.reduce((acc, color) => {
  acc[color] = (acc[color] ?? 0) + 1;
  return acc;
}, {} as Record<string, number>);

// { red: 1, blue: 1, green: 1 }
```

### Generic Version

객체에 대한 인덱스로 사용할 수 있는 값의 타입은 `(string | number | symbol)`이며 `keyof any`와 동일하다.

타입스크립트의 빌트인 타입인 `PropertyKey` 역시 `(string | number | symbol)` 타입이다.

```typescript
type PropertyKey 
  = (string | number | symbol)
  = keyof any

const countBy = <T, TId extends PropertyKey>(
  list: readonly T[],
  identity: (item: T) => TId
): Record<TId, number> => {
  if (!list) return {} as Record<TId, number>;
  
  return list.reduce((acc, item) => {
    const id = identity(item);
    acc[id] = (acc[id] ?? 0) + 1;
    return acc;
  }, {} as Record<TId, number>);
};

countBy(colors, (color) => color);

// { red: 1, blue: 1, green: 1 }
```
