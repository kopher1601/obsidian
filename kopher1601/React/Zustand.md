
## Store
```ts
import { create } from "zustand";

interface Store {
  count: number;
  increment: () => void;
  decrement: () => void;
}
export const useCountStore = create<Store>((set, get) => {
  return {
    count: 0,
    increment: () => {
      set({ count: get().count + 1 });
    },
    decrement: () => {
      set({ count: get().count - 1 });
    },
  }; // 여기서 반환하는 객체가 create 함수가 생성하는 store 로 설정이 된다.  
});
```

## Selector
밑의 코드는 육안으로 봤을 때는 increment 만 사용하고 있음으로 state 변경으로 인한 컴포는터 리렌더링이 발생하지 않을것 처럼 보이지만 실상은 `useCountStore` 자체는 store 전체를 반환하고 있고 그 중에 구조분해할당으로 increment 만을 사용하고 있는것이기 때문에 state 가 변경되면 현재 컴포넌트도 리렌더링되고 만다.
```ts
export default function Sample() {
	const {increment} = useCountStore();
	...
}
```
이것을 방지하기 위해 selector 를 사용해서 store 에서 특정 요소만 불러올 수 있다. 이를 통해 불필요한 리렌더링을 방지할 수 있다.
```ts
const increment = useCountStore((store) => store.increment);  
const decrement = useCountStore((store) => store.decrement);
```
더 나아가서는 관리 효율을 위해 하나의 파일에서 분리해서 관리하는게 좋다.
```ts
import { create } from "zustand";  
  
interface Store {  
  count: number;  
  increment: () => void;  
  decrement: () => void;  
}  
const useCountStore = create<Store>((set, get) => {  
  return {  
    count: 0,  
    increment: () => {  
      set({ count: get().count + 1 });  
    },  
    decrement: () => {  
      set({ count: get().count - 1 });  
    },  
  };  
});  
  
export function useCount() {  
  return useCountStore((store) => store.count);  
}  
  
export function useIncrementCount() {  
  return useCountStore((store) => store.increment);  
}  
  
export function useDecrementCount() {  
  return useCountStore((store) => store.decrement);  
}
```

**리렌더되는지 확인하는 방법**
- `Highlight updates when components render` 를 활성화해줘야 한다
![[Screenshot 2025-10-29 at 15.59.33.png]]

## Middleware
- middleware 적용시 순서가 중요함
	- 밑의 Persist 코드 참고
**middleware 의 종류**
![[Pasted image 20251029161227.png]]
### Combine
combine 미들웨어를 사용하면 초기 상태와 새로운 상태 슬라이스 및 동작을 추가하는 상태 생성 함수를 병합하여 응집력 있는 상태를 생성할 수 있습니다. 이 기능은 자동으로 유형을 추론하므로 명시적인 유형 정의가 필요하지 않아 매우 유용합니다.
```ts
import { create } from "zustand";  
import { combine } from "zustand/middleware";  
  
const useCountStore = create(  
  combine(  
    {  
      count: 0,  
    },  
    (set) => ({  
      increment: () => {  
        set((state) => ({ count: state.count + 1 }));  
      },  
      decrement: () => {  
        set((state) => ({ count: state.count - 1 }));  
      },  
    }),  
  ),  
);
```
### Immer
불변성을 관리해주는 미들웨어
```ts
import { create } from "zustand";  
import { combine } from "zustand/middleware";  
import { immer } from "zustand/middleware/immer";  
  
const useCountStore = create(  
  immer(  
    combine(  
      {  
        count: 0,  
      },  
      (set) => ({  
        increment: () => {  
          set((state) => (state.count += 1));  
        },  
        decrement: () => {  
          set((state) => (state.count -= 1));  
        },  
      }),  
    ),  
  ),  
);
```

### SubscribeWithSelector
```ts
import { create } from "zustand";  
import { combine, subscribeWithSelector } from "zustand/middleware";  
import { immer } from "zustand/middleware/immer";  
  
const useCountStore = create(  
  subscribeWithSelector(  
    immer(  
      combine(  
        {  
          count: 0,  
        },  
        (set) => ({  
          increment: () => {  
            set((state) => (state.count += 1));  
          },  
          decrement: () => {  
            set((state) => (state.count -= 1));  
          },  
        }),  
      ),  
    ),  
  ),  
);  
  
// useEffect 와 비슷한 역할을 한다  
useCountStore.subscribe(  
  (state) => state.count,  
  // listener  
  (count, prevCount) => {  
    console.log("count changed", count);  
    console.log("prevCount", prevCount);  
  },  
);
```

### Persist
```ts
import { combine, persist, subscribeWithSelector } from "zustand/middleware";  
import { immer } from "zustand/middleware/immer";  
  
const useCountStore = create(  
  persist(  
    // 브라우저의 localStorage 에 state 를 보관하게 해줌  
    subscribeWithSelector(  
      immer(  
        combine(  
          {  
            count: 0,  
          },  
          (set) => ({  
            increment: () => {  
              set((state) => (state.count += 1));  
            },  
            decrement: () => {  
              set((state) => (state.count -= 1));  
            },  
          }),  
        ),  
      ),  
    ),  
    {  
      name: "count-store",  
      // localStorage 에 보관하고 싶은 값만 지정할 수 있다.  
      partialize: (state) => ({ count: state.count }),  
      // 브라우저의 sessionStorage 에 값을 보관게 지정  
	  storage: createJSONStorage(() => localStorage),
    },  
  ),  
);
```
