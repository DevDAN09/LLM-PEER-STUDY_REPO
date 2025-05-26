### `useSyncExternalStore`에 대한 설명

`useSyncExternalStore`는 React 18에서 도입된 훅으로, 외부 상태(예: Zustand의 스토어와 같은 외부 데이터 소스)를 React의 리렌더링 시스템과 동기화하기 위해 설계되었습니다. 이 훅은 React 컴포넌트가 외부 상태를 구독하고, 상태 변화에 따라 효율적으로 리렌더링되도록 보장합니다. 특히, React의 Concurrent Rendering과 같은 최신 기능과 호환되며, 서버 사이드 렌더링(SSR) 및 클라이언트 사이드 렌더링(CSR)에서 일관된 동작을 제공합니다.

아래에서 `useSyncExternalStore`의 주요 개념, 사용법, 동작 원리, 그리고 Zustand에서의 활용 사례를 자세히 설명하겠습니다.

---

### 1. `useSyncExternalStore`란?

`useSyncExternalStore`는 외부 상태 관리 시스템(예: Zustand, Redux, 또는 브라우저 API 등)을 React의 상태 관리와 통합하기 위한 훅입니다. React 내부의 상태(`useState`, `useReducer`)와 달리, 외부 상태는 React의 관리 범위 밖에 존재하므로, 이를 React 컴포넌트와 동기화하려면 특별한 메커니즘이 필요합니다. `useSyncExternalStore`는 이 문제를 해결하며, 다음과 같은 상황에서 유용합니다:

- **외부 상태 관리 라이브러리와의 통합**: Zustand, Redux, MobX 등.
- **브라우저 API와의 동기화**: `window.localStorage`, `MediaQueryList` 등.
- **서버 사이드 렌더링(SSR)**: 클라이언트와 서버 간 상태 불일치 방지.

이 훅은 React의 Concurrent Rendering(동시 렌더링)과 호환되며, 불필요한 리렌더링을 방지하고, "tearing" 문제를 해결합니다. (Tearing은 비동기 렌더링 중 상태 불일치로 인해 UI가 일관성 없이 렌더링되는 문제입니다.)

---

### 2. `useSyncExternalStore`의 기본 구조

`useSyncExternalStore`는 세 가지 인자를 받으며, 다음과 같은 시그니처를 가집니다:

```typescript
function useSyncExternalStore<T>(
  subscribe: (onStoreChange: () => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T
): T;
```

#### 인자 설명
1. **`subscribe`**:
   - 외부 상태의 변경을 구독하는 함수입니다.
   - 콜백(`onStoreChange`)을 받아 상태 변경 시 이를 호출하고, 구독 해제 함수(예: `unsubscribe`)를 반환합니다.
   - 예: Zustand의 `store.subscribe`는 상태 변경 시 콜백을 실행합니다.

2. **`getSnapshot`**:
   - 현재 외부 상태의 스냅샷을 반환하는 함수입니다.
   - React는 이 함수를 호출하여 최신 상태를 가져오고, 상태 변화 여부를 판단합니다.
   - 예: Zustand의 `store.getState()`를 호출하여 스토어의 현재 상태를 반환합니다.

3. **`getServerSnapshot`** (선택적):
   - 서버 사이드 렌더링(SSR) 또는 정적 생성(SSG) 시 사용되는 스냅샷을 반환합니다.
   - 클라이언트와 서버 간 상태 일관성을 유지하기 위해 사용됩니다.
   - 제공되지 않으면 `getSnapshot`이 기본적으로 사용됩니다.

#### 반환값
- `useSyncExternalStore`는 `getSnapshot`에서 반환된 값을 React 컴포넌트에 제공합니다.
- 상태가 변경되면 React는 컴포넌트를 자동으로 리렌더링합니다.

---

### 3. 동작 원리

`useSyncExternalStore`는 외부 상태를 React의 리렌더링 시스템과 동기화하는 데 다음과 같은 과정을 거칩니다:

1. **구독 설정**:
   - `subscribe` 함수를 호출하여 외부 상태의 변경을 구독합니다.
   - 상태가 변경될 때마다 `onStoreChange` 콜백이 호출되어 React에게 리렌더링이 필요함을 알립니다.

2. **스냅샷 비교**:
   - React는 `getSnapshot`을 호출하여 최신 상태를 가져옵니다.
   - 이전 스냅샷과 비교하여 변경 여부를 확인합니다. 변경이 있으면 컴포넌트를 리렌더링합니다.
   - 얕은 비교(shallow equality) 또는 사용자 정의 비교 로직이 적용됩니다.

3. **리렌더링 최적화**:
   - `getSnapshot`이 동일한 값을 반환하면 리렌더링이 스킵됩니다.
   - 이는 성능 최적화에 중요하며, Zustand와 같은 라이브러리에서 셀렉터(selector)를 통해 필요한 상태만 추출하여 불필요한 리렌더링을 방지합니다.

4. **SSR 지원**:
   - `getServerSnapshot`을 통해 서버 렌더링 시 초기 상태를 제공하여 클라이언트와 서버 간 상태 불일치를 방지합니다.

5. **Tearing 방지**:
   - React의 Concurrent Rendering에서 여러 스레드 간 상태 불일치(tearing)를 방지하기 위해, `useSyncExternalStore`는 상태를 단일 소스에서 동기적으로 가져옵니다.

---

### 4. Zustand에서의 `useSyncExternalStore` 활용

