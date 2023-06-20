# mergeProps

```typescript
const mergedProps = mergeProps(
  { onClick: fn1 },
  { onClick: fn2, className: "blue" },
  { onClick: fn3, className: "button", style: { display: "block" }},
  { style: { display: "flex", color: "red" }}
);

/**
  { 
    className: "blue button",
    style: { display: "flex", color: "red" },
    onClick: () => {
      fn1();
      fn2();
      fn3();
    }
  }
 */
```

`renderProps`를 사용할 때나 부모로부터 전달받은 `className`, `style`, 이벤트 핸들러를 합성할 때 유용하게 사용할 수 있다.&#x20;

```typescript
function pushProp(target: Record<string, any>, key: string, value: any): void {
  if (key === "className") {
    target.className = [target.className, value]
      .filter(Boolean)
      .join(" ")
      .trim();
  } else if (key === "style") {
    target.style = { ...target.style, ...value };
  } else if (typeof value === "function") {
    target[key] = target[key] ? chain(target[key], value) : value;
  } else if (
    // skip merging undefined values
    value === undefined ||
    // skip if both value are the same primitive value
    (typeof value !== "object" && value === target[key])
  ) {
    return;
  } else if (!Object.prototype.hasOwnProperty.call(target, key)) {
    target[key] = value;
  } else {
    throw new Error(
      `Didn’t know how to merge prop '${key}'. ` +
        `Only 'className', 'style', and event handlers are supported`
    );
  }
}

/**
 * Merges sets of props together:
 *  - duplicate `className` props get concatenated
 *  - duplicate `style` props get shallow merged (later props have precedence for conflicting rules)
 *  - duplicate functions (to be used for event handlers) get called in order from left to right
 * @param props Sets of props to merge together. Later props have precedence.
 */
export function mergeProps<T extends Record<string, any>[]>(
  ...props: T
): {
  [K in keyof UnionToIntersection<T[number]>]: K extends "className"
    ? string
    : K extends "style"
    ? UnionToIntersection<T[number]>[K]
    : Exclude<Extract<T[number], { [Q in K]: unknown }>[K], undefined>;
} {
  if (props.length === 1) {
    return props[0] as any;
  }

  return props.reduce((merged, ps: any) => {
    for (const key of Object.keys(ps)) {
      pushProp(merged, key, ps[key]);
    }
    return merged;
  }, {}) as any;
}
```



### 참고 자료

* [GitHub - andrewbranch/merge-props: Merges className, style, and event handler props for React elements](https://github.com/andrewbranch/merge-props#readme)
* [mergeProps](https://react-spectrum.adobe.com/react-aria/mergeProps.html)
