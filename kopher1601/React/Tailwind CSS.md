Tailwind CSS 를 사용하면 브라우저의 기본 스타일이 다 초기화 된다.
- 예) `h1` 의 스타일이 적용되지 않고 기본 텍스트로 표시됨
```ts
export default function CounterPage() {  
  return (  
    <div>  
      <h1>Counter</h1>  
    </div>  );  
}
```
## Width
- [width - Sizing](https://tailwindcss.com/docs/width)

| `w-<number>` | `width: calc(var(--spacing) * <number>);` |
| ------------ | ----------------------------------------- |
-  `"--spacing"` 의 기본값은 `4px` 이다 (`height`, `padding` 등 spacing이 사용되는 곳은 동일)
- 즉 `w-20` 이라고 설정할 시 `width: 80px` 이 설정된다
- 기본 spacing 값 없이 지정한 pixel 그대로 설정하고 싶은 경우 `w-[20px]` 설정하면 된다


