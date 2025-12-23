JSX는 HTML보다 엄격합니다. JSX에서는 `<br />`같이 태그를 닫아야 합니다. 또한 컴포넌트는 여러 개의 JSX 태그를 반환할 수 없습니다. `<div>...</div>` 또는 빈 `<>...</>` 래퍼와 같이 공유되는 부모로 감싸야 합니다.
```ts
function AboutPage() {
  return (
    <>
      <h1>About</h1>
      <p>Hello there.<br />How do you do?</p>
    </>
  );
}
```
React에서는 `className`으로 CSS 클래스를 지정합니다. 이것은 HTML의 [`class`](https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/class) 어트리뷰트와 동일한 방식으로 동작합니다.
```ts
<img className="avatar" />
```



```ts
<>
      <h1>{user.name}</h1>
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Photo of ' + user.name}
        style={{
          width: user.imageSize,
          height: user.imageSize
        }}
      />
</>
```
위의 예시에서 `style={{}}`은 특별한 문법이 아니라 `style={ }` JSX 중괄호 안에 있는 일반 `{}` 객체입니다. 스타일이 자바스크립트 변수에 의존하는 경우 `style` 어트리뷰트를 사용할 수 있습니다.