Zustand는 `useSyncExternalStore`를 사용하여 스토어의 상태를 React 컴포넌트와 동기화합니다. [Zustand의 `src/react.ts` 파일](https://github.com/pmndrs/zustand/blob/main/src/react.ts)에서 `useStore` 훅의 구현을 통해 이를 확인할 수 있습니다. 아래는 관련 코드의 간략화된 예시입니다:

```typescript
import { useSyncExternalStore } from 'react';
import type { StoreApi } from './vanilla';

export function useStore<S, U>(
  store: StoreApi<S>,
  selector: (state: S) => U = (state) => state as any,
  equalityFn?: (a: U, b: U) => boolean
): U {
  return useSyncExternalStore(
    (onStoreChange) => store.subscribe(onStoreChange), // 구독 설정
    () => selector(store.getState()), // 클라이언트 스냅샷
    () => selector(store.getState()) // 서버 스냅샷
  );
}
```

#### 동작 과정
- **구독**: `store.subscribe`를 통해 Zustand 스토어의 상태 변경을 구독합니다. 상태가 변경되면 `onStoreChange` 콜백이 호출되어 React가 리렌더링을 트리거합니다.
- **스냅샷**: `selector(store.getState())`를 호출하여 스토어의 현재 상태를 선택적으로 추출합니다. 셀렉터는 필요한 상태만 반환하여 성능을 최적화합니다.
- **서버 렌더링**: 동일한 `selector(store.getState())`를 `getServerSnapshot`으로 사용하여 SSR 시 상태 일관성을 유지합니다.

#### 예시: Zustand 사용
```javascript
import create from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

function Counter() {
  const count = useStore((state) => state.count); // 셀렉터로 count만 추출
  const increment = useStore((state) => state.increment);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

여기서 `useStore`는 `useSyncExternalStore`를 내부적으로 사용하여 `count` 상태를 구독하고, 상태 변화 시 컴포넌트를 리렌더링합니다.

---

### 5. `useSyncExternalStore`의 장점과 한계

#### 장점
- **외부 상태 통합**: Zustand, Redux 같은 라이브러리나 브라우저 API와의 통합이 간단합니다.
- **Concurrent Rendering 호환**: React 18의 동시 렌더링과 호환되어 tearing 문제를 방지합니다.
- **SSR 지원**: `getServerSnapshot`을 통해 서버와 클라이언트 간 상태 일관성을 유지합니다.
- **성능 최적화**: 셀렉터를 통해 필요한 상태만 구독하여 불필요한 리렌더링을 최소화합니다.

#### 한계
- **복잡성**: `subscribe`와 `getSnapshot`을 직접 구현해야 하므로, 간단한 내부 상태에는 `useState`가 더 적합할 수 있습니다.
- **브라우저 호환성**: React 18 이상에서만 사용 가능하므로, React 17 이하에서는 폴리필(`use-sync-external-store` 패키지)을 사용해야 합니다.
- **추상화 필요**: Zustand와 같은 라이브러리가 이를 추상화하지 않았다면, 개발자가 직접 구독 로직을 작성해야 합니다.

---

### 6. 실제 사용 예시 (Zustand 외)
`useSyncExternalStore`는 Zustand 외에도 다양한 외부 상태를 관리하는 데 사용됩니다. 예를 들어, 브라우저의 `window.matchMedia`를 사용하여 미디어 쿼리 상태를 관리하는 예시입니다:

```javascript
import { useSyncExternalStore } from 'react';

function useMediaQuery(query) {
  return useSyncExternalStore(
    (callback) => {
      const mediaQuery = window.matchMedia(query);
      mediaQuery.addEventListener('change', callback);
      return () => mediaQuery.removeEventListener('change', callback);
    },
    () => window.matchMedia(query).matches,
    () => false // SSR 기본값
  );
}

function App() {
  const isDarkMode = useMediaQuery('(prefers-color-scheme: dark)');
  return <div>{isDarkMode ? 'Dark Mode' : 'Light Mode'}</div>;
}
```

이 예시는 `useSyncExternalStore`가 브라우저 API와 React를 연결하는 데 어떻게 사용되는지를 보여줍니다.

---

### 7. Zustand와 `useSyncExternalStore`의 관계
Zustand는 `useSyncExternalStore`를 사용하여 React와의 통합을 간소화합니다. 이는 Zustand가 프로바이더 없이 전역 상태를 관리할 수 있는 핵심 이유 중 하나입니다:
- **전역 스토어**: Zustand의 스토어는 JavaScript 모듈 스코프에 저장되어 전역적으로 접근 가능합니다.
- **훅 통합**: `useStore` 훅은 `useSyncExternalStore`를 활용해 스토어의 상태를 React 컴포넌트와 동기화합니다.
- **효율성**: 셀렉터를 통해 필요한 상태만 구독하므로, 성능 저하 없이 상태 관리가 가능합니다.

---

### 8. 결론
`useSyncExternalStore`는 React 18에서 외부 상태를 React의 리렌더링 시스템과 동기화하기 위해 도입된 강력한 훅입니다. Zustand는 이 훅을 활용해 프로바이더 없이 전역 상태를 관리하며, `useStore` 훅을 통해 스토어의 상태를 컴포넌트에 효율적으로 연결합니다. 이는 Zustand의 간결성과 성능 최적화의 핵심이며, React의 Concurrent Rendering 및 SSR 환경에서도 안정적으로 동작합니다.

---

### 추가 참고 자료
- **React 공식 문서**: [useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)
- **Zustand GitHub**: [src/react.ts](https://github.com/pmndrs/zustand/blob/main/src/react.ts)
- **Zustand v5 마이그레이션 가이드**: [Migrating to v5](https://github.com/pmndrs/zustand/blob/main/docs/migrations/migrating-to-v5.md)
- **React 18 Concurrent Rendering**: [React 18 Working Group](https://react.dev/blog/2022/03/29/react-v18)

궁금한 점이 더 있거나 특정 부분을 더 깊이 파고 싶다면 말씀해주세요! 😊