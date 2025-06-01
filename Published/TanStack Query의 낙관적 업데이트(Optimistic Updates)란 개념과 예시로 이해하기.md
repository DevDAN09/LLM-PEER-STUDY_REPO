

## 낙관적 업데이트란?

낙관적 업데이트는 **서버 요청의 응답을 기다리지 않고, 클라이언트 UI를 즉시 업데이트**하여 사용자에게 빠른 반응성을 제공하는 기법입니다. 서버 요청이 성공할 것이라고 "낙관적으로" 가정하고, UI를 먼저 변경한 뒤, 서버 응답에 따라 최종 상태를 조정하거나 실패 시 원래 상태로 되돌리는(롤백) 방식으로 동작합니다.

### **왜 중요한가?**
현대 웹 애플리케이션에서 사용자는 즉각적인 피드백을 기대합니다. 예를 들어, 소셜 미디어에서 "좋아요" 버튼을 눌렀을 때, 1~2초의 네트워크 지연이 있다면 사용자는 앱이 느리다고 느낄 수 있습니다. 낙관적 업데이트는 이런 지연을 숨기고, 버튼 클릭 즉시 UI를 변경해 부드러운 사용자 경험을 제공합니다. TanStack Query는 이를 쉽게 구현할 수 있도록 도와줍니다.

### **주요 특징**
1. **UI 업데이트**: 서버 응답 전에 클라이언트 캐시를 수정해 UI에 반영.
2. **롤백 메커니즘**: 서버 요청이 실패하면 UI를 원래 상태로 복원.
3. **서버 동기화**: 성공 시 서버 데이터를 기반으로 캐시를 갱신.
4. **간결한 코드**: TanStack Query의 `useMutation` 훅으로 복잡한 로직을 단순화.

---

## TanStack Query에서 낙관적 업데이트의 동작 원리

TanStack Query에서 낙관적 업데이트는 주로 `useMutation` 훅을 통해 구현됩니다. `useMutation`은 데이터 변경 작업(POST, PUT, DELETE 요청)을 처리하며, 아래 콜백을 사용해 낙관적 업데이트를 관리합니다:

- **`onMutate`**: 서버 요청 전에 실행. 캐시를 낙관적으로 업데이트하고, 롤백용 데이터를 저장.
- **`onError`**: 요청 실패 시 이전 데이터를 복원(롤백)하고, 에러 메시지 표시.
- **`onSettled`**: 성공/실패 여부에 관계없이 캐시를 최신화(예: `invalidateQueries` 호출).
- **`QueryClient`**: 캐시를 조작하거나 쿼리를 갱신하는 데 사용.

이제 실제 예시를 통해 어떻게 작동하는지 살펴보겠습니다.

---

## 예시: 좋아요 버튼에 낙관적 업데이트 적용하기

소셜 미디어 게시물의 "좋아요" 버튼을 구현한다고 가정해 봅시다. 사용자가 버튼을 누르면 즉시 좋아요 수가 증가하고 버튼이 "Unlike"로 바뀌어야 합니다. 서버 요청이 실패하면 UI를 원래 상태로 되돌리고 사용자에게 에러를 알립니다.

### **시나리오**
- 게시물에 좋아요 수(`likes`)와 좋아요 상태(`isLiked`)가 있음.
- 버튼 클릭 시 POST 요청으로 서버에 좋아요를 추가.
- UI는 즉시 좋아요 수를 +1하고, 버튼을 "Unlike"로 변경.
- 서버 요청이 실패하면 원래 상태로 복원.

