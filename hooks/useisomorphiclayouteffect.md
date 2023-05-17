# useIsomorphicLayoutEffect

### `useEffect` vs. `useLayoutEffect`

* `useEffect`는 컴포넌트들이 render, paint 된 이후에 **비동기적(asynchronous)으로 실행**된다.
* `useLayoutEffect`는 컴포넌트들이 render 된 이후에 실행되며, **동기적(synchronous)으로 실행**된 다음 paint를 수행하게 된다.

### Server Side Rendering

> Warning: `useLayoutEffect` does nothing on the server, because its effect cannot be encoded into the server renderer's output format. This will lead to a mismatch between the initial, non-hydrated UI and the intended UI. To avoid this, `useLayoutEffect` should only be used in components that render exclusively on the client.

* SSR 환경에서 `useEffect`, `useLayoutEffect`에 전달한 코드는 자바스크립트가 모두 다운로드 될 때까지 실행되지 않으며, 브라우저에서 paint 단계를 수행한 이후에 `useLayoutEffect` -> `useEffect` 순으로 실행된다.
* SSR 환경에서 `useLayoutEffect`를 직접 사용하면 경고를 노출하는 이유는, `useEffect`는 pre-render 된 HTML이 브라우저 상에서 paint 된 이후에 실행되더라도 기본 동작 방식과 동일하지만, `useLayoutEffect`의 경우에는 paint 이전에 실행되어야 하는 코드이므로 명시적으로 **브라우저에서만 실행되는 코드에만 적용**해야 하기 때문이다.

### `useIsomorphicLayoutEffect`

```typescript
import { useLayoutEffect, useEffect } from 'react';

const canUseDOM = () =>
  Boolean(
    typeof window !== "undefined" &&
    window.document &&
    window.document.createElement
  );
  
const useIsomorphicLayoutEffect = canUseDOM()
  ? useLayoutEffect
  : useEffect;

export { useIsomorphicLayoutEffect as default };
```





