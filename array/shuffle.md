# shuffle

### 피셔 에이츠 알고리즘 (Fisher Yates Shuffle)

배열을 순회하면서 해당 요소 이전(혹은 이후)에 있는 임의의 요소와 해당 요소를 바꾸는 알고리즘으로 생성되는 순열이 거의 유사한 빈도로 만들어지며 정렬이 수행되지 않기 때문에 성능 상 이점이 있다.

```typescript
function shuffle<T>(array: T[]): T[] {
  let currentIndex = array.length,
    randomIndex;

  while (currentIndex !== 0) {
    // 0보다 크고 currentIndex보다 작은 수 중에 무작위의 숫자를 선택한다.
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex--;

    [array[currentIndex], array[randomIndex]] = [
      array[randomIndex],
      array[currentIndex],
    ];
  }

  return array;
}
```
