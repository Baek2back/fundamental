# Primitive

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

### 참고 자료

* [https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts](https://github.com/piotrwitek/utility-types/blob/master/src/aliases-and-guards.ts)
