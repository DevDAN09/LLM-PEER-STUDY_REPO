
### **1. 기본 정의**
- **`useQuery`**:
  - **목적**: 서버에서 데이터를 **조회**(읽기, GET 요청)하는 데 사용.
  - **주요 역할**: 데이터를 가져오고, 캐싱, 로딩/에러 상태 관리, 자동 리프레시 등을 처리.
  - **사용 사례**: 게시물 목록, 사용자 프로필, 검색 결과 등 데이터를 가져올 때.
- **`useMutation`**:
  - **목적**: 서버 데이터를 **변경**(쓰기, POST/PUT/DELETE 요청)하는 데 사용.
  - **주요 역할**: 데이터 생성/수정/삭제 요청을 실행하고, 낙관적 업데이트, 에러 처리, 캐시 갱신 등을 관리.
  - **사용 사례**: 좋아요 버튼 토글, 댓글 추가, 프로필 업데이트 등 데이터 변경 작업.

---

### **2. 주요 차이점**
| 항목                | `useQuery`                              | `useMutation`                           |
|---------------------|-----------------------------------------|-----------------------------------------|
| **목적**            | 데이터 조회 (읽기, GET)                 | 데이터 변경 (쓰기, POST/PUT/DELETE)     |
| **호출 방식**       | 컴포넌트 마운트 시 자동 실행            | 명시적으로 `mutate` 함수 호출           |
| **상태 관리**       | `isLoading`, `isError`, `data`, `refetch` 등 제공 | `isPending`, `isError`, `data`, `mutate` 등 제공 |
| **캐싱**            | 쿼리 키 기반으로 데이터 캐싱 및 재사용 | 캐싱 없음, 캐시 갱신은 개발자가 제어  |
| **자동 리프레시**   | 포커스, 네트워크 재연결 시 자동 리프레시 | 자동 리프레시 없음, 수동 갱신 필요    |
| **주요 사용 사례**  | 데이터 표시 (목록, 상세 정보)          | 데이터 변경 (폼 제출, 버튼 클릭)       |
| **반환 값**         | 데이터, 상태, 리프레시 함수 등          | `mutate` 함수와 상태 정보              |

---

### **3. 동작 방식**
- **`useQuery`**:
  - **자동 실행**: 컴포넌트가 마운트되면 `queryFn`을 실행해 데이터를 가져옴.
  - **캐싱**: `queryKey`를 기반으로 데이터를 캐싱하고, 동일한 키로 재요청 시 캐시 반환.
  - **상태 제공**: `isLoading`, `isError`, `data`, `error`, `refetch` 등으로 상태 관리.
  - **예시**:
    ```javascript
    import { useQuery } from '@tanstack/react-query';

    function UserComponent() {
      const { data, isLoading, error } = useQuery({
        queryKey: ['user', 1],
        queryFn: () => fetch('/api/user/1').then((res) => res.json()),
      });

      if (isLoading) return <div>Loading...</div>;
      if (error) return <div>Error: {error.message}</div>;
      return <div>Hello, {data.name}</div>;
    }
    ```
    - **설명**: 사용자 데이터를 가져오고, 로딩/에러 상태를 자동 관리. 동일한 `queryKey`로 캐시 재사용.

- **`useMutation`**:
  - **수동 실행**: `mutate` 함수를 호출해야 요청이 실행됨.
  - **캐시 관리**: 기본적으로 캐싱하지 않으며, `onSuccess`, `onError`, `onSettled`로 캐시 갱신 제어.
  - **낙관적 업데이트**: 요청 전에 UI를 업데이트하고, 실패 시 롤백 가능.
  - **예시**:
    ```javascript
    import { useMutation, useQueryClient } from '@tanstack/react-query';

    function LikeButton({ postId }) {
      const queryClient = useQueryClient();
      const mutation = useMutation({
        mutationFn: () => fetch(`/api/posts/${postId}/like`, { method: 'POST' }).then((res) => res.json()),
        onSuccess: () => {
          queryClient.invalidateQueries(['post', postId]); // 캐시 갱신
        },
      });

      return <button onClick={() => mutation.mutate()}>Like</button>;
    }
    ```
    - **설명**: 좋아요 버튼 클릭 시 서버에 POST 요청을 보내고, 성공 시 캐시를 갱신.