### **코드 구현**
```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { toast } from 'react-toastify'; // 에러 알림용

// 좋아요 토글 API 호출 (모의)
const toggleLike = async (postId) => {
  const response = await fetch(`/api/posts/${postId}/like`, {
    method: 'POST',
  });
  if (!response.ok) throw new Error('좋아요 토글에 실패했습니다.');
  return response.json();
};

// 좋아요 버튼 컴포넌트
function LikeButton({ postId }) {
  const queryClient = useQueryClient();

  // 게시물 데이터 가져오기
  const { data: post, isLoading } = useQuery({
    queryKey: ['post', postId],
    queryFn: async () => {
      const response = await fetch(`/api/posts/${postId}`);
      if (!response.ok) throw new Error('게시물 조회에 실패했습니다.');
      return response.json();
    },
  });

  // 좋아요 토글 mutation
  const mutation = useMutation({
    mutationFn: toggleLike,
    // 낙관적 업데이트
    onMutate: async () => {
      // 진행 중인 쿼리 취소
      await queryClient.cancelQueries(['post', postId]);
      // 롤백용 이전 데이터 저장
      const previousPost = queryClient.getQueryData(['post', postId]);
      // 캐시 업데이트
      queryClient.setQueryData(['post', postId], (old) => ({
        ...old,
        likes: old.isLiked ? old.likes - 1 : old.likes + 1,
        isLiked: !old.isLiked,
      }));
      return { previousPost }; // 롤백용 데이터 반환
    },
    // 실패 시 롤백
    onError: (error, postId, context) => {
      queryClient.setQueryData(['post', postId], context.previousPost);
      toast.error('좋아요 토글에 실패했습니다. 다시 시도해주세요.');
    },
    // 성공/실패 후 캐시 갱신
    onSettled: () => {
      queryClient.invalidateQueries(['post', postId]);
    },
  });

  if (isLoading) return <div>로딩 중...</div>;

  return (
    <button onClick={() => mutation.mutate(postId)}>
      {post.isLiked ? 'Unlike' : 'Like'} ({post.likes})
    </button>
  );
}

export default LikeButton;
```

### **코드 동작 과정**
1. **초기 상태**: `useQuery`로 게시물 데이터를 가져옴(예: `{ id: postId, likes: 10, isLiked: false }`).
2. **버튼 클릭**: 사용자가 "Like" 버튼을 클릭하면 `mutation.mutate(postId)` 호출.
3. **낙관적 업데이트 (`onMutate`)**:
   - 진행 중인 쿼리를 취소(`cancelQueries`)하여 충돌 방지.
   - 이전 데이터를 저장(`previousPost`).
   - 캐시를 업데이트(`setQueryData`)하여 `likes`를 +1, `isLiked`를 `true`로 변경.
   - UI는 즉시 "Unlike (11)"로 반영.
4. **서버 요청**: `toggleLike` 함수로 POST 요청 전송.
5. **성공 시**: 서버가 성공 응답(예: `{ likes: 1, isLiked: true }`)을 반환하면, `onSettled`에서 `invalidateQueries`로 최신 데이터를 가져와 캐시 갱신.
6. **실패 시**: `onError`에서 이전 데이터(`previousPost`)로 캐시를 복원, UI는 "Like (10)"로 롤백, 에러 알림 표시.

### **실행 흐름 예시**
- **초기**: "Like (10)", `isLiked: false`.
- **클릭 후**: 즉시 "Unlike (11)", `isLiked: true` (낙관적 업데이트).
- **성공**: 서버 응답으로 캐시 유지, UI는 "Unlike (11)" 그대로.
- **실패**: 캐시 롤백, UI는 "Like (10)"로 복원, 에러 메시지 표시.

---

## 낙관적 업데이트의 장점과 주의점

### **장점**
1. **빠른 UX**: 네트워크 지연을 숨겨 사용자에게 즉각적인 피드백 제공.
2. **간결한 코드**: TanStack Query의 `useMutation`으로 복잡한 캐시 관리와 롤백 로직을 쉽게 구현.
3. **데이터 일관성**: 실패 시 롤백과 캐시 갱신으로 서버-클라이언트 상태 동기화.

### **주의점**
1. **실패 가능성**: 서버 요청 실패가 빈번한 경우, 잦은 롤백으로 UX가 저하될 수 있음. 성공 확률이 높은 작업(예: 좋아요, 댓글 추가)에 적합.
2. **복잡한 캐시 관리**: 여러 쿼리가 연관된 경우, 캐시 업데이트 로직이 복잡해질 수 있음.
3. **에러 처리**: 사용자에게 명확한 에러 피드백(예: 토스트 알림, 재시도 버튼) 제공 필수.

---

## 언제 낙관적 업데이트를 사용해야 할까?
낙관적 업데이트는 다음과 같은 상황에서 특히 유용합니다:
- **빠른 피드백이 중요한 경우**: 좋아요, 댓글 추가, 폼 제출 등.
- **서버 요청 성공률이 높은 경우**: 실패 가능성이 낮은 안정적인 API.
- **캐시와 UI 동기화가 필요한 경우**: TanStack Query의 캐시 관리와 잘 맞음.

반대로, 실패 가능성이 높거나 데이터 무결성이 매우 중요한 작업(예: 금융 거래)에는 신중히 사용해야 합니다.

---

**참고**:
- TanStack Query 공식 문서: [https://tanstack.com/query](https://tanstack.com/query)
- 작성일: 2025년 6월 2일

---
