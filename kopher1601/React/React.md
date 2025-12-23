- 프런트엔드 개발시 주의해야할것은 UI를 처리하는 로직과, 비즈니스 기능을 처리하는 로직이 섞이지 않게 하는것이다. 즉 레이어의 분리가 중요하고 코드를 작성할 때 이점을 생각하면서 작성해야 한다

- React에서는 주로 이벤트를 나타내는 Prop에는 `onSomething`과 같은 이름을 사용하고, 이벤트를 처리하는 함수를 정의 할 때는 `handleSomething`과 같은 이름을 사용합니다




```ts
const products = [
  { title: 'Cabbage', isFruit: false, id: 1 },
  { title: 'Garlic', isFruit: false, id: 2 },
  { title: 'Apple', isFruit: true, id: 3 },
];

export default function ShoppingList() {
  const listItems = products.map(product =>
    <li
      key={product.id}
      style={{
        color: product.isFruit ? 'magenta' : 'darkgreen'
      }}
    >
      {product.title}
    </li>
  );

  return (
    <ul>{listItems}</ul>
  );
}
```
React는 나중에 항목을 삽입, 삭제 또는 재정렬할 때 어떤 일이 일어났는지 알기 위해 `key`를 사용합니다.

# 이벤트 응답하기
```ts
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```
`onClick={handleClick}`의 끝에 소괄호(`()`)가 없는 것을 주목하세요! 이벤트 핸들러 함수를 _호출_ 하지 않고 _전달_ 만 하면 됩니다. React는 사용자가 버튼을 클릭할 때 이벤트 핸들러를 호출합니다.

# Hook 사용하기
`use`로 시작하는 함수를  _Hook_ 이라고 합니다. `useState`는 React에서 제공하는 내장 Hook입니다.

Hook은 다른 함수보다 더 제한적입니다. 컴포넌트(또는 다른 Hook)의 _상단_ 에서만 Hook을 호출할 수 있습니다. 조건이나 반복에서 `useState`를 사용하고 싶다면 새 컴포넌트를 추출하여 그곳에 넣으세요.

# 컴포넌트 간에 데이터 공유하기

![[Screenshot 2025-10-23 at 20.15.20.png]]![[Screenshot 2025-10-23 at 20.15.32.png]]

# State 끌어 올리기
여러 자식 컴포넌트에서 데이터를 수집하거나 두 자식 컴포넌트가 서로 통신하도록 하려면, 부모 컴포넌트에서 공유 State를 선언하세요. 부모 컴포넌트는 Props를 통해 해당 State를 자식 컴포넌트에 전달할 수 있습니다. 이렇게 하면 자식 컴포넌트가 서로 동기화되고 부모 컴포넌트와도 동기화되도록 유지할 수 있습니다.

# 불변성
불변성을 사용하면 복잡한 기능을 훨씬 쉽게 구현할 수 있습니다. 우리는 이 자습서의 뒷부분에서 게임의 진행 과정을 검토하고 과거 움직임으로 “돌아가기”를 할 수 있는 “시간 여행” 기능을 구현할 예정입니다. 특정 작업을 실행 취소하고 다시 실행하는 기능은 이 게임에만 국한된 것이 아닌 앱의 일반적인 요구사항입니다. 직접적인 데이터 변경을 피하면 이전 버전의 데이터를 그대로 유지하여 나중에 재사용(또는 초기화)할 수 있습니다.

불변성을 사용하는 것의 또 다른 장점이 있습니다. 기본적으로 부모 컴포넌트의 State가 변경되면 모든 자식 컴포넌트가 자동으로 다시 렌더링 됩니다. 여기에는 변경 사항이 없는 자식 컴포넌트도 포함됩니다. 리렌더링 자체가 사용자에게 보이는 것은 아니지만 성능상의 이유로 트리의 영향을 받지 않는 부분의 리렌더링을 피하는 것이 좋습니다. 불변성을 사용하면 컴포넌트가 데이터의 변경 여부를 저렴한 비용으로 판단할 수 있습니다. [`memo` API 참고서](https://ko.react.dev/reference/react/memo)에서 React가 컴포넌트를 다시 렌더링할 시점을 선택하는 방법에 대해 살펴볼 수 있습니다.


### Outlet
react-router에서 페지이 컴포넌트가 실제 렌더링될 위치를 표시해준다
```ts
import { Outlet } from "react-router";

export default function GlobalLayout() {
  return (
    <div>
      <header>Header</header>
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

```ts
import { Navigate, Route, Routes } from "react-router";
import SignInPage from "@/pages/sign-in-page.tsx";
import SignUpPage from "@/pages/sign-up-page.tsx";
import ForgetPasswordPage from "@/pages/forget-password-page.tsx";
import IndexPage from "@/pages/index-page.tsx";
import PostDetailPage from "@/pages/post-detail-page.tsx";
import ProfileDetailPage from "@/pages/profile-detail-page.tsx";
import ResetPasswordPage from "@/pages/reset-password-page.tsx";
import GlobalLayout from "@/components/layout/global-layout.tsx";

export default function RootRoute() {
  return (
    <Routes>
      <Route element={<GlobalLayout />}>
        <Route path="/sign-in" element={<SignInPage />} />
        <Route path="/sign-up" element={<SignUpPage />} />
        <Route path="/forget-password" element={<ForgetPasswordPage />} />

        <Route path="/" element={<IndexPage />} />
        <Route path="/post/:postId" element={<PostDetailPage />} />
        <Route path="/profile/:userId" element={<ProfileDetailPage />} />
        <Route path="/reset-password" element={<ResetPasswordPage />} />

        <Route path="*" element={<Navigate to="/" />} />
      </Route>
    </Routes>
  );
}
```
# 서버 상태 관리
- [[TanStack Query]]

# createPortal
`createPortal`을 사용하면 일부 자식을 DOM의 다른 부분으로 렌더링 할 수 있습니다.
컴포넌트의 계층 구조를 벗어나서 지정한 DOM 요소 아래에 바로 렌더링 될 수 있도록 만들어주는게 `createPortal` 이다
```ts
createPortal(<그리고싶은컴포넌트 />, document.getElementById("modal-root"))
```
**예제**
```ts
export default function ModalProvider({  
  children,  
}: {  
  children: React.ReactNode;  
}) {  
  return (  
    <>  
      {createPortal(  
        <PostEditorModal />,  
        document.getElementById("modal-root")!,  
      )}  
      {children}  
    </>  
  );  
}
```