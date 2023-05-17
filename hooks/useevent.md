# useEvent

* `useCallback`의 한계는 의존성 배열(deps)만을 이용해 제어하므로 인수로 전달한 함수가 (1) **다시 생성되어야 한다는 의도**와 함수 내부에서 (2) **어떤 값을 참조하겠다는 의도**를 분리하기 어렵다는 점이다.
* 일반적으로 `useCallback`을 적용하고자 하는 함수는 해당 컴포넌트의 `state`나 부모로부터 전달받은 `props`을 이용해 특정 작업을 수행한다.
* 이때 자식 컴포넌트에 전달할 함수 객체의 참조 값이 변화하여 리렌더링이 발생하는 것을 막기 위해 빈 의존성 배열을 이용해 `useCallback`을 적용하면 함수 내부에서 참조하는 값이 선언 시점으로 고정되어 버리는 문제가 발생한다.

### `useEvent` (`useStableCallback`, `useCommittedCallback`)

* [RFC](https://github.com/reactjs/rfcs/blob/useevent/text/0000-useevent.md)에 따르면 `useEvent`는 `useCallback`에서 겪는 두 가지 이슈를 해결하고자 등장하였다.
  * Stable Identity Callbacks
  * Fresh Values in Callbacks

```typescript
function useLatestValue<T>(value: T) {
  const cache = useRef(value);

  // 실제 구현체에서는 Layout Effect보다 이른 시점에 호출될 것이다.
  useIsomorphicLayoutEffect(() => {
    cache.current = value;
  }, [value]);

  return cache;
}

function useEvent<
  F extends (...args: any[]) => any,
  P extends any[] = Parameters<F>,
  R = ReturnType<F>
>(cb: (...args: P) => R) {
  const cache = useLatestValue(cb);

  return useCallback(
    (...args: P) => cache.current(...args),
    [cache]
  );
}
```

* 먼저 함수 객체의 참조를 항상 동일하게 유지하기 위해 `ref`를 사용하고, `ref`가 담고 있는 값(함수 객체)을 계속해서 갱신하는 방식이다.
* 다만 함수 객체의 생성 자체를 막을 방법은 제공되지 않으며, `useEvent`에 전달한 함수는 렌더링 단계에서는 호출되어서는 안된다는 [제약](https://github.com/reactjs/rfcs/pull/220#issuecomment-1118055107)이 존재한다.
  * 리액트에서는 기본적으로 입력(`props`, `state`, `context`)이 동일하다면 동일한 결과를 생성해야 하므로 `useEvent`에 전달한 함수(`ref`에 담겨 있는 함수)의 상태에 따라 렌더링 결과가 달라질 수 있다는 것은 **비결정적(non-deterministic)이기 때문**이다.
  * 따라서 렌더링 단계에서 호출되는 함수에는 `useEvent` 대신 `useCallback`을 사용해야 한다.
* 이벤트는 특정 순간에 객관적으로 발생한 일로 생각하는 것이 명확하며, 일반적으로 함수 이름이 `on` 또는 `handle`이라는 접미사와 함께 정의되어 있다면 이벤트라고 여기는 것이 좋다.

### 참고 자료

* [headlessui/use-event.ts at main · tailwindlabs/headlessui](https://github.com/tailwindlabs/headlessui/blob/main/packages/@headlessui-react/src/hooks/use-event.ts)
* [reach-ui/use-stable-callback.ts at dev · reach/reach-ui](https://github.com/reach/reach-ui/blob/dev/packages/utils/src/use-stable-callback.ts)
* [rfcs/0000-useevent.md at useevent · reactjs/rfcs](https://github.com/reactjs/rfcs/blob/useevent/text/0000-useevent.md)

