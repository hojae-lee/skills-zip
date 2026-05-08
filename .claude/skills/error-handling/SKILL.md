---
name: error-handling
description: 에러 처리 패턴 스킬. API 에러(status code 기반)와 UI 에러(Error Boundary)를 명확하게 분리해서 처리한다. "에러 처리해줘", "에러 핸들링", "Error Boundary", "에러 페이지", "toast 알림", "API 에러", "500 에러", "401 처리", "에러 메시지", "fallback UI", "에러 상태 관리" 등의 요청에 반드시 사용한다. 에러 관련 작업이면 이 스킬을 먼저 확인해라.
---

# 에러 처리 스킬

## 핵심 원칙: 에러 책임 분리

| 에러 유형 | 누가 결정 | 처리 방식 |
| --------- | --------- | --------- |
| **API 에러** | 서버는 status code만, 메시지는 클라이언트가 결정 | status code 기반 분기 |
| **UI 에러** | 예측 불가능한 런타임 예외 | Error Boundary로 격리 |

서버에서 에러 메시지 문자열을 직접 내려주더라도, UI에 그대로 노출하지 않는다. 사용자에게 보여줄 메시지는 클라이언트에서 status code 기반으로 결정한다. 서버 메시지는 로깅용으로만 쓴다.

---

## 1. API 에러 처리

`api-integration` 스킬의 `isApiError()` 타입 가드와 연계해서 사용한다.

### Status Code → UI 행동 매핑

```typescript
// src/lib/error/apiErrorHandler.ts
import { isApiError } from '@/lib/api/client'

export type ApiErrorAction =
  | { type: 'redirect'; to: string }
  | { type: 'toast'; message: string }
  | { type: 'inline'; message: string }
  | { type: 'empty' }
  | { type: 'retry' }

export const resolveApiError = (error: unknown): ApiErrorAction => {
  if (!isApiError(error)) return { type: 'toast', message: ERROR_MESSAGES.unknown }

  if (error.status === 401) return { type: 'redirect', to: '/login' }
  if (error.status === 403) return { type: 'inline', message: ERROR_MESSAGES.forbidden }
  if (error.status === 404) return { type: 'empty' }
  if (error.status === 409) return { type: 'inline', message: ERROR_MESSAGES.conflict }
  if (error.status === 422) return { type: 'inline', message: ERROR_MESSAGES.validation }
  if (error.status >= 500) return { type: 'retry', }

  return { type: 'toast', message: ERROR_MESSAGES.unknown }
}
```

### 에러 메시지 상수 (하드코딩 금지)

에러 메시지를 코드 곳곳에 직접 쓰지 않는다. 한 파일에 모아서 관리한다.

```typescript
// src/lib/error/messages.ts
export const ERROR_MESSAGES = {
  unknown:    '일시적인 오류가 발생했습니다. 잠시 후 다시 시도해주세요.',
  forbidden:  '접근 권한이 없습니다.',
  conflict:   '이미 존재하는 데이터입니다.',
  validation: '입력 값을 확인해주세요.',
  network:    '네트워크 연결을 확인해주세요.',
} as const

// i18n을 사용하는 프로젝트라면 메시지 대신 i18n 키를 상수로 관리한다
export const ERROR_KEYS = {
  unknown:    'error.unknown',
  forbidden:  'error.forbidden',
} as const
```

### TanStack Query와 연계

```typescript
const UserList = () => {
  const { data, error, isLoading, refetch } = useQuery(userQueries.list())
  const router = useRouter()

  if (isLoading) return <Skeleton />

  if (error) {
    const action = resolveApiError(error)

    if (action.type === 'redirect') {
      router.push(action.to)
      return null
    }
    if (action.type === 'empty') return <EmptyState />
    if (action.type === 'retry') return <RetryState onRetry={refetch} />
    if (action.type === 'inline') return <ErrorMessage message={action.message} />
  }

  return <List items={data} />
}
```

### Mutation 에러 (toast 패턴)

mutation 에러는 사용자 동작에 대한 피드백이므로 대부분 toast로 처리한다.

```typescript
const useCreateUser = () => {
  const { toast } = useToast()

  return useMutation({
    mutationFn: (data: CreateUserInput) =>
      apiClient.request<User>('/users', { method: 'POST', body: JSON.stringify(data) }),
    onError: (error) => {
      const action = resolveApiError(error)
      if (action.type === 'toast' || action.type === 'inline') {
        toast({ variant: 'destructive', description: action.message })
      }
    },
  })
}
```

---

## 2. UI 에러 처리 (Error Boundary)

렌더링 중 발생하는 예측 불가능한 JS 예외를 잡아서 앱 전체가 흰 화면이 되는 것을 방지한다.

### Error Boundary 배치 전략

