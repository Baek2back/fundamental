---
description: 콜백으로 전달받은 함수를 한 번만 호출되도록하는 함수를 생성해 줍니다.
---

# once

```typescript
function once<A extends any[], R, T>(
  fn: (this: T, ...args: A) => R
): (this: T, ...args: A) => R | undefined {
  let done = false;
  return function (this: T, ...args: A) {
    return done ? undefined : ((done = true), fn.apply(this, args));
  };
}
```

## Example

```ts
const foo = () => console.log("hi");

const onceLog = once(foo);

onceLog(); // "hi"
onceLog(); // noting on console
onceLog(); // noting on console
```
