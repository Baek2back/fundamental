# chain

동일한 인수로 모든 함수들을 연쇄적으로 호출하기 위해 사용하는 유틸리티 함수

```typescript
function chain(...callbacks: any[]): (...args: any[]) => void {
  return (...args: any[]) => {
    for (const callback of callbacks) {
      if (typeof callback === "function") {
        callback(...args);
      }
    }
  }
}
```

### 참고 자료

[react-spectrum/packages/@react-aria/utils/src/chain.ts at main · adobe/react-spectrum](https://github.com/adobe/react-spectrum/blob/main/packages/@react-aria/utils/src/chain.ts)
