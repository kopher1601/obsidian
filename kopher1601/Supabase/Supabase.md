

Supabase Auth 를 사용해서 회원가입을하고 요청에 성공할 경우 Supabase 는 해당 유저를 로그인된 상태로 처리한다. 즉, `access_token과` `refresh_token` 을 발급하고 브라우저의 Local Storage 에 자동으로 저장해준다.(Supabase Client 를 사용하고 있을 경우)

> **슈퍼베이스는 왜 Local Storage에 토큰을 저장하는가**
> [Advanced guide | Supabase Docs](https://supabase.com/docs/guides/auth/server-side/advanced-guide)

## Event Trigger
Supabase 의 Session 이 변경되었을때 실행되는 이벤트핸들러
```ts
import RootRoute from "@/root-route.tsx";  
import { useEffect } from "react";  
import supabase from "@/lib/supabase.ts";  
  
function App() {  
  useEffect(() => {  
    supabase.auth.onAuthStateChange((event, session) => {});  
  
    return () => {  
      // 登録したevent handlerを消すためにclean upが必要だが、AppがUnmountされたということはサービスから出たという意味なので  
      // ここでは設定しない  
    };  
  }, []);  
  
  return <RootRoute />;  
}  
  
export default App;
```