```text
app/
├── layout.tsx          ← Root Boundary: 최후 방어선
├── error.tsx           ← 페이지 레벨 (Next.js App Router 자동)
├── [domain]/
│   ├── error.tsx       ← 도메인 레벨
│   └── page.tsx
└── components/
    └── Widget.tsx      ← 독립 위젯은 별도 Boundary 고려
```

| 레벨 | 언제 쓰나 | fallback |
| ---- | --------- | -------- |
| Root (layout) | 앱 전체 크래시 방지 | 단순 에러 페이지 |
| 페이지 (error.tsx) | 페이지 단위 격리 | 재시도 버튼 + 안내 |
| 섹션 | 독립적으로 동작하는 위젯, 사이드바 | 섹션만 에러 표시 |
| 컴포넌트 | 드물게, 정말 독립적인 경우만 | 최소 fallback |

**컴포넌트 단위 Error Boundary는 남발하지 않는다.** 에러가 전파되도록 두는 것이 디버깅에 유리하다. 독립적인 기능 단위일 때만 격리한다.

### Next.js App Router

`error.tsx`를 두면 자동으로 Error Boundary가 된다.

```typescript
// app/[domain]/error.tsx
'use client'

const DomainError = ({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) => (
  <div role="alert">
    <h2>문제가 발생했습니다</h2>
    <p>{ERROR_MESSAGES.unknown}</p>
    <button onClick={reset}>다시 시도</button>
  </div>
)

export default DomainError
```

### React (App Router 없는 경우)

`react-error-boundary` 라이브러리를 사용한다. 클래스 컴포넌트를 직접 작성하지 않아도 된다.

```bash
pnpm add react-error-boundary
```

라우터의 페이지 단위로 감싼다.

```typescript
// src/app/router.tsx (또는 라우트 설정 파일)
import { ErrorBoundary } from 'react-error-boundary'

const PageErrorFallback = ({ resetErrorBoundary }: { resetErrorBoundary: () => void }) => (
  <div role="alert">
    <p>{ERROR_MESSAGES.unknown}</p>
    <button onClick={resetErrorBoundary}>다시 시도</button>
  </div>
)

const router = createBrowserRouter([
  {
    path: '/users',
    element: (
      <ErrorBoundary FallbackComponent={PageErrorFallback}>
        <UsersPage />
      </ErrorBoundary>
    ),
  },
])
```

섹션 단위 격리가 필요할 때도 동일하게 `ErrorBoundary`로 감싼다.

```typescript
<ErrorBoundary FallbackComponent={WidgetError}>
  <RevenueWidget />
</ErrorBoundary>
```

`error.message`를 직접 화면에 출력하지 않는다. 내부 구현이 노출될 수 있다.

---

## 3. 에러 상태 UX 패턴

### 에러 유형별 UI 선택 기준

| 상황 | UI 패턴 | 이유 |
| ---- | ------- | ---- |
| 사용자 액션 실패 (저장, 삭제) | Toast | 현재 작업 흐름을 끊지 않음 |
| 페이지 진입 시 데이터 없음 (404) | Empty State | 에러가 아닌 상태로 표현 |
| 폼 입력 오류 (422) | Inline 에러 | 어느 필드가 문제인지 명확히 |
| 서버 오류 (5xx) | 재시도 버튼 | 사용자가 직접 해결할 수 없음을 안내 |
| 권한 없음 (403) | 페이지 내 안내 | 동작을 막지만 앱은 유지 |
| 인증 만료 (401) | 리다이렉트 | 로그인 후 복귀 |
| 렌더링 예외 | Error Boundary fallback | 앱 크래시 방지 |

### Empty State vs Error State 구분

404는 "없음"이지 에러가 아니다. 에러처럼 표시하면 사용자가 자신의 잘못으로 오해할 수 있다.

```typescript
// ❌ 404를 에러로 표현
if (error) return <ErrorMessage message="오류가 발생했습니다" />

// ✅ 404는 Empty State로
if (isApiError(error) && error.status === 404) return <EmptyState message="데이터가 없습니다" />
if (error) return <RetryState onRetry={refetch} />
```

---

## 4. 에러 로깅

에러는 사용자에게 표시하는 것과 별개로 반드시 로깅한다. 두 관심사를 섞지 않는다.

```typescript
// src/lib/error/logger.ts
export const logError = (error: unknown, context?: Record<string, unknown>) => {
  if (process.env.NODE_ENV === 'development') {
    console.error('[Error]', error, context)
    return
  }
  // 프로덕션: Sentry, Datadog 등 연동
  // captureException(error, { extra: context })
}
```

Error Boundary의 `componentDidCatch`, mutation의 `onError`, 전역 `unhandledrejection` 핸들러에서 이 함수를 호출한다.
