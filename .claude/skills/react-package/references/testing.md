# React 테스트 가이드

## 환경

- **Runner**: Vitest (`globals: true`, `environment: jsdom`)
- **라이브러리**: `@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom`

---

## 시작 전 필수 작업

1. **대상 파일 읽기** — 테스트할 파일을 직접 읽어 구현을 파악한다
2. **의존성 파악** — 외부 의존성(i18n, store, router 등) 목록 작성
3. **테스트 종류 판단** — 컴포넌트면 `.ui.test.tsx`, 훅/유틸이면 `.unit.test.ts`

---

## 파일 구조

- **파일 위치**: 테스트 대상과 **같은 폴더** (코로케이션)
- **파일명**: `.ui.test.tsx` (컴포넌트), `.unit.test.ts` (훅/유틸)

```
components/
├── Button.tsx
├── Button.ui.test.tsx    ← 컴포넌트 테스트
hooks/
├── useCounter.ts
├── useCounter.unit.test.ts  ← 훅 테스트
```

---

## Import 패턴

```ts
import { render, screen, act } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";
```

> `describe`, `it`, `expect`, `vi`는 `globals: true`로 전역 주입되어 import 없이도 사용 가능하지만,
> 일관성을 위해 명시적으로 import한다.

---

## 테스트 시나리오 구조

```ts
describe('ComponentName', () => {
  it('기본 렌더링을 수행한다', () => { ... })
  it('버튼 클릭 시 onSubmit을 호출한다', async () => { ... })
  it('disabled 상태에서 클릭이 동작하지 않는다', async () => { ... })
  it('데이터가 없을 때 빈 상태를 표시한다', () => { ... })
})
```

`it` 설명: "조건/상황 → 기대 결과" 형식으로 작성한다.

---

## 쿼리 우선순위

접근성 기반 쿼리를 우선한다. 아래로 갈수록 최후 수단.

1. `getByRole` — ARIA role 기반 (버튼, 링크, 입력 등)
2. `getByLabelText` — form label
3. `getByPlaceholderText` — input placeholder
4. `getByText` — 화면에 보이는 텍스트
5. `getByTestId` — `data-testid` (최후 수단)

`className`·내부 state 기반 검증은 피한다. DOM 구조가 유일한 방법일 때는 `container.querySelector` 허용.

---

## 의존성 처리 패턴

> `vi.mock()`은 반드시 import 문 **위**에 선언 (Vitest가 자동으로 호이스팅).

### react-i18next

```ts
vi.mock('react-i18next', () => ({
  useTranslation: () => ({ t: (key: string) => key }),
  Trans: ({ children }: any) => <>{children}</>
}))
```

`t(key)`는 key를 그대로 반환하므로 테스트에서 번역 키로 검증한다.

### motion/react (framer-motion)

```ts
vi.mock('motion/react', () => ({
  motion: {
    div: ({ children, ...props }: any) => <div {...props}>{children}</div>,
    p: ({ children, ...props }: any) => <p {...props}>{children}</p>,
    span: ({ children, ...props }: any) => <span {...props}>{children}</span>
  }
}))
```

### 커스텀 훅

```ts
const mockUseXxx = vi.fn();

vi.mock("./hooks/useXxx", () => ({
  useXxx: (...args: any[]) => mockUseXxx(...args),
}));

// 각 테스트에서 원하는 상태 주입
mockUseXxx.mockReturnValue({ loading: true, data: null });
```

### Zustand store

selector 함수를 실행해 필요한 값만 반환한다.

```ts
vi.mock("../store", () => ({
  useStore: (selector: any) =>
    selector({
      items: [],
      isLoading: false,
      setItems: vi.fn(),
    }),
}));
```

### React Router

`useLocation`, `useNavigate`, `Link` 등을 사용하는 컴포넌트는 `MemoryRouter`로 감싼다.

```tsx
import { MemoryRouter } from "react-router";

render(
  <MemoryRouter>
    <MyComponent />
  </MemoryRouter>,
);
```

### TanStack Query 훅

컴포넌트 단위 테스트에서는 훅 자체를 모킹한다.

```ts
vi.mock("./hooks/useGetPost", () => ({
  useGetPost: () => ({
    data: { id: "1", title: "Test Post" },
    isPending: false,
    isError: false,
  }),
}));
```

### 외부 API/서비스

실제 호출이 일어나지 않도록 resolved Promise를 반환한다.

```ts
vi.mock("../api/postApi", () => ({
  createPost: vi.fn().mockResolvedValue({ id: "1", title: "New Post" }),
}));
```

### Portal 컴포넌트 (`createPortal`)

jsdom에서 `document.body`에 렌더링된다. RTL이 자동으로 조회하므로 별도 처리 불필요.

---

## 복잡한 서브컴포넌트 처리

상위 컴포넌트 테스트 시 하위 컴포넌트가 무거운 의존성을 갖는다면 모킹한다.

```ts
vi.mock('./HeavySubComponent', () => ({
  HeavySubComponent: () => <div data-testid="heavy-sub" />
}))
```

---

## 타이머 테스트

`setTimeout`/`setInterval`을 사용하는 컴포넌트:

```ts
beforeEach(() => { vi.useFakeTimers() })
afterEach(() => { vi.useRealTimers() })

it('2초 후 onClose를 호출한다', () => {
  const handleClose = vi.fn()
  render(<Toast isOpen onClose={handleClose} duration={2000} />)
  act(() => { vi.advanceTimersByTime(2000) })
  expect(handleClose).toHaveBeenCalledOnce()
})
```

---

## 중요 원칙

- **사용자 관점 검증**: 화면에 보이는 텍스트, role, 상태 기준으로 검증한다
- **비동기는 항상 await**: `userEvent`는 `setup()` 후 `await`로 사용한다
- **React Compiler 환경**: `useMemo`/`useCallback`/`memo` 수동 추가를 지양한다
- **vi.mock 호이스팅**: `vi.mock()`은 파일 최상단 import 이전에 선언한다

---

## 테스트 건너뛰기 기준

다음 경우에는 무리하게 단위 테스트를 작성하지 않는다:

- 여러 외부 시스템(API + store + router 등)이 **모두** 얽혀 있는 컴포넌트
- 복잡한 비동기 플로우가 핵심 로직인 경우
- 모킹 코드가 컴포넌트 코드보다 많아지는 경우

이런 경우는 통합 테스트나 E2E로 커버하는 것이 적합하다.

---

## 실행

```bash
# 단일 파일
pnpm test Button.ui.test.tsx

# 감시 모드
pnpm test --watch
```
