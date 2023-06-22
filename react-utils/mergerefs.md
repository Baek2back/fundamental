# mergeRefs

* UI 컴포넌트를 구현할 때 `React.forwardRef`를 통해 전달받은 외부의 `ref`와 컴포넌트 자체가 보유한 `ref`(local ref) 모두 지원하게끔 하는 것이 일반적이다.
* 하지만 기본적으로 리액트에서는 하나의 `ref` 속성에 여러 개의 `ref`를 설정하는 방법을 제공하지 않으므로, 유틸리티 함수를 정의해 사용한다.

```typescript
import type { MutableRefObject, LegacyRef, RefCallback } from "react";

/**
 * @example
 * const Component = forwardRef((props, ref) => {
 *   const ownRef = useRef();
 *   const domRef = mergeRefs([ref, ownRef]); 
 *   return <div ref={domRef}>...</div>;
 * });
 */
 
function mergeRefs<T = any>(
  refs: (MutableRefObject<T> | LegacyRef<T>)[]
): RefCallback<T> {
  return (value) => {
    refs.forEach((ref) => {
      if (typeof ref === "function") {
        ref(value);
      } else if (ref) {
        (ref as MutableRefObject<T | null>).current = value;
      }
    });
  };
}
```

### 참고 자료

* [Merging refs](https://dev.to/thekashey/merging-refs-41l8)
* [https://github.com/theKashey/use-callback-ref](https://github.com/theKashey/use-callback-ref)
* [react-merge-refs/index.tsx at main · gregberge/react-merge-refs](https://github.com/gregberge/react-merge-refs/blob/main/src/index.tsx)
* [nextui/packages/react/src/utils/refs.ts at main · nextui-org/nextui](https://github.com/nextui-org/nextui/blob/main/packages/react/src/utils/refs.ts)
