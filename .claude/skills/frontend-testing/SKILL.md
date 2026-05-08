---
name: frontend-testing
description: Vitest 기반 프론트엔드 테스트 작성 스킬. 컴포넌트 UI 테스트(React Testing Library)와 유닛 테스트(함수·훅)를 구분해 작성한다. "테스트 작성해줘", "test 추가", "컴포넌트 테스트", "훅 테스트", "unit test", "UI test", "테스트 커버리지", "spec 작성", "테스트 파일 만들어줘" 등의 요청에 반드시 사용한다. 테스트 관련 작업이라면 1%의 가능성만 있어도 이 스킬을 먼저 실행해라.
---

# Vitest 프론트엔드 테스트 스킬

## 0단계: 프로젝트 환경 파악 (항상 먼저)

테스트를 작성하기 전에 아래 항목을 확인한다. 이미 같은 세션에서 파악했다면 건너뛴다.

### 확인할 항목

```bash
# vitest 설정 찾기
find . -name "vitest.config*" -not -path "*/node_modules/*"

# 테스트 setup 파일 찾기
find . -name "setupTests*" -o -name "setup.ts" -o -name "test-utils*" | grep -v node_modules

# React Compiler 설정 확인
grep -r "reactCompiler\|babel-plugin-react-compiler\|react-compiler" --include="*.json" --include="*.ts" --include="*.js" --include="*.mjs" -l | grep -v node_modules | head -5

# 기존 테스트 파일 샘플 확인 (패턴 파악용)
find . -name "*.test.tsx" -o -name "*.test.ts" -o -name "*.spec.tsx" | grep -v node_modules | head -5
```

### 파악 후 확인할 것

- **커스텀 render 함수**: `test-utils.tsx`나 `renderWithProviders` 같은 래퍼가 있는지 → 있으면 반드시 사용
- **React Compiler 적용 여부**: `reactCompiler: true` 또는 babel plugin 존재 → 있으면 메모이제이션 테스트 불필요
- **Provider 목록**: i18n, store, router, theme 등 wrapping 필요한 Provider
- **전역 mock 설정**: `setupFiles`에 이미 mock된 것 (window, fetch, localStorage 등)
- **테스트 커맨드**: `package.json`의 test 스크립트

---

## 기존 테스트 환경이 있을 때

vitest 설정 + 기존 테스트 파일이 발견되면 아래 순서로 환경을 읽고, 그 패턴을 그대로 따라 케이스를 작성한다. 이 단계를 건너뛰고 임의의 패턴을 쓰지 않는다.

### 1. setup 파일 읽기

```bash
# vitest.config의 setupFiles 항목 확인
grep -A5 "setupFiles" vitest.config.*

# setup 파일 실제 내용 읽기 (전역 mock, matchers, 환경 설정 파악)
cat <setupFiles 경로>
```

확인할 것:

- `@testing-library/jest-dom` matchers가 import돼 있는지 (`toBeInTheDocument` 등 사용 가능 여부)
- 전역으로 mock된 모듈 목록 (이미 mock된 건 테스트에서 다시 mock하지 않는다)
- `server.listen()` / MSW 설정 여부

### 2. 커스텀 render/test-utils 읽기

test-utils 파일이 있으면 반드시 읽는다.

```bash
cat <test-utils 경로>
```

확인할 것:

- export되는 render 함수 이름과 시그니처
- 기본으로 wrapping되는 Provider 목록 (i18n, store, router, theme 등)
- 추가로 export되는 헬퍼 (예: `renderWithUser`, `createMockStore`)

→ 이 커스텀 render를 `@testing-library/react`의 기본 render 대신 사용한다.

### 3. 기존 테스트 파일 샘플 읽기

같은 디렉터리나 인접한 곳에 기존 테스트가 있으면 1~2개 읽는다.

```bash
# 대상 파일 근처 테스트 찾기
find <대상_파일_디렉터리> -name "*.test.*" | head -3
```

확인할 것:

- `describe` / `it` 문구 스타일 (한국어/영어, 어미 패턴)
- mock 선언 방식 (`vi.mock` inline vs `__mocks__` 폴더)
- `beforeEach` / `afterEach` 공통 패턴
- 프로젝트 특유의 헬퍼나 픽스처 사용 여부

→ 새 테스트 파일은 이 스타일을 그대로 따른다. 스타일이 다르면 사용자에게 확인한다.

