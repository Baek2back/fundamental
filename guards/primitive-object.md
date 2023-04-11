# Primitive, object

```typescript
type Primitive =
  | string
  | number
  | bigint
  | boolean
  | symbol
  | null
  | undefined;
```

```typescript
type Various = number | string | object;

// Expect: object
type Cleaned = Exclude<Various, Primitive>;

const isPrimitive = (
  value: unknown
): value is Primitive => {
  if (value === null || value === undefined) {
    return true;
  }

  switch (typeof value) {
    case "string":
    case "number":
    case "bigint":
    case "boolean":
    case "symbol": {
      return true;
    }

    default: {
      return false;
    }
  }
};
```

### `{}` vs. `object`

* `{}` 타입: `Nullish` 타입을 제외한 모든 값을 포함하는 타입
* `object` 타입: 원시 타입의 값이 아닌 **모든 객체 타입의 값을 포함**하는 타입

```typescript
function isObject(value: unknown): value is object {
  return !isPrimitive(value);
}
```

### 참고 자료

* [https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts](https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts)
