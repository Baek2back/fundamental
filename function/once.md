---
description: 최초 한번의 함수 호출만 허용
---

# once

```typescript
function once<A extends any[], R, T>(
  fn: (this: T, ...args: A) => R
): (this: T, ...args: A) => R | undefined {
  let done = false;
  return function (this: T, ...args: A) {
    return done
      ? undefined
      : ((done = true), fn.apply(this, args));
  };
}
```
