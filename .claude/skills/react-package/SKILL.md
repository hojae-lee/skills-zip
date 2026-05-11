---
name: react-package
description: >
  독립 React/TypeScript 프로젝트의 모든 작업에 반드시 사용한다. Next.js가 아닌 순수 React
  프로젝트에서 컴포넌트 작성, 훅 작성, 성능 최적화, 상태 관리, 테스트 작성 요청 시 이 스킬을
  사용한다. "컴포넌트 만들어줘", "훅 작성해줘", "성능 개선해줘", "리렌더링 줄여줘", "테스트 짜줘",
  "Zustand 써줘", "타입 잡아줘", "번들 최적화", "코드 스플리팅", "React 코드 리뷰해줘" 와 같은
  모든 React 프로젝트 작업에 이 스킬을 사용한다. Next.js App Router 환경이라면
  nextjs-app-router 스킬을 사용한다.
---

# Frontend React 개발 가이드

## TypeScript 규칙

**타입 선언**

- 컴포넌트 Props → `type` 사용
- 일반 객체/데이터 구조 → `interface` 사용
- `any` 사용 금지

**Type Import**

- 타입만 import할 때는 반드시 `import type` 사용

```ts
// ✅
import type { User } from "./types";
import { api, type ApiResponse } from "./api";

// ❌
import { User } from "./types";
```

---

## 컴포넌트 패턴

```tsx
// ✅ 페이지/독립 컴포넌트: arrow function + export default
type Props = {
  title: string;
  count: number;
};

const MyComponent = ({ title, count }: Props) => {
  return (
    <div>
      {title}: {count}
    </div>
  );
};

export default MyComponent;

// ✅ 공유/공통 컴포넌트: named export
export const SharedButton = ({ label }: { label: string }) => (
  <button>{label}</button>
);
```

컴포넌트를 컴포넌트 내부에 정의하지 않는다 — 리렌더링마다 새 참조가 생성된다.

---

## Import 규칙

- **같은 모듈/폴더 내**: 상대경로 (`./hooks/useX`, `../components/Button`)
- **다른 모듈**: 프로젝트 절대경로 alias 사용 (`@common/...`, `@api/...` 등)
- 미사용 import는 즉시 제거

---

## 성능 최적화

### React Compiler

React Compiler는 컴포넌트와 훅을 자동으로 메모이제이션한다. Compiler가 활성화된 프로젝트에서는 수동 최적화가 불필요하다.

**수동 작성 금지** (Compiler가 자동 처리):

```tsx
// ❌ Compiler 프로젝트에서는 이 코드들이 불필요
const memoValue = useMemo(() => compute(a, b), [a, b]);
const stableCallback = useCallback(() => doSomething(x), [x]);
const MemoComponent = memo(({ value }) => <div>{value}</div>);

// ✅ 그냥 작성하면 Compiler가 최적화
const memoValue = compute(a, b);
const stableCallback = () => doSomething(x);
const MyComponent = ({ value }) => <div>{value}</div>;
```

**Compiler 동작 조건 — Rules of React 준수 필수:**
Compiler는 Rules of React를 지키는 코드만 최적화한다. 위반 시 해당 함수는 최적화에서 제외된다.

```tsx
// ❌ Rules of React 위반 — Compiler 최적화 불가
const getUser = () => {
  if (condition) {
    return useUserStore((s) => s.user); // 조건부 훅 호출
  }
};

// ❌ 렌더링 중 외부 변수 직접 변이
let count = 0;
const Counter = () => {
  count++; // 렌더링 중 side effect
  return <div>{count}</div>;
};

// ✅ 올바른 패턴
const Counter = () => {
  const [count, setCount] = useState(0);
  return <div onClick={() => setCount((c) => c + 1)}>{count}</div>;
};
```

**Compiler에서 제외해야 할 경우** (`"use no memo"` 지시어):

```tsx
const MyComponent = () => {
  "use no memo"; // 함수 본문 첫 줄에 선언, 백틱 불가
  // TODO: 제외 이유와 제거 조건 주석 명시
  return <div>...</div>;
};

// 모듈 전체 제외 시 파일 최상단에 선언
("use no memo");
```

**Compiler 적용 확인**: React DevTools의 컴포넌트 이름 앞에 `Memo(...)` 표시가 붙으면 Compiler가 적용된 것이다.

### 코드 스플리팅

무거운 컴포넌트(차트, 에디터, 복잡한 모달)는 지연 로딩:

```tsx
const HeavyChart = React.lazy(() => import("./HeavyChart"));

const Parent = () => (
  <Suspense fallback={<Spinner />}>
    <HeavyChart />
  </Suspense>
);
```

### 리스트 key

안정적인 고유 값 사용. 순서가 고정된 경우에만 index 허용.

**상세 성능 최적화 규칙** → `references/performance.md` 참조

---

## 상태 관리 (Zustand)

전역 상태가 필요한 경우에만 사용한다. 서버 상태(API 데이터)는 TanStack Query가 담당하므로 Zustand는 순수 클라이언트 상태(UI 상태, 사용자 세션 등)에만 쓴다.

도메인별로 store를 분리해서 만든다. 하나의 거대한 store는 만들지 않는다.

```ts
// store/useUserStore.ts
import { create } from "zustand";
import { devtools } from "zustand/middleware";

interface UserStore {
  user: User | null;
  isLoggedIn: boolean;
  setUser: (user: User) => void;
  clearUser: () => void;
}

export const useUserStore = create<UserStore>()(
  devtools(
    (set) => ({
      user: null,
      isLoggedIn: false,
      setUser: (user) => set({ user, isLoggedIn: true }, false, "setUser"),
      clearUser: () =>
        set({ user: null, isLoggedIn: false }, false, "clearUser"),
    }),
    { name: "UserStore" },
  ),
);
```

컴포넌트에서는 필요한 상태만 selector로 선택해 불필요한 리렌더링을 막는다.

```tsx
// ❌ store 전체 구독 — 모든 상태 변경 시 리렌더링
const store = useUserStore();

// ✅ 필요한 값만 선택
const user = useUserStore((s) => s.user);
const setUser = useUserStore((s) => s.setUser);

// ✅ 여러 값이 필요하면 useShallow
import { useShallow } from "zustand/react/shallow";
const { user, isLoggedIn } = useUserStore(
  useShallow((s) => ({ user: s.user, isLoggedIn: s.isLoggedIn })),
);
```

---

## 데이터 페칭 (TanStack Query)

- **컴포넌트에서 API 함수 직접 호출 금지** → TanStack Query를 통해 처리
- **컴포넌트에서 `useQuery` 직접 호출 금지** → 도메인별 커스텀 훅으로 래핑

```tsx
// ❌
const MyComponent = () => {
  const data = await fetchUser(id); // 직접 호출 금지
  const { data } = useQuery({ queryKey: ["user"], queryFn: fetchUser }); // 직접 사용 금지
};

// ✅ 커스텀 훅으로 래핑
const useUser = (id: string) => useQuery(userQueries.getUser(id));
```

**Query Factory 패턴 및 상세 가이드** → `references/tanstack-query.md` 참조

---

## 테스트

- 컴포넌트: `.ui.test.tsx`, 훅/유틸: `.unit.test.ts`
- 대상 파일과 같은 폴더에 배치 (코로케이션)

**테스트 컨벤션 상세** → `references/testing.md` 참조

---

## 코드 품질

작업 완료 후 반드시 lint/format 실행:

```bash
pnpm run lint:fix
pnpm run format
```