### 4. 케이스 작성 후 실행

테스트를 작성하고 나서 반드시 실행해서 통과를 확인한다.

```bash
# 해당 파일만 실행
pnpm vitest run <테스트파일경로>

# watch 모드로 확인
pnpm vitest <테스트파일경로>
```

실패하면 에러 메시지를 읽고 고친다. "작성 완료"는 테스트가 초록불일 때만 선언한다.

---

## 테스트 작성 여부 판단

### 테스트를 작성하지 않는 경우

아래에 해당하면 테스트를 작성하지 않는다. 사용자에게 "이 컴포넌트/함수는 테스트 작성 기준에 해당하지 않습니다"라고 명확히 알린다.

- **정적 컴포넌트**: props를 받지 않고 항상 같은 UI를 렌더링 (예: 로고, 구분선, 정적 배너)
- **순수 래퍼**: 다른 컴포넌트를 `className`만 추가해 감싸는 단순 래퍼
- **trivial 유틸**: `const add = (a, b) => a + b` 수준의 한 줄 함수
- **타입/상수 파일**: `.types.ts`, `constants.ts`만 export
- **자동 생성 코드**: codegen, scaffold로 만들어진 boilerplate

테스트의 가치는 "미래의 회귀를 잡는 것"이다. 변하거나 복잡해질 가능성이 없는 코드는 테스트가 유지비용이 된다.

---

## 테스트 유형 구분

### UI 테스트 (컴포넌트)

> 사용자가 보고 상호작용하는 것을 검증한다.

**대상**: React 컴포넌트 (`*.tsx`)  
**파일명**: `ComponentName.test.tsx`  
**도구**: `@testing-library/react`, `@testing-library/user-event`

### 유닛 테스트 (함수·훅)

> 입력 → 출력의 로직을 검증한다.

**대상**: 유틸 함수 (`*.ts`), 커스텀 훅 (`use*.ts`)  
**파일명**: `functionName.test.ts` / `useHookName.test.ts`  
**도구**: `vitest`, `@testing-library/react` (`renderHook`)

---

## 작성 워크플로우

### 1단계: 대상 분석

- props / 파라미터, 반환값, 주요 동작 파악
- 외부 의존성 목록 작성 (i18n, store, router, bridge API, fetch 등)
- React Compiler 환경이면 useMemo/useCallback/memo 테스트 제외

### 2단계: 테스트 시나리오 도출

| 시나리오 | 설명 |
| -------- | ---- |
| 기본 렌더링 | 필수 props만 주고 주요 요소가 화면에 보이는지 |
| 주요 인터랙션 | 클릭, 입력, 선택 등 사용자 동작 후 변화 |
| 조건부 렌더링 | props/상태 변화에 따른 UI 분기 |
| 엣지 케이스 | 빈 값, disabled, loading, 최대/최소 값 |
| 에러 케이스 | 필요 시 (API 실패, validation 오류) |

시나리오가 1~2개뿐이라면 테스트 작성 여부를 다시 판단한다.

### 3단계: 테스트 작성

**파일 위치**: 대상 파일과 **같은 폴더**에 co-locate

```text
src/app/user/components/
├── UserCard.tsx
└── UserCard.test.tsx     ← 여기
```

---

## UI 테스트 패턴

### 기본 구조

```typescript
import { render, screen } from '@testing-library/react'
// 커스텀 render가 있으면 위 대신 아래 사용
import { render, screen } from '@/test-utils'
import userEvent from '@testing-library/user-event'
import { ComponentName } from './ComponentName'

describe('ComponentName', () => {
  it('기본 props로 주요 텍스트를 렌더링한다', () => {
    render(<ComponentName label="저장" />)
    expect(screen.getByText('저장')).toBeInTheDocument()
  })

  it('버튼 클릭 시 onSave 핸들러를 호출한다', async () => {
    const onSave = vi.fn()
    const user = userEvent.setup()

    render(<ComponentName onSave={onSave} />)
    await user.click(screen.getByRole('button', { name: '저장' }))

    expect(onSave).toHaveBeenCalledOnce()
  })
})
```

### 쿼리 우선순위

