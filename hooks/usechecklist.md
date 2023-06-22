# useCheckList

```typescript
export interface ListItem {
  id: string | number;
  checked?: boolean;
}

export interface CheckListActions<T extends ListItem> {
  set: (newList: HookStateSetAction<T[]>) => void;
  checkAll: () => void;
  unCheckAll: () => void;
  check: (id: T["id"]) => void;
  unCheck: (id: T["id"]) => void;
  updateItem: (id: T["id"], checked: boolean) => void;
  toggle: (id: T["id"]) => void;
  toggleAll: () => void;
  updateAll: (checked: boolean) => void;
  reset: () => void;
}

export interface CheckListSelectors<T extends ListItem> {
  getCheckedList: () => T[];
  getCheckedIds: () => T["id"][];
  isChecked: (id: T["id"]) => boolean;
  isAllChecked: () => boolean;
}

export function useCheckList<T extends ListItem>(
  initialList: HookStateInitAction<T[]> = []
): [T[], CheckListActions<T>, CheckListSelectors<T>] {
  const list = useRef(resolveHookState(initialList));
  const forceUpdate = useForceUpdate();

  const actions = useMemo(
    () =>
      ({
        set: (newList) => {
          list.current = resolveHookState(newList, list.current);
          forceUpdate();
        },
        updateItem: (id, checked) => {
          actions.set((currentList) =>
            currentList.map((item) => ({
              ...item,
              checked: item.id === id ? checked : item.checked,
            }))
          );
        },
        toggle: (id: T["id"]) => {
          actions.set((currentList) =>
            currentList.map((item) => ({
              ...item,
              checked: item.id === id ? !item.checked : item.checked,
            }))
          );
        },
        updateAll: (checked) => {
          actions.set((currentList) =>
            currentList.map((item) => ({ ...item, checked }))
          );
        },
        checkAll: () => {
          actions.updateAll(true);
        },
        unCheckAll: () => {
          actions.updateAll(false);
        },
        toggleAll: () => {
          actions.set((currentList) =>
            currentList.map((item) => ({ ...item, checked: !item.checked }))
          );
        },
        reset: () => {
          actions.set(resolveHookState(initialList).slice());
        },
      } as CheckListActions<T>),
    []
  );

  const selectors = useMemo(
    () =>
      ({
        getCheckedList: () => list.current.filter((item) => item.checked),
        getCheckedIds: () =>
          list.current.filter((item) => item.checked).map(({ id }) => id),
        isChecked: (id) =>
          Boolean(list.current.find((item) => item.id === id && item.checked)),
        isAllChecked: () => list.current.every(({ checked }) => checked),
      } as CheckListSelectors<T>),
    []
  );

  return [list.current, actions, selectors];
}

```
