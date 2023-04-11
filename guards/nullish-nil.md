---
description: 널 병합 연산자(Nullish Coalescing)에서 차용한 타입
---

# Nullish(Nil)

```typescript
type Nullish = null | undefined;

type Nullable<T> = T | Nullish;

function isUndefined(value: unknown): value is undefined {
  return value === undefined;
}

function isNotUndefined<T>(
  value: T
): value is Exclude<T, undefined> {
  return value !== undefined;
}

function isNull(value: unknown): value is null {
  return value === null;
}

function isNotNull<T>(value: T): value is Exclude<T, null> {
  return value !== null;
}

function isNullish(value: unknown): value is Nullish {
  return isNull(value) || isUndefined(value);
}

function isNotNullish<T>(
  value: T
): value is NonNullable<T> {
  return !isNullish(value);
}
```

### 참고 자료

* [https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts](https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts)
* [https://github.com/lazarljubenovic/type-guards/blob/master/src/index.ts](https://github.com/lazarljubenovic/type-guards/blob/master/src/index.ts)
