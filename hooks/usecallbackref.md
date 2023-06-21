# useCallbackRef

```tsx
type Ref<T> =
  | RefCallback<T>
  | RefObject<T>
  | null;
```

* `useRef`가 반환한 `ref`는 내용이 변경(`ref.current = value`)되더라도 **리렌더링을 발생시키지 않는 특징**을 갖는다.
  * JSX Element(ex. `<div>`)에 `ref`를 전달하면 컴포넌트가 렌더링된 이후 해당 DOM 노드에 대한 참조를 `ref.current`에 할당해주는 역할을 수행한다.
  * 만약 해당 요소가 unmount된다면 `ref.current`에 할당된 값을 `null`로 변경해주는 동작을 수행한다.

```tsx
<input
  ref={(node) => { 
    ref.current = node; 
  }}
/>
```

* 즉, `Ref`의 타입에서도 알 수 있듯이 기본적으로는 `RefObject`를 전달한 경우 위와 같은 **문법적 설탕(Syntatic Sugar)**을 제공한다고 생각할 수 있다.
* 이때 리액트는 동일한 참조를 갖는 `ref`를 전달한 경우 `ref`에 전달한 함수를 다시 실행하지 않게된다.

```tsx
const ref = useCallback((node) => {
  node.focus();
}, []);

<input ref={ref} />
```

* 따라서 `useCallback`에 빈 의존성 배열(deps)을 전달해 만든 `RefCallback`을 전달하게 된다면 **DOM 노드가 렌더링된 이후에 최초 한 번만 실행되게끔 구현하는 것이 가능**하다.
* 일반적이지는 않지만 `ref.current`가 변경될 때마다 특정한 동작을 수행하게끔 하고 싶다면 `useCallbackRef`를 사용하면 된다.
  * 이전 값과 새로운 값이 동일하다면 전달한 `callback`은 실행되지 않는다.
  * `useRef` 대신 `useState`를 이용해 생성한 상태(ref)를 활용하므로 의존성 배열에 전달하면 변경될 때마다 반응하게끔 할 수 있게 된다.

```tsx
import { useCallbackRef } from "use-callback-ref";
import { useRef, useState } from "react";

const Component = () => {
  const [, forceUpdate] = useState();
  // 1. ref의 변경에 반응할 필요가 없는 경우
  const ref = useRef(null);
 
  useEffect(() => {
    // ref가 변경되더라도 실행되지 않음.
  }, [ref.current]);

  // 2. ref의 변경에 따라 반응해야 하는 경우
  const callbackRef = useCallbackRef(null, () => forceUpdate());

  useEffect(() => {
    // callbackRef가 변경되면 실행됨.
  }, [callbackRef.current]);
};
```

### Implementation

```tsx
import type { MutableRefObject } from "react";
import { useState } from "react";

/**
 * creates a MutableRef with ref change callback
 *
 * @example
 * const ref = useCallbackRef(0, (newValue, oldValue) => console.log(`${oldValue} -> ${newValue}`));
 * ref.current = 1;
 * // 0 -> 1
 */
function useCallbackRef<T>(
  initialValue: T | null,
  callback: (newValue: T | null, lastValue: T | null) => void
): MutableRefObject<T | null> {
  const [ref] = useState(() => ({
    // value
    value: initialValue,
    // last callback
    callback,
    facade: {
      get current() {
        return ref.value;
      },
      set current(value) {
        const last = ref.value;

        if (!Object.is(last, value)) {
          ref.value = value;
          ref.callback(value, last);
        }
      }
    }
  }));
  // update callback
  ref.callback = callback;

  return ref.facade;
}
```

### 참고 자료

* [The same useRef, but it will callback 🤙](https://dev.to/thekashey/the-same-useref-but-it-will-callback-8bo)
* [https://github.com/theKashey/use-callback-ref](https://github.com/theKashey/use-callback-ref)
