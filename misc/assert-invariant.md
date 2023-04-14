# assert (invariant)

> 개발 환경에서는 설명과 함께 오류를 제공하고, 프로덕션 환경에서는 설명을 제외한 오류만 제공하도록 하는 것이 목적이다.

```typescript
const isProduction = process.env.NODE_ENV === "production";
const prefix = "Invariant failed";

export default function invariant(
  condition: any,
  message?: string | (() => string)
): asserts condition {
  if (condition) {
    return;
  }

  if (isProduction) {
    throw new Error(prefix);
  }

  const provided: string | undefined =
    typeof message === "function" ? message() : message;

  const value: string = provided
    ? `${prefix}: ${provided}`
    : prefix;

  throw new Error(value);
}
```

### proposal-throw-expressions (stage2)

```typescript
cond || throw x
cond ?? throw x
```

### 참고 자료

* [https://stackoverflow.com/questions/39055159/using-facebooks-invariant-vs-if-throw](https://stackoverflow.com/questions/39055159/using-facebooks-invariant-vs-if-throw)
* [https://github.com/alexreardon/tiny-invariant/blob/master/src/tiny-invariant.ts](https://github.com/alexreardon/tiny-invariant/blob/master/src/tiny-invariant.ts)
* [https://github.com/tc39/proposal-throw-expressions](https://github.com/tc39/proposal-throw-expressions)
