
```ts
const {  
  data: todos,  
  isLoading,  
  error,  
} = useQuery({  
  queryFn: fetchTodos, // 컴포넌트가 마운시 호출해줌  
  queryKey: ["todos"], // tanstack query의 저장소에 지정한 키 이름으로 응답값이 저장됨  
  retry: 0, // 자동 재실행 기능 끄기
  // ... 등등 옵션이 많음
});
```

## Caching
### Tanstack Query 캐시의 5가지 상태
![[Pasted image 20251101202925.png]]

#### Fetching
데이터를 불러오는 중 일때
![[Pasted image 20251101203042.png]]

#### Fresh
데이터가 신선한 상태일 때
![[Pasted image 20251101203100.png]]

#### Stale
유통기한이 지나 데이터가 상한 상태
![[Pasted image 20251101203328.png]]
- `staleTime` : 기본 0초로 설정되어 있음, data 가 fresh 일 때만 유효함
![[Pasted image 20251101203201.png]]
![[Pasted image 20251101203517.png]]

#### InActive
![[Pasted image 20251101212800.png]]
![[Pasted image 20251101212824.png]]

## useMutation
- `useMutation` 은 `useQuery` 와 다르게 컴포넌트가 마운트된다고 바로 작동하지는 않는다. `mutate` 함수를 사용해서 트리거 해야 한다.

### 캐시 무효화를 통한 데이터 최신화
```ts
export function useCreateTodoMutation() {  
  const queryClient = useQueryClient();  
  
  return useMutation({  
    mutationFn: createTodo,  
    onSuccess: () => {  
      queryClient.invalidateQueries({ queryKey: ["todos"] });  
    },  
  });  
}
```
지정한 키의 캐시 데이터를 무효화한다. 캐시가 무효화되면 자동적으로 refetching 을 하게 된다
하지만 지정한 키의 캐시 데이터가 무효화 되기 때문에 원하지 않는곳에서도 데이터가 무효화됨에 따른 비효율성이 발생할 수 있다. 
해당 내용을 해결하기 위해 키 값을 조금 더 구조화 시키는 방법이 있을 수 있다.
```ts
export const QUERY_KEYS = {  
  todo: {  
    all: ["todo"],  
    list: ["todo", "list"],  
    detail: (id: string) => ["todo", "detail", id],  
  },  
};
```

```ts
export function useTodosData() {  
  return useQuery({  
    queryFn: fetchTodos,  
    queryKey: QUERY_KEYS.todo.list,  
  });  
}

...

export function useTodoDataById(id: string) {  
  return useQuery({  
    queryFn: () => fetchTodoById(id),  
    queryKey: QUERY_KEYS.todo.detail(id),  
  });  
}

...

export function useCreateTodoMutation() {  
  const queryClient = useQueryClient();  
  
  return useMutation({  
    mutationFn: createTodo,  
    onSuccess: async () => {  
      await queryClient.invalidateQueries({ queryKey: QUERY_KEYS.todo.list });  
    },  
  });  
}
```

## 낙관적 업데이트
빠른 반응을 제공해야 하는 SNS에서 주로 사용된다 (ex: 좋아요)
```ts
const { mutate } = useUpdateTodoMutation();  
  
const handleCheckboxClick = () => {  
  mutate({ id, isDone: !isDone });  
};
```

```ts
export function useUpdateTodoMutation() {  
  const queryClient = useQueryClient();  
  
  return useMutation({  
    mutationFn: updateTodo,  
    onMutate: (updatedTodo) => {  
      // mutate 함수가 실행될때 전달된 매개변수를 받아올 수 있다  
      // 여기서 낙관적 쿼리관련 로직을 구현  
      queryClient.setQueryData<Todo[]>(QUERY_KEYS.todo.list, (oldData) => {  
        if (!oldData) return;  
  
        return oldData.map((prevTodo) =>  
          prevTodo.id === updatedTodo.id  
            ? { ...prevTodo, ...updatedTodo }  
            : prevTodo,  
        );  
      });  
    },  
  });  
}
```

### 요청 실패시 복구
**에러 발생시 복구하기**
```ts
import { useMutation, useQueryClient } from "@tanstack/react-query";  
import { updateTodo } from "@/api/update-todo.ts";  
import type { Todo } from "@/types.ts";  
import { QUERY_KEYS } from "@/lib/constants.ts";  
  
export function useUpdateTodoMutation() {  
  const queryClient = useQueryClient();  
  
  return useMutation({  
    mutationFn: updateTodo,  
    onMutate: async (updatedTodo) => {  
      // 데이터 변경 처리 전에 조회 처리 후 업데이트 처리가 있을 경우 
      // 데이터 일관성의 불일치가 발생할 수 있기 때문에 쿼리 처리가 있는 경우 캔슬시킨다.  
      await queryClient.cancelQueries({  
        queryKey: QUERY_KEYS.todo.list,  
      });  
  
      // 실패를 대비한 백업 데이터  
      const prevTodos = queryClient.getQueryData<Todo[]>(QUERY_KEYS.todo.list);  
  
      // mutate 함수가 실행될때 전달된 매개변수를 받아올 수 있다  
      // 여기서 낙관적 쿼리관련 로직을 구현  
      queryClient.setQueryData<Todo[]>(QUERY_KEYS.todo.list, (oldData) => {  
        if (!oldData) return;  
  
        return oldData.map((prevTodo) =>  
          prevTodo.id === updatedTodo.id  
            ? { ...prevTodo, ...updatedTodo }  
            : prevTodo,  
        );  
      });  
  
      return {  
        prevTodos,  
      };  
    },  
    onError: (_error, _updatedTodo, context) => {  
      if (context && context.prevTodos) {  
        queryClient.setQueryData<Todo[]>(  
          QUERY_KEYS.todo.list,  
          context.prevTodos,  
        );  
      }  
    },  
    onSettled: async () => {  
      await queryClient.invalidateQueries({  
        queryKey: QUERY_KEYS.todo.list,  
      });  
    },  
  });  
}
```

> 안전장치로 요청이 종료될 때 불리는 `onSettled` 메서드에서 캐시 데이터를 무효화시켜서 `refetching` 시켜서 데이터의 일관성을 담보한다

## 캐시 정규화
**데이터의 평탄화**
![[Pasted image 20251104142649.png]]
![[Pasted image 20251104142550.png]]
![[Pasted image 20251104143250.png]]
**중복 데이터의 제거**
![[Pasted image 20251104142616.png]]
