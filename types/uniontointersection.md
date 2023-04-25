# UnionToIntersection

```typescript
type UnionToIntersection<T> = (
  T extends unknown ? (x: T) => void : never
) extends (x: infer R) => void
  ? R
  : never;
```

```typescript
type Format320 = { urls: { format320p: string } };
type Format480 = { urls: { format480p: string } };
type Format720 = { urls: { format720p: string } };
type Format1080 = { urls: { format1080p: string } };

type Video = Format320 | Format480 | Format720 | Format1080;

const video1: Video = {
  urls: {
    format320p: "https://...",
  },
}; // ✅

const video2: Video = {
  urls: {
    format320p: "https://...",
    format480p: "https://...",
  },
}; // ✅

const video3: Video = {
  urls: {
    format1080p: "https://...",
  },
}; // ✅

// never
type FormatKeys = keyof Video["urls"];
```

위의 코드에서 FormatKeys 타입은 타입 내에서 교차(intersect)하는 공통의 키(key)가 없기 때문에 never 타입이 된다.

### Naked 타입

```typescript
type Naked<T> =
  T extends ... // naked!

type Unnaked<T> =
  { o: T } extends ... // not naked!
```

Naked 타입이란 T라는 타입을 무언가로 감싸지 않고 그 자체가 서브 타입 조건에 있는지 검증한다는 것을 의미한다.

```typescript
type Naked<T> =
  T extends unknown ? { o: T } : never;

type Foo = Naked<string | number | boolean>;

type Foo = Naked<string> | Naked<number> | Naked<boolean>;

type Foo = 
  | string extends unknown ? { o: string } : never
  | number extends unknown ? { o: number } : never
  | boolean extends unknown ? { o: boolean } : never;
  
type Foo = { o: string } | { o: number } | { o: boolean };
```

```typescript
type Unnaked<T> =
  { o: T } extends unknown ? { o: T } : never;

type Bar = Unnaked<string | number | boolean>;

type Bar = { o: string | number | boolean } extends unknown ?
  { o: string | number | boolean } : never;

type Bar = { o: string | number | boolean };
```

Naked 타입의 경우 T가 유니온이라면 분산 조건부 타입(distributive conditional types)이 적용되어 유니온 타입에 적용된 조건을 조건 타입의 유니온으로 변환하게 된다.

```typescript
type ToUnionOfFunction<T> = 
  T extends unknown ? (x: T) => void : never;

type Phase1 = ToUnionOfFunction<
  { a: string } | { b: string }
>;

type Phase2 =
  | ToUnionOfFunction<{ a: string }>
  | ToUnionOfFunction<{ b: string }>;

type UnionOfFunctions =
  | ((x: { a: string }) => void)
  | ((x: { b: string }) => void);
  
declare const foo: UnionOfFunctions;

// "b" is missing
foo({ a: "hello" });

// "a" is missing
foo({ b: "world" });

// OK
foo({ a: "hello", b: "world" });
```

여기서 핵심은 UnionOfFunction 타입을 갖는 함수 foo를 안전하게 호출하려면 함수의 요구 사항을 모두 만족하는 타입({ a: string, b: string })의 값을 전달해야 한다는 것이다.

### 반변 (Contravariant)

```typescript
UnionOfFunctions extends (x: infer R) => void
  ? R // { a: string } & { b: string }
  : never;
```

&#x20;그 다음으로 하는 작업은 함수 타입을 이용해 타입 T를 래핑하였다가 다시 언래핑하는 작업인데, 함수 인수를 통해 이러한 작업을 수행하는 이유는 추론된 타입 R이 반변(contravariant) 타입이 되기 때문이다.

```typescript
declare let str: string;
declare let strOrNum: string | number;

strOrNum = str; // ✅ string | number(상위 타입)에 string(하위 타입) 할당 가능

type Func<X> = (...args: X[]) => void;

declare let f: Func<string>;
declare let g: Func<string | number>;

g = f; // 💥 string | number(상위 타입)에 string(하위 타입) 할당 불가
```

함수 인수에 대해서는 상위 타입(super type)에 하위 타입(sub type)을 할당할 수 없다. 따라서 타입스크립트 입장에서는 반변하게 동작해야 하는 함수 인수에서 해당 타입을 추론(infer)해야 하므로 유니온 타입에서 인터섹션 타입으로 변환하여 추론하는 것이다.

### 정리

```typescript
type ToUnionOfFunction<T> = T extends unknown
  ? (x: T) => void
  : never;

type UnionToIntersection<T> = ToUnionOfFunction<T> extends (
  x: infer R
) => void
  ? R
  : never;

type Intersected = UnionToIntersection<Video["urls"]>;

type Intersected = UnionToIntersection<
  | { format320p: string }
  | { format480p: string }
  | { format720p: string }
  | { format1080p: string }
>;

type Intersected =
  | UnionToIntersection<{ format320p: string }>
  | UnionToIntersection<{ format480p: string }>
  | UnionToIntersection<{ format720p: string }>
  | UnionToIntersection<{ format1080p: string }>;

type Intersected = 
  (x: { format320p: string }) => void extends 
    (x: infer R) => void ? R : never | 
  (x: { format480p: string }) => void extends 
    (x: infer R) => void ? R : never | 
  (x: { format720p: string }) => void extends 
    (x: infer R) => void ? R : never | 
  (x: { format1080p: string }) => void extends 
    (x: infer R) => void ? R : never;

// infer R
type Intersected = 
  { format320p: string } & 
  { format480p: string } &
  { format720p: string } & 
  { format1080p: string };

// "format320p" | "format480p" | "format720p" | "format1080p"
type FormatKeys = keyof UnionToIntersection<Video["urls"]>;
```

### 참고 자료

[TypeScript: Union to intersection type](https://fettblog.eu/typescript-union-to-intersection/)

[Explain Like I'm Five: TypeScript UnionToIntersection type](https://dev.to/anuraghazra/explain-like-im-five-typescript-uniontointersection-type-5ako)

[Transform union type to intersection type](https://stackoverflow.com/questions/50374908/transform-union-type-to-intersection-type/50375286#50375286)

[type-fest/union-to-intersection.d.ts at main · sindresorhus/type-fest](https://github.com/sindresorhus/type-fest/blob/main/source/union-to-intersection.d.ts)