```text
1순위: getByRole         — 접근성 역할 기준 (button, textbox, heading 등)
2순위: getByLabelText    — 폼 입력
3순위: getByPlaceholderText
4순위: getByText         — 화면에 보이는 텍스트
5순위: getByDisplayValue — 현재 값
6순위: getByAltText      — 이미지
7순위: getByTitle
최후: getByTestId        — data-testid (다른 방법이 없을 때만)
```

### 비동기 처리

```typescript
// userEvent는 항상 setup() 후 await
const user = userEvent.setup()
await user.click(element)
await user.type(input, 'hello')

// 비동기 상태 변화 대기
await screen.findByText('로딩 완료')  // findBy = waitFor + getBy
await waitFor(() => expect(mockFn).toHaveBeenCalled())
```

### Mock 작성

```typescript
// vi.mock은 파일 최상단 import 이전에 선언 (호이스팅)
vi.mock('@/lib/api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: 1, name: '홍길동' }),
}))

vi.mock('next/navigation', () => ({
  useRouter: () => ({ push: vi.fn(), replace: vi.fn() }),
  usePathname: () => '/test',
}))

// 각 테스트 후 mock 초기화
afterEach(() => {
  vi.clearAllMocks()
})
```

### Provider가 필요한 경우

```typescript
// 커스텀 render가 없을 때 직접 wrapping
const renderWithProviders = (ui: React.ReactElement) =>
  render(
    <QueryClientProvider client={new QueryClient()}>
      <I18nProvider>
        {ui}
      </I18nProvider>
    </QueryClientProvider>
  )
```

---

## 유닛 테스트 패턴

### 유틸 함수

```typescript
import { describe, it, expect } from 'vitest'
import { formatPrice } from './formatPrice'

describe('formatPrice', () => {
  it('숫자를 원화 형식으로 변환한다', () => {
    expect(formatPrice(1000)).toBe('1,000원')
  })

  it('0이면 "무료"를 반환한다', () => {
    expect(formatPrice(0)).toBe('무료')
  })

  it('음수 입력 시 에러를 throw한다', () => {
    expect(() => formatPrice(-1)).toThrow('유효하지 않은 가격')
  })
})
```

### 커스텀 훅

```typescript
import { renderHook, act } from '@testing-library/react'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('초기값이 0이다', () => {
    const { result } = renderHook(() => useCounter())
    expect(result.current.count).toBe(0)
  })

  it('increment 호출 시 count가 1 증가한다', () => {
    const { result } = renderHook(() => useCounter())
    act(() => result.current.increment())
    expect(result.current.count).toBe(1)
  })
})
```

---

## 중요 원칙

### 사용자 관점 검증

구현 세부사항(state 변수명, CSS 클래스)이 아닌 화면에 보이는 것, 역할, 상태를 기준으로 검증한다.

```typescript
// ❌ 구현 세부사항
expect(wrapper.find('.btn-primary')).toHaveLength(1)
expect(component.state.isOpen).toBe(true)

// ✅ 사용자 관점
expect(screen.getByRole('button', { name: '확인' })).toBeEnabled()
expect(screen.getByRole('dialog')).toBeVisible()
```

### React Compiler 환경

React Compiler가 활성화된 프로젝트에서는:

- `useMemo`, `useCallback`, `memo`의 존재 여부를 테스트하지 않는다
- 컴파일러가 자동으로 최적화하므로 수동 메모이제이션 추가를 지양한다
- 렌더링 횟수(render count)보다 실제 UI 동작에 집중한다

### vi.mock 호이스팅

```typescript
// ✅ 올바른 위치: import 이전 (최상단)
vi.mock('@/lib/api')

import { render } from '@testing-library/react'
import { MyComponent } from './MyComponent'
```

### 테스트 격리

```typescript
beforeEach(() => {
  vi.clearAllMocks()  // 각 테스트 전 mock 초기화
})
// 또는
afterEach(() => {
  cleanup()  // RTL 자동 cleanup (보통 자동 설정)
})
```

---

## it 문구 작성 가이드

"상황 또는 조건 → 기대 결과" 형식으로 작성한다.

```text
✅ 'disabled prop이 true일 때 버튼을 클릭할 수 없다'
✅ '검색어가 비어있으면 검색 버튼이 비활성화된다'
✅ 'API 호출 중에 로딩 스피너를 표시한다'
✅ '유효하지 않은 이메일 입력 시 에러 메시지를 보여준다'

❌ 'works correctly'
❌ 'renders'
❌ 'test button'
```
