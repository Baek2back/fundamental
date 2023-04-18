# to

> Golang / Lua에서의 응답과 동일한 형태로 비동기 로직을 처리할 수 있게 도와주는 함수

```typescript
// response -> [data, undefined]
// response -> [null, Error]

function to<T, U = Error>(
): Promise<[U, undefined] | [null, T]> {
  return promise
    .then<[null, T]>((data: T) => [null, data])
    .catch<[U, undefined]>((error: U) => {
      return [error, undefined];
    });
}


(async () => {
  const [error, response] = await to(findById(1));
  
  if (error) { ... } // API 요청 시 에러가 발생한 상황
  if (!response) { ... } // API 요청은 성공했으나 데이터가 없는 상황
})();
```

### 참고 자료

* [https://github.com/scopsy/await-to-js/blob/master/src/await-to-js.ts](https://github.com/scopsy/await-to-js/blob/master/src/await-to-js.ts)
