# useList

### resolveHookState

> S | () => S | ((prevState: S) => S 형태로 다양하게 전달받을 수 있는 State Initializer, SetStateAction에 대응하기 위한 유틸리티 함수

```typescript
// hookState.ts
export type HookStateInitialSetter<S> = () => S;
export type HookStateInitAction<S> = S | HookStateInitialSetter<S>;

export type HookStateSetter<S> = ((prevState: S) => S) | (() => S);
export type HookStateSetAction<S> = S | HookStateSetter<S>;

export type HookStateResolvable<S> =
  | S
  | HookStateInitialSetter<S>
  | HookStateSetter<S>;

export function resolveHookState<S>(nextState: HookStateInitAction<S>): S;
export function resolveHookState<S, C extends S>(
  nextState: HookStateSetAction<S>,
  currentState?: C
): S;
export function resolveHookState<S, C extends S>(
  nextState: HookStateResolvable<S>,
  currentState?: C
): S {
  if (typeof nextState === "function") {
    return nextState.length
      ? (nextState as Function)(currentState)
      : (nextState as Function)();
  }

  return nextState;
}
```

### `useForceUpdate`

> `ref`를 이용해 리스트 데이터를 관리하므로, 데이터가 변경되었을 때 `forceUpdate()`를 수행하게끔 해주어야 UI에 반영이 가능하다.&#x20;

```typescript
import { useReducer } from "react";

const updateReducer = (value: number): number => (value + 1) % 1_000_000;

export default function useForceUpdate() {
  const [, forceUpdate] = useReducer(updateReducer, 0);
  
  return forceUpdate;
}
```

### useList

```typescript
import { useMemo, useRef } from "react";
import useForceUpdate from "./useForceUpdate";
import type { HookStateInitAction, HookStateSetAction } from "./hookState";
import { resolveHookState } from "./hookState";

export interface ListActions<T> {
  set: (newList: HookStateSetAction<T[]>) => void;
  push: (...items: T[]) => void;
  updateAt: (index: number, item: T) => void;
  insertAt: (index: number, item: T) => void;
  update: (predicate: (a: T, b: T) => boolean, newItem: T) => void;
  updateFirst: (predicate: (a: T, b: T) => boolean, newItem: T) => void;
  // update + insert
  upsert: (predicate: (a: T, b: T) => boolean, newItem: T) => void;
  sort: (compareFn?: (a: T, b: T) => number) => void;
  // Array.prototype.filter
  filter: <S extends T>(
    callbackFn: (value: T, index?: number, array?: T[]) => value is S,
    thisArg?: any
  ) => void;
  removeAt: (index: number) => void;
  clear: () => void;
  reset: () => void;
}

export default function useList<T>(
  initialList: HookStateInitAction<T[]> = []
): [T[], ListActions<T>] {
  const list = useRef(resolveHookState(initialList));
  const forceUpdate = useForceUpdate();

  const actions = useMemo(
    () =>
      ({
        set: (newList) => {
          list.current = resolveHookState(newList, list.current);
          forceUpdate();
        },
        push: (...items) => {
          items.length && actions.set((prevList) => prevList.concat(items));
        },
        updateAt: (index, item) => {
          actions.set((currentList) => {
            const nextList = currentList.slice();
            nextList[index] = item;
            return nextList;
          });
        },
        insertAt: (index, item) => {
          actions.set((currentList) => {
            const nextList = currentList.slice();
            index > nextList.length
              ? (nextList[index] = item)
              : nextList.splice(index, 0, item);
            return nextList;
          });
        },
        update: (predicate, newItem) => {
          actions.set((currentList) =>
            currentList.map((item) =>
              predicate(item, newItem) ? newItem : item
            )
          );
        },
        updateFirst: (predicate, newItem) => {
          const index = list.current.findIndex((item) =>
            predicate(item, newItem)
          );
          index >= 0 && actions.updateAt(index, newItem);
        },
        upsert: (predicate, newItem) => {
          const index = list.current.findIndex((item) =>
            predicate(item, newItem)
          );
          index >= 0 ? actions.updateAt(index, newItem) : actions.push(newItem);
        },
        sort: (compareFn) => {
          actions.set((currentList) => currentList.slice().sort(compareFn));
        },
        filter: (callbackFn, thisArg) => {
          actions.set((currentList) =>
            currentList.slice().filter(callbackFn, thisArg)
          );
        },
        removeAt: (index: number) => {
          actions.set((currentList) => {
            const nextList = currentList.slice();
            nextList.splice(index, 1);
            return nextList;
          });
        },
        clear: () => {
          actions.set([]);
        },
        reset: () => {
          actions.set(resolveHookState(initialList).slice());
        },
      } as ListActions<T>),
    []
  );

  return [list.current, actions];
}
```

### 참고 자료

* [react-use/useList.ts at master · streamich/react-use](https://github.com/streamich/react-use/blob/master/src/useList.ts)
* [react-spectrum/packages/@react-stately/data at main · adobe/react-spectrum](https://github.com/adobe/react-spectrum/tree/main/packages/@react-stately/data)
