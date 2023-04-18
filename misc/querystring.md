# QueryString

> 특정 리소스를 식별하고 싶은 경우에는 `Path Variable` / 정렬이나 필터링이 필요한 경우에는 `Query Parameter`를 사용하는 것이 권장된다.

### `Path Variable` vs. `Query Parameter`

* Path Variable로 전달된 값을 이용해 리소스를 식별할 수 없다면 404가 발생할 것이고, Query Parameter를 이용해 조회를 한 경우에는 빈(empty) 값을 반환하게 될 것이다.
* 필터링을 수행할 때 404가 발생하는 것은 부적절하므로 정렬이나 필터링을 수행해야 하는 경우에 Query Parameter를 사용하는 것이다.

#### `req.params` -> Path Variable

```typescript
/:id/:name?query_1=...&query_2=...
   -> req.params = { id: ..., name: ... }
```

#### `req.query` -> Query Parameter

```typescript
/:id/:name?query_1=...&query_2=... 
  -> req.query = { query_1: ..., query_2: ... }
```

