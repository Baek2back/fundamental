# hasOwn()

### `in` 연산자

```typescript
'name' in { name: 'Name' }; // true
'name' in {}; // false
'toString' in {}; // true
```

`in` 연산자는 프로토타입 체인을 따라 확인할 수 있는 프로퍼티에 대해서는 `true` 를 반환하게 된다.

### `Object.prototype.hasOwnProperty()`

```typescript
Object.create(null).hasOwnProperty('name');
// TypeError: ...hasOwnProperty is not a function

({ hasOwnProperty: 'yes' }).hasOwnProperty('name');
// TypeError: ...hasOwnProperty is not a function
```

따라서 해당 객체가 상속받은 속성 외의 속성에 대해서만 검증하기 위해 `Object.prototype.hasOwnProperty()`를 사용하더라도 두 가지 문제가 존재하게 된다.

1. `Object.create(null)`을 이용하여 생성한 객체는 `Object.prototype`을 상속받지 않기 때문에 해당 메서드를 사용하는 것이 불가능하다.
2. 객체 자체에 `hasOwnProperty`라는 속성을 재정의하게 되면 해당 속성이 `Object.prototype.hasOwnProperty()`를 가려버리게 된다.

### `Object.hasOwn()`

```typescript
function hasOwn<
  X extends object,
  Y extends PropertyKey
>(obj: X, prop: Y): obj is X & Record<Y, unknown> {
  return Object.prototype.hasOwnProperty.call(
    obj,
    prop
  );
}

hasOwn(Object.create(null), 'name'); // false
hasOwn({ hasOwn: 'yes' }, 'name'); // false

Reflect.has({}, 'toString'); // true
Object.hasOwn({}, 'toString'); // false

Object.hasOwn(Object.create(null), 'name'); // false
Object.hasOwn({ hasOwn: 'yes' }, 'name'); // false
```

ES2022에 도입된 `Object.hasOwn()` 을 사용하거나 `hasOwn()` 형태로 래핑하여 사용하게 되면 상속받은 속성은 제외하게끔 할 수 있다.
