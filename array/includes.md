# includes

```typescript
const actions = [
  "CREATE",
  "READ",
  "UPDATE",
  "DELETE"
] as const;
// readonly ["CREATE", "READ", "UPDATE", "DELETE"]

const execute = (action: string) => {
  if (actions.includes(action)) {
    // 💥 ERROR
  }
};
```

> Argument of type `'string'` is not assignable to parameter of type `'CREATE' | 'READ' | 'UPDATE' | 'DELETE'` . = Error 2345

```typescript
interface Array<T> {
  includes(searchElement: T, fromIndex?: number): boolean;
}

interface ReadonlyArray<T> {
  includes(searchElement: T, fromIndex?: number): boolean;
}
```

`Array<T>`와 `ReadonlyArray<T>`의 선언을 살펴보면 **찾으려는 요소(`searchElement`)의 타입과 배열이 담고있는 값의 타입이 동일**해야 한다.

이때 `string`은 `T`(`'CREATE' | 'READ' | 'UPDATE' | 'DELETE'`)보다 훨씬 넓은 타입이므로 타입 캐스팅을 이용해 타입을 좁혀주어야 한다.

```typescript
function includes<T extends U, U>(
  array: ReadonlyArray<T>,
  element: U
): element is T {
  return array.includes(element as T);
}
```

최종적으로는 `T` 가 `U` 를 확장(extends)하는 지 검증하며, 이는 **`U`가 `T`의 상위 집합(superset) 혹은 `T`가 `U`의 하위 집합(subset)임을 보장** 할 수 있게 된다.
