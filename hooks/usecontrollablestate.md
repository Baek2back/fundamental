# useControllableState

* **`value` (O) / `onChangeHandler` (O)**
  * `Controlled` -> 컴포넌트의 상태는 props를 이용해 전달받음.
* **`value` (O) / `onChangeHandler` (X)**
  * `Controlled` -> 컴포넌트의 상태가 강제되며, 값을 변경할 수 있는 방법이 제공되지 않음.
* **`value` (X) / `onChangeHandler` (O)**
  * `Uncontrolled` -> 컴포넌트 자체의 지역 상태로 유지되며, 상위 컴포넌트에서는 `onChangeHandler`를 이용해 값이 변경되었다는 사실만 인지할 수 있음.
* **`value` (X) / `onChangeHandler` (X)**
  * `Uncontrolled` -> 컴포넌트의 상태 변경에 대해 상위 컴포넌트에서 고려하지 않아도 되는 경우

````typescript
interface UseControllableStateProps<T> {
  value?: T;
  defaultValue?: T | (() => T);
  onChange?: (value: T) => void;
  shouldUpdate?: (prev: T, next: T) => boolean;
}

function useControllableState<T>(
  props: UseControllableStateProps<T>
) {
  const {
    value: valueProp,
    defaultValue,
    onChange,
    shouldUpdate = Object.is,
  } = props;

  const onChangeProp = useCallbackRef(onChange);
  const shouldUpdateProp = useCallbackRef(shouldUpdate);

  const [uncontrolledState, setUncontrolledState] = useState(defaultValue as T);
  const controlled = valueProp !== undefined;

  const value = controlled ? valueProp : uncontrolledState;
  const setValue = useCallbackRef(
    (next: React.SetStateAction<T>) => {
      const setter = next as (prevState?: T) => T;
      const nextValue =
        typeof next === "function" ? setter(value) : next;

      if (!shouldUpdateProp(value, nextValue)) {
        return;
      }

      if (!controlled) {
        setUncontrolledState(nextValue);
      }

      onChangeProp(nextValue);
    },
    [controlled, onChangeProp, value, shouldUpdateProp]
  );

  return [value, setValue] as [
    T,
    React.Dispatch<React.SetStateAction<T>>
  ];
}

```
````

### 참고 자료

[chakra-ui/index.ts at main · chakra-ui/chakra-ui](https://github.com/chakra-ui/chakra-ui/blob/main/packages/hooks/use-controllable-state/src/index.ts)
