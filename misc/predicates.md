---
description: 논리 연산자를 대체할 수 있는 함수
---

# predicates

```typescript
type Predicate<T extends any[]> = (...args: T) => boolean;

type PredicateOperator = <T extends any[]>(
  predicate: Predicate<T>
) => Predicate<T>;

type PredicateCombiner = <T extends any[]>(
  ...predicates: Predicate<T>[]
) => Predicate<T>;
```

### not

```typescript
/**
 * @example
 * const isOdd = not(isEven);
 * isOdd(x) === !isEven(x); // true
 *
 */

const not: PredicateOperator = 
  (predicate) =>
  (...inputs) =>
    !predicate(...inputs);
```

### all

```typescript
/**
 * @example
 * const isPositiveEven = all(isPositive, isEven);
 * isPositiveEven(2); // true, both positive & even
 * isPositiveEven(1); // false, only positive
 * isPositiveEven(-2); // false, only even
 */

const all: PredicateCombiner =
  (...predicates) =>
  (...inputs) =>
    predicates.every((predicate) => predicate(...inputs));
```

### any

```typescript
/**
 * @example
 * const isPositiveOrEven = any(isPositive, isEven)
 * isPositiveOrEven(1); // true, positive
 * isPositiveOrEven(-2); // true, even
 * isPositiveOrEven(-3); // false, neither
 */

const any: PredicateCombiner =
  (...predicates) =>
  (...inputs) =>
    predicates.some((predicate) => predicate(...inputs));
```

### none

```typescript
/**
 * @example
 * const isNotPostiveNorEven = none(isPositive, isEven);
 * isNotPositiveNorEven(-3); // true, neither
 * isNotPositiveNorEven(1); // false, positive
 * isNotPositiveNorEven(-2); // false, even
 */

const none: PredicateCombiner = (...predicates) =>
  not(any(...predicates));
```

### xor

```typescript
/**
 * @example
 * const isEitherPositiveOrEven = xor(isPositive, isEven);
 * isEitherPositiveOrEven(1); // true, only positive
 * isEitherPositiveOrEven(2); // false, both positive & even
 * isEitherPositiveOrEven(-3); // false, neither
 */

const xor: PredicateCombiner =
  (...predicates) =>
  (...inputs) => {
    let hasTrue = false;
    for (const predicate of predicates) {
      if (!predicate(...inputs)) continue;
      if (hasTrue) return false;
      hasTrue = true;
    }

    return hasTrue;
  };
```
