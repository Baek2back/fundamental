# keys(), values(), entries()

```typescript
interface ObjectConstructor {
  keys(o: object): string[];
}
```

### `Object.keys()` 가 `string[]` 타입의 값을 반환하는 이유

`Object.keys()`가 `(keyof T)[]` 대신 `string[]` 타입의 값을 반환한다고 명시되어 있는 이유는 바로 타입스크립트에서의 **구조적 타이핑(structural typing)** 때문이다.

```typescript
interface Point {
  x: number;
  y: number;
}

function fn(key: keyof Point) {
  if (key === "x" || key === "y") {
    console.log(key);
  } else {
    throw new Error("Impossible Path");
  }
}
```

위의 `fn()` 은 `Point` 타입의 값이 전달된다면 절대로 에러를 `throw` 하지 않겠지만 `Point` 타입의 값을 기대하는 곳에 할당 가능한 `PointWithName` 이라는 타입의 값을 전달하게 되면 에러가 `throw` 될 것이다.

```typescript
interface PointWithName extends Point {
  name: string;
}

const pointWithName: PointWithName = {
  name: "origin",
  x: 0,
  y: 0
};

((point: Point) => {
  for (const key of Object.keys(point)) {
    // type 'string' is not assignable to
    // parameter of type 'keyof Point'
    fn(key);
  }
})(pointWithName);
```

만약 `Object.keys()` 가 `(keyof Point)[]` 타입을 반환한다고 선언되어 있다면 위의 함수 호출은 유효한 호출이 될 것이고, 의도와 달리 에러가 `throw` 될 수 있게 되는 것이다.

```typescript
function isKey<T extends object>(
  obj: T,
  key: PropertyKey
): key is keyof T {
  return Object.hasOwn(obj, key);
}

((point: Point) => {
  for (const key of Object.keys(point)) {
    if (isKey(point, key)) fn(key);
  }
})(pointWithName);
```

따라서 위와 같이 타입 가드 함수를 사용해야만 안전하게 사용할 수 있지만 **객체에 추가적인 속성은 없다는 제약을 준수 할 수 있다면** `Object.keys()` 를 감싼 래핑 함수를 고려해볼 수 있다.

```typescript
function keys<T extends object>(obj: T) {
  return Object.keys(obj) as unknown as (keyof T)[];
}

function values<T extends object>(obj: T) {
  return Object.values(obj) as unknown as T[keyof T][];
}

type Entries<T extends object> = {
  [K in keyof T]: [K, T[K]];
}[keyof T];

function entries<T extends object>(obj: T) {
  return Object.entries(obj) as unknown as Entries<T>;
}
```
