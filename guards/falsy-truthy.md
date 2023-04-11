# Falsy, Truthy

```typescript
type Falsy =
  | false
  | ""
  | 0
  | -0
  | 0n
  | -0n
  | null
  | undefined
  | void;

function isFalsy(value: unknown): value is Falsy {
  return !value;
}

type Truthy<T> = Exclude<T, Falsy>;

function isTruthy<T>(value: T): value is Truthy<T> {
  return !!value;
}
```

### 참고 자료

* [https://www.lucaspaganini.com/academy/custom-type-guards-typescript-narrowing-3](https://www.lucaspaganini.com/academy/custom-type-guards-typescript-narrowing-3)
* [https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts](https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts)
