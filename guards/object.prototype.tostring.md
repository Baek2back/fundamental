---
description: 런타임에 타입 검사가 필요한 경우에는 사용을 고려해보자
---

# Object.prototype.toString()

### `typeof` 연산자

```typescript
typeof null;       // object
typeof [];         // object
typeof {};         // object
typeof new Date(); // object
typeof /foo/;      // object
typeof (() => {}); // function
```

`typeof` 연산자는 `number`, `string`, `undefined`, `boolean`, `symbol`, `function`에 대해 잘 동작하지만 주의해야 할 사항이 존재한다.

1. `typeof null === "object"`는 널리 알려진 버그 중 하나이다.
2. `function`을 제외하고 객체 타입이라면 모두 `"object"`를 반환한다.

### `instanceof` 연산자

```typescript
function Car() {}
const car = new Car();
car instanceof Car; // ✅ true
```

`instanceof` 연산자는 객체의 생성자(constructor)를 확인하는 방식으로 동작하며, 주어진 값을 생성한 생성자 함수(또는 클래스)를 확인하는 것이다.

따라서 객체에 대한 타입은 올바르게 검증할 수 있지만 원시 타입의 값에 대해서는 검증이 불가능하다.

```typescript
[] instanceof Array;            // ✅ true
(() => {}) instanceof Function; // ✅ true
new Map() instanceof Map;       // ✅ true

1 instanceof Number;     // ❌ false 
'foo' instanceof String; // ❌ false
```

또한 객체의 생성자(constructor)를 검사하는 방식이므로 런타임에 객체의 프로토타입을 수정하면 의도와 다른 결과가 나오게 될 수 있다.

```typescript
const array = [];

array instanceof Array; // ✅ true

Object.setPrototypeOf(array, null);

array instanceof Array; // ❌ false
```

게다가 `Object.create(null)` 로 생성한 객체를 제외하면 모든 객체 타입은 `Object.prototype` 을 상속받으므로 `instanceof Object` 를 수행하면 `true`가 나온다.

```typescript
[] instanceof Object;                  // ✅ true
({}) instanceof Object;                // ✅ true
(() => {}) instanceof Object;          // ✅ true
new Date() instanceof Object;          // ✅ true
Object.create(null) instanceof Object; // ❌ false
```

### `Object.prototype.toString()`

{% hint style="info" %}
`Function.prototype.call()` 을 사용하여 호출하는 이유는 `Object.create(null)` 로 생성한 객체와 `null`, `undefined` 는 `toString()` 메서드를 갖지 않아 런타임 에러를 발생시키기 때문이다.
{% endhint %}

```typescript
Object.prototype.toString.call({}); // [object Object]
Object.prototype.toString.call(Object.create(null)); // [object Object]
Object.prototype.toString.call(1); // [object Number]
Object.prototype.toString.call('1'); // [object String]
Object.prototype.toString.call(true); // [object Boolean]
Object.prototype.toString.call(() => {}); // [object Function]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call(undefined); // [object Undefined]
Object.prototype.toString.call(/123/g); // [object RegExp]
Object.prototype.toString.call(new Date()); // [object Date]
Object.prototype.toString.call([]); // [object Array]
Object.prototype.toString.call(document); // [object HTMLDocument]
Object.prototype.toString.call(window); // [object Window]
```
