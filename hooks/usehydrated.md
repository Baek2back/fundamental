# useHydrated

```typescript
import { useEffect, useState } from 'react';
import type { ReactNode, PropsWithChildren } from "react";

let hydrating = true;

function useHydrated() {
  const [hydrated, setHydrated] = useState(() => !hydrating);
  
  useEffect(() => {
    hydrating = false;
    setHydrated(true);
  }, []);
  
  return hydrated;
}

function ClientOnly({
  children,
  fallback = null,
}: PropsWithChildren<{ fallback?: ReactNode }>) {
  return useHydrated() ? <>{children}</> : <>{fallback}</>;
}
```

* useEffect는 hydration이 끝나기 전에는 호출되지 않는다는 것이 보장되는 것을 이용해, hydration이 종료된 이후의 동작을 제어해야 하는 경우 사용할 수 있다.
  * Server-Side Rendering -> hydrated는 항상 false를 반환한다.
  * Client-Side Rendering -> hydrated는 첫번째 렌더링 시에는 항상 false를 반환하고, 그 이후에는 항상 true를 반환한다.
* hydration 이후의 동작을 제어하는 대신 컴포넌트의 노출 여부를 결정하려면 \<ClientOnly /> 형태로 추상화된 컴포넌트의 사용을 고려할 수도 있다.

```typescript
import type { ComponentProps } from 'react';

const Button = (props: ComponentProps<'button'>) => {
  const hydrated = useHydrated();
  
  return (
    <button type="button" disabled={!hydrated} {...props}>
      Click
    </button>
  );
};

const Page = () => {
  return (
    <ClientOnly fallback={<FakeChart />}>
      <Chart />
    </ClientOnly>
    <Button onClick={handleRefreshChart} />
  )
};
```

### 참고 자료

* [remix-utils/use-hydrated.ts at main · sergiodxa/remix-utils](https://github.com/sergiodxa/remix-utils/blob/main/src/react/use-hydrated.ts)
* [remix-utils/client-only.tsx at main · sergiodxa/remix-utils](https://github.com/sergiodxa/remix-utils/blob/main/src/react/client-only.tsx)
