---
name: frontend-test
description: >
  이 프로젝트의 테스트 컨벤션에 맞게 테스트를 작성합니다. "테스트 만들어줘", "테스트 작성해줘",
  "테스트 추가해줘", "테스트 코드 작성", "unit test", "ui test", "테스트 짜줘" 와 같은 요청 시 반드시 사용합니다.
  컴포넌트, 훅, 유틸 함수 등 모든 테스트 작성 요청에 이 스킬을 사용하세요.
---

프로젝트의 테스트 가이드라인에 따라 테스트를 작성하는 역할입니다.

## 시작 전 필수 작업

1. **대상 파일 읽기** — 테스트할 파일을 직접 읽어 구현을 파악한다. 파일을 읽지 않고 테스트를 작성하지 않는다.
2. **의존성 파악** — `useTranslation`, `useStore`, `useNavigate` 등 외부 의존성이 있는지 확인한다.
3. **테스트 종류 판단** — 대상이 컴포넌트면 `.ui.test.tsx`, 훅/유틸이면 `.unit.test.ts`.

---

## 테스트 환경

- **Runner**: Vitest (`globals: true`, `environment: jsdom`)
- **라이브러리**: `@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom`
- **테스트 실행**: `pnpm test <파일명>`

### Import 패턴

```ts
import { render, screen, act } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, it, expect, vi } from 'vitest'
```

> `describe`, `it`, `expect`, `vi`는 `globals: true`로 전역 주입되어 import 없이도 사용 가능하지만,
> 기존 파일들과 일관성을 위해 명시적으로 import한다.

---

## 작성 워크플로우

### 1단계: 대상 분석

- props/파라미터, 반환값, 주요 동작, 엣지 케이스 파악
- 외부 의존성 목록 작성 (i18n, store, router, bridge API 등)

### 2단계: 테스트 시나리오 도출

- 기본 렌더링 / 기본 동작
- 주요 인터랙션 (클릭, 입력 등)
- props 변화에 따른 조건부 렌더링
- 엣지 케이스 (빈 값, disabled, loading 등)
- 에러 케이스 (필요 시)

### 3단계: 테스트 작성

- 파일 위치: 대상 파일과 **같은 폴더**에 배치 (코로케이션)
- `describe` — 컴포넌트명 또는 함수명
- `it` — "상황 또는 조건 → 기대 결과" 형식

### 4단계: 실행 확인

```bash
pnpm test <파일명>
```

---

## 쿼리 우선순위

접근성 기반 쿼리를 우선한다. 내려갈수록 최후 수단이다.

1. `getByRole` — 버튼, 링크, 입력 등 ARIA role 기반
2. `getByLabelText` — form 요소의 label
3. `getByPlaceholderText` — input placeholder
4. `getByText` — 화면에 보이는 텍스트
5. `getByTestId` — `data-testid` (최후 수단)

`className`·내부 state 기반 검증은 피한다. 단, `style` 값 검증(`toHaveStyle`)이나
DOM 구조가 유일한 방법일 때는 `container.querySelector`를 사용해도 된다.

---

## 의존성 처리 패턴

> `vi.mock()`은 반드시 import 문 **위**에 선언해야 한다 (Vitest가 자동으로 호이스팅).

### react-i18next

`useTranslation`을 사용하는 컴포넌트에 필수. `t(key)`는 key를 그대로 반환하므로
테스트에서 번역 키로 검증한다.

```ts
vi.mock('react-i18next', () => ({
  useTranslation: () => ({ t: (key: string) => key }),
  Trans: ({ children }: any) => <>{children}</>
}))
```

### motion/react (framer-motion)

애니메이션 컴포넌트를 일반 HTML 요소로 대체한다.

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

`vi.fn()`으로 모킹 후 `mockReturnValue`로 원하는 상태를 주입한다.

```ts
const mockUseXxx = vi.fn()

vi.mock('@common/hooks/useXxx', () => ({
  useXxx: (...args: any[]) => mockUseXxx(...args)
}))

// 각 테스트에서:
mockUseXxx.mockReturnValue({ loading: true, data: null })
```

### Zustand store (`@store/index`)

selector 함수를 실행해 필요한 값만 반환한다.

```ts
vi.mock('@store/index', () => ({
  useStore: (selector: any) =>
    selector({
      favoriteFolders: [],
      indexPath: ['/path', 'set'],
      // 필요한 state만 추가
    }),
  useBoundStore: (selector: any) =>
    selector({ favoriteFolders: [] })
}))
```

### React Router

`useLocation`, `useNavigate`, `Link` 등을 사용하는 컴포넌트는 `MemoryRouter`로 감싼다.

```tsx
import { MemoryRouter } from 'react-router'

render(
  <MemoryRouter>
    <MyComponent />
  </MemoryRouter>
)
```

### overlay-kit

모달 open 여부만 검증할 때는 spy만으로 충분하다.

```ts
vi.mock('overlay-kit', () => ({
  overlay: { open: vi.fn() }
}))
```

### TanStack Query 훅

컴포넌트 단위 테스트에서는 훅 자체를 모킹하는 것이 낫다.

```ts
vi.mock('@app/home/hooks/useIndexingStatus', () => ({
  useIndexingStatus: () => ({
    data: { isProcessing: false, lastCompletedAt: null }
  })
}))
```

### Portal 컴포넌트 (`createPortal`)

`createPortal`은 jsdom에서 `document.body`에 렌더링되며, RTL이 자동으로 조회한다. 별도 처리 불필요.

---

## 타이머 테스트

`setTimeout`/`setInterval`을 사용하는 컴포넌트는 fake timer를 사용한다.

```ts
beforeEach(() => { vi.useFakeTimers() })
afterEach(() => { vi.useRealTimers() })

it('duration 후 onClose를 호출한다', () => {
  const handleClose = vi.fn()
  render(<Toast isOpen onClose={handleClose} duration={2000} />)
  act(() => { vi.advanceTimersByTime(2000) })
  expect(handleClose).toHaveBeenCalledOnce()
})
```

---

## 복잡한 서브컴포넌트 처리

상위 컴포넌트 테스트 시 하위 컴포넌트가 무거운 의존성을 갖는다면 모킹한다.

```ts
vi.mock('@common/components/modal/settings/contents/AISearchSettings', () => ({
  AISearchSettings: () => <div data-testid="ai-search-settings" />
}))
```

---

## 테스트 건너뛰기 기준

다음 경우에는 테스트를 무리하게 작성하지 않는다:

- Bridge API 호출 + TanStack Query + store가 **모두** 얽혀 있는 컴포넌트
- 복잡한 비동기 플로우가 핵심 로직인 컴포넌트 (예: `SearchBar`, `FilterPanel`)
- 테스트를 위한 모킹 코드가 컴포넌트 코드보다 많아지는 경우

이런 경우는 연동 테스트나 E2E로 커버하는 것이 적합하다.

---

## 중요 원칙

- **사용자 관점 검증**: 화면에 보이는 텍스트, 역할, 상태 기준으로 검증한다.
- **비동기는 항상 await**: `userEvent`는 `setup()` 후 `await`로 사용한다.
- **React Compiler 환경**: `useMemo`/`useCallback`/`memo` 수동 추가를 지양한다.
- **vi.mock 호이스팅**: `vi.mock()`은 파일 최상단 import 이전에 선언한다.
