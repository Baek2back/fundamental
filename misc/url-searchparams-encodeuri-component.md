# URL(SearchParams), encodeURI(Component)

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

### 퍼센트 인코딩 (Percent Encoding)

퍼센트 인코딩 규약은 RFC 3986에 정의되어 있으며 명세에 따르면 URL에서 중요하게 사용되는 **예약(reserved) 문자**가 있고, 인코딩이 필요하지 않은 **비예약(unreserved) 문자**가 존재한다.



**RFC 3986 section 2.2 Reserved Characters**

예약 문자는 문법적인 의미를 가지고 있으므로 해당 문법적 의미로 사용되지 않게 하려면 반드시 인코딩해야 한다.

`!`, `*`, `'`, `(`, `)`, `;`, `:`, `@`, `&`, `=`, `+`, `$`, `,`, `/`, `?`, `#`, `[`, `]`

여기에 더해 안전하지 않다고 여겨지는 문자들 역시 인코딩을 수행해 주어야 한다.

&#x20;, `"`, `<`, `>`, `%`, `{`, `}`, `|`, `\`, `^`, `` ` ``



**RFC 3986 section 2.3 Unreserved Characters**

`[a-zA-Z]`, `[0-9]`, `-`, `.`, `_`, `~`

### `URL`, `URLSearchParams`

```typescript
URL === window.URL; // true

new URL(url, [base]);

const url = new URL("https://example.com?q=1");
url.searchParams.get("q"); // "1"

new URLSearchParams("q=1").get("q"); // "1"
```

RFC 3986 명세에 따라 자동으로 인코딩하여 변환해주며, 가장 최근에 도입된 API이다.

### `encodeURI` vs. `encodeURIComponent`

```typescript
const set = "-.!~*'()"; // Unreserved Marks

encodeURI(set);          // "-.!~*'()"
encodeURIComponent(set); // "-.!~*'()"
```

두 함수 모두 RFC 2396 명세 기준으로 비예약(unreserved) 문자로 취급되는 `-`, `.`, `!`, `~`, `*`, `'`, `(`, `)`는 인코딩하지 않는다.

```typescript
const set = ";/?:@&=+$,#"; // Reserved Characters

encodeURI(set);          // ";/?:@&=+$,#"
encodeURIComponent(set); // "%3B%2C%2F%3F%3A%40%26%3D%2B%24%23"
```

다만 `encodeURI`는 URL에 사용할 수 없는 문자만 인코딩하지만, `encodeURIComponent`는 예약 문자까지 모두 인코딩하는 차이가 존재한다.

```typescript
`?q=${encodeURI("a&b")}`;          // "?q=a&b"
`?q=${encodeURIComponent("a&b")}`; // "?q=a%26b"
```

따라서 Query Parameter에 적용하는 경우에는 `encodeURIComponent`를 사용하는 것이 더 올바른 접근이다.

### 정리

`URL`, `URLSearchParams`는 모두 RFC 3986을 기반으로 동작하지만, `encodeURI`, `encodeURIComponent` 함수는 보다 이전 버전인 RFC 2396을 기반으로 하므로 기본적으로는 `URL`, `URLSearchParams`를 고려하는 것이 좋다.

```typescript
const url = new URL("https://example.com?q=1+2");
url.search; // "?q=1+2"
url.searchParams.get("q"); // "1 2"

new URLSearchParams("q=1+2").get("q"); // "1 2"
```

다만 `URLSearchParams`은 **`+` 기호를 공백(space)로 해석**하기 때문에 문제를 발생시킬 수 있으므로 보다 안전하게 사용하기 위해서는 `encodeURIComponent`를 확장하여 RFC 2396 → RFC 3986을 충족시킬 수 있게끔 만드는 것도 고려하자.

```typescript
function encodeRFC3986URIComponent(value: string) {
  return encodeURIComponent(value).replace(
    /[!'()*]/g,
    (c) => `%${c.charCodeAt(0).toString(16).toUpperCase()}`
  );
}
```

### 참고 자료

[퍼센트 인코딩](https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%84%BC%ED%8A%B8\_%EC%9D%B8%EC%BD%94%EB%94%A9)

[URLSearchParams - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams#preserving\_plus\_signs)

[encodeURI() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global\_Objects/encodeURI#encoding\_for\_rfc3986)

[encodeURIComponent() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global\_Objects/encodeURIComponent#encoding\_for\_rfc3986)
