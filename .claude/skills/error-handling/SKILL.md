---
name: error-handling
description: 에러 처리 패턴 스킬. API 에러(status code 기반)와 UI 에러(Error Boundary)를 명확하게 분리해서 처리한다. "에러 처리해줘", "에러 핸들링", "Error Boundary", "에러 페이지", "toast 알림", "API 에러", "500 에러", "401 처리", "에러 메시지", "fallback UI", "에러 상태 관리" 등의 요청에 반드시 사용한다. 에러 관련 작업이면 이 스킬을 먼저 확인해라.
---

# 에러 처리 스킬

## 핵심 원칙: 에러 처리 레벨 구분

| 에러 유형                               | 처리 레벨   | 처리 방식                                 |
| --------------------------------------- | ----------- | ----------------------------------------- |
| **API 에러** (401, 404, 500 등)         | 컴포넌트    | `resolveApiError()` → status 기반 UI 분기 |
| **컴포넌트 UI 에러** (예측 가능한 상태) | 컴포넌트    | `if (error)` 분기, toast, inline 메시지   |
| **페이지 UI 에러** (예측 불가 JS 예외)  | 페이지 레벨 | Error Boundary → fallback UI              |

**컴포넌트 레벨**: API 에러처럼 예측 가능한 에러는 컴포넌트 안에서 직접 처리한다. `if (isLoading)`, `if (error)` 분기로 상태에 맞는 UI를 렌더링한다.

**페이지 레벨**: 렌더링 도중 발생하는 예측 불가능한 JS 예외는 컴포넌트 안에서 잡을 수 없다. Error Boundary를 페이지 단위로 배치해서 앱 전체가 흰 화면이 되는 것을 방지한다.

서버에서 에러 메시지 문자열을 내려주더라도 UI에 그대로 노출하지 않는다. 사용자에게 보여줄 메시지는 클라이언트에서 status code 기반으로 결정한다.

---

## 1. 컴포넌트 레벨: API 에러 처리

`api-integration` 스킬의 `isApiError()` 타입 가드와 연계해서 사용한다.

### Status Code → UI 행동 매핑

```typescript
// src/lib/error/apiErrorHandler.ts
import { isApiError } from "@/lib/api/client";

export type ApiErrorAction =
  | { type: "redirect"; to: string }
  | { type: "toast"; message: string }
  | { type: "inline"; message: string }
  | { type: "empty" }
  | { type: "retry" };

export const resolveApiError = (error: unknown): ApiErrorAction => {
  if (!isApiError(error))
    return { type: "toast", message: ERROR_MESSAGES.unknown };

  if (error.status === 401) return { type: "redirect", to: "/login" };
  if (error.status === 403)
    return { type: "inline", message: ERROR_MESSAGES.forbidden };
  if (error.status === 404) return { type: "empty" };
  if (error.status === 409)
    return { type: "inline", message: ERROR_MESSAGES.conflict };
  if (error.status === 422)
    return { type: "inline", message: ERROR_MESSAGES.validation };
  if (error.status >= 500) return { type: "retry" };

  return { type: "toast", message: ERROR_MESSAGES.unknown };
};
```

### 에러 메시지 상수 (하드코딩 금지)

```typescript
// src/lib/error/messages.ts
export const ERROR_MESSAGES = {
  unknown: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해주세요.",
  forbidden: "접근 권한이 없습니다.",
  conflict: "이미 존재하는 데이터입니다.",
  validation: "입력 값을 확인해주세요.",
  network: "네트워크 연결을 확인해주세요.",
} as const;

// i18n 사용 시 메시지 대신 키로 관리
export const ERROR_KEYS = {
  unknown: "error.unknown",
  forbidden: "error.forbidden",
} as const;
```

### TanStack Query에서 처리

```typescript
const UserList = () => {
  const { data, error, isLoading, refetch } = useQuery(userQueries.list())
  const router = useRouter()

  if (isLoading) return <Skeleton />

  if (error) {
    const action = resolveApiError(error)
    if (action.type === 'redirect') { router.push(action.to); return null }
    if (action.type === 'empty')  return <EmptyState />
    if (action.type === 'retry')  return <RetryState onRetry={refetch} />
    if (action.type === 'inline') return <ErrorMessage message={action.message} />
  }

  return <List items={data} />
}
```

404는 "없음"이지 에러가 아니다. `EmptyState`로 표현한다.

### Mutation 에러 (toast)

```typescript
const useCreateUser = () => {
  const { toast } = useToast();

  return useMutation({
    mutationFn: (data: CreateUserInput) =>
      apiClient.request<User>("/users", {
        method: "POST",
        body: JSON.stringify(data),
      }),
    onError: (error) => {
      const action = resolveApiError(error);
      if (action.type === "toast" || action.type === "inline") {
        toast({ variant: "destructive", description: action.message });
      }
    },
  });
};
```

---

## 2. 페이지 레벨: Error Boundary

렌더링 중 발생하는 예측 불가능한 JS 예외를 잡는다. API 에러는 잡지 않는다.

### Next.js App Router

`error.tsx`를 페이지·도메인 폴더에 두면 자동으로 Error Boundary가 된다.

```text
app/
├── layout.tsx        ← global-error.tsx와 쌍으로: 최후 방어선
├── error.tsx         ← 루트 페이지 레벨
└── [domain]/
    ├── error.tsx     ← 도메인 페이지 레벨
    └── page.tsx
```

```typescript
// app/[domain]/error.tsx
'use client'

const DomainError = ({
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) => (
  <div role="alert">
    <p>{ERROR_MESSAGES.unknown}</p>
    <button onClick={reset}>다시 시도</button>
  </div>
)

export default DomainError
```

### React (App Router 없는 경우)

`react-error-boundary` 라이브러리를 사용한다.

```bash
pnpm add react-error-boundary
```

라우트 설정에서 페이지 단위로 감싼다.

```typescript
// src/app/router.tsx
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

`error.message`를 직접 화면에 출력하지 않는다.

---

## 3. UX 패턴 가이드

| 상황                          | UI 패턴                 | 이유                        |
| ----------------------------- | ----------------------- | --------------------------- |
| 사용자 액션 실패 (저장, 삭제) | Toast                   | 현재 작업 흐름을 끊지 않음  |
| 데이터 없음 (404)             | Empty State             | 에러가 아닌 상태로 표현     |
| 폼 입력 오류 (422)            | Inline 에러             | 어느 필드가 문제인지 명확히 |
| 서버 오류 (5xx)               | 재시도 버튼             | 사용자가 직접 해결 불가     |
| 권한 없음 (403)               | 페이지 내 안내          | 동작을 막지만 앱은 유지     |
| 인증 만료 (401)               | 리다이렉트              | 로그인 후 복귀              |
| 렌더링 예외 (JS crash)        | Error Boundary fallback | 앱 크래시 방지              |

---

## 4. 에러 로깅

```typescript
// src/lib/error/logger.ts
export const logError = (error: unknown, context?: Record<string, unknown>) => {
  if (process.env.NODE_ENV === "development") {
    console.error("[Error]", error, context);
    return;
  }
  // 프로덕션: Sentry, Datadog 등 연동
  // captureException(error, { extra: context })
};
```

mutation의 `onError`, Error Boundary의 에러 콜백에서 호출한다.