---

### **4. 세부 차이점**
1. **실행 시점**:
   - `useQuery`: 컴포넌트가 렌더링될 때 자동으로 쿼리 실행. `enabled` 옵션으로 제어 가능.
   - `useMutation`: `mutate` 또는 `mutateAsync`를 명시적으로 호출해야 실행.

2. **캐싱**:
   - `useQuery`: 쿼리 키로 캐싱, `staleTime`, `cacheTime`으로 캐시 동작 제어.
   - `useMutation`: 캐싱 없음. `onSuccess` 등으로 `QueryClient`를 사용해 캐시 업데이트.

3. **상태 관리**:
   - `useQuery`: `isLoading`, `isFetching`, `data`, `error` 등으로 쿼리 상태 관리.
   - `useMutation`: `isPending`, `isSuccess`, `isError`, `data` 등으로 mutation 상태 관리.

4. **낙관적 업데이트**:
   - `useQuery`: 주로 읽기 작업이라 낙관적 업데이트 불필요.
   - `useMutation`: `onMutate`로 낙관적 업데이트 구현, 실패 시 `onError`로 롤백.

5. **사용 패턴**:
   - `useQuery`: 데이터 표시, 리스트 렌더링, 페이지네이션(`useInfiniteQuery`).
   - `useMutation`: 버튼 클릭, 폼 제출, 데이터 수정/삭제.

---

### **5. 실제 예시: 좋아요 버튼**
#### **useQuery (데이터 조회)**
```javascript
function PostComponent({ postId }) {
  const { data: post, isLoading } = useQuery({
    queryKey: ['post', postId],
    queryFn: () => fetch(`/api/posts/${postId}`).then((res) => res.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  return <div>Likes: {post.likes}</div>;
}
```
- **역할**: 게시물 데이터를 가져오고, 캐싱하여 재사용.

#### **useMutation (데이터 변경)**
```javascript
function LikeButton({ postId }) {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: () => fetch(`/api/posts/${postId}/like`, { method: 'POST' }).then((res) => res.json()),
    onMutate: async () => {
      await queryClient.cancelQueries(['post', postId]);
      const previousPost = queryClient.getQueryData(['post', postId]);
      queryClient.setQueryData(['post', postId], (old) => ({
        ...old,
        likes: old.likes + 1,
      }));
      return { previousPost };
    },
    onError: (err, variables, context) => {
      queryClient.setQueryData(['post', postId], context.previousPost);
    },
    onSettled: () => {
      queryClient.invalidateQueries(['post', postId]);
    },
  });

  return <button onClick={() => mutation.mutate()}>Like</button>;
}
```
- **역할**: 좋아요 상태를 변경하고, 낙관적 업데이트로 즉시 UI 반영, 실패 시 롤백.

---

### **6. 언제 무엇을 사용하나?**
- **`useQuery`**:
  - 서버에서 데이터를 읽어야 할 때.
  - 캐싱, 자동 리프레시, 페이지네이션 필요 시.
  - 예: 사용자 목록, 게시물 상세 정보.
- **`useMutation`**:
  - 서버 데이터를 생성/수정/삭제해야 할 때.
  - 낙관적 업데이트, 에러 처리, 캐시 갱신 필요 시.
  - 예: 폼 제출, 좋아요 토글, 댓글 삭제.

---

### **7. 결론**
- **`useQuery`**: 데이터 **조회**와 캐싱, 상태 관리에 특화. 자동 실행과 캐시 기반 최적화.
- **`useMutation`**: 데이터 **변경**과 낙관적 업데이트, 수동 실행에 특화. 캐시 갱신은 개발자가 제어.
- **공통점**: 둘 다 비동기 작업을 간소화하고, 로딩/에러 상태를 체계적으로 관리.
- **차이점**: `useQuery`는 읽기 작업과 캐싱 중심, `useMutation`은 쓰기 작업과 캐시 업데이트 중심.