---
name: api-integration
description: TanStack Query + fetch 기반 API 연동 스킬. Query Factory 패턴으로 쿼리 키를 관리하고, fetch 클라이언트에 인터셉터·환경변수·에러 처리를 설정한다. 하드코딩은 절대 금지. "API 연동해줘", "fetch 설정", "React Query", "TanStack Query", "쿼리 만들어줘", "mutation 작성", "API 클라이언트", "인터셉터", "토큰 갱신", "서버 상태 관리", "데이터 페칭" 등의 요청에 반드시 사용한다.
---

# API Integration 스킬

## 0단계: 프로젝트 환경 파악

```bash
# TanStack Query 설치 여부
grep -E "@tanstack/react-query" package.json

# 기존 API 클라이언트 파일 찾기
find . -name "*.ts" -path "*/lib/*" -o -name "*.ts" -path "*/api/*" | grep -v node_modules | head -10

# 환경변수 파일 확인
ls -la .env* 2>/dev/null

# 기존 fetch/axios 사용 패턴 확인
grep -rn "axios\|fetch(" --include="*.ts" --include="*.tsx" -l | grep -v node_modules | head -5
```

파악 후 확인할 것:

- 이미 API 클라이언트가 있으면 그것을 기반으로 확장한다
- 환경변수 네이밍 컨벤션 확인 (`NEXT_PUBLIC_` 등)
- QueryClient 설정이 있으면 읽고 따른다

---

## 하드코딩 금지 원칙

API 관련 코드에서 아래 항목은 절대 코드에 직접 쓰지 않는다.

| 금지 항목 | 대신 사용 |
| --------- | --------- |
| Base URL (`https://api.example.com`) | `process.env.NEXT_PUBLIC_API_URL` |
| API Key, Secret | `process.env.API_SECRET_KEY` |
| Timeout 숫자 | `API_CONFIG.timeout` 상수 |
| 에러 메시지 문자열 | 상수 파일 또는 i18n 키 |
| 재시도 횟수 | `API_CONFIG.retryCount` 상수 |

환경변수가 없으면 `.env.local` 예시를 만들어 사용자에게 제공한다.

```bash
# .env.local 예시
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_API_TIMEOUT=10000
```

---

## 1. Fetch 클라이언트 설정

`src/lib/api/client.ts` (또는 프로젝트 구조에 맞는 위치)에 작성한다.

```typescript
const API_CONFIG = {
  baseUrl: process.env.NEXT_PUBLIC_API_URL ?? '',
  timeout: Number(process.env.NEXT_PUBLIC_API_TIMEOUT ?? 10000),
  retryCount: 1,
} as const

type RequestInterceptor = (config: RequestInit & { url: string }) => RequestInit & { url: string }
type ResponseInterceptor = (response: Response) => Promise<Response>

const createApiClient = () => {
  const requestInterceptors: RequestInterceptor[] = []
  const responseInterceptors: ResponseInterceptor[] = []

  const addRequestInterceptor = (interceptor: RequestInterceptor) => {
    requestInterceptors.push(interceptor)
    return client
  }

  const addResponseInterceptor = (interceptor: ResponseInterceptor) => {
    responseInterceptors.push(interceptor)
    return client
  }

  const request = async <T>(endpoint: string, options: RequestInit = {}): Promise<T> => {
    let config = { url: `${API_CONFIG.baseUrl}${endpoint}`, ...options }

    for (const interceptor of requestInterceptors) {
      config = interceptor(config)
    }

    const { url, ...init } = config
    const controller = new AbortController()
    const timeoutId = setTimeout(() => controller.abort(), API_CONFIG.timeout)

    let response = await fetch(url, { ...init, signal: controller.signal })
    clearTimeout(timeoutId)

    for (const interceptor of responseInterceptors) {
      response = await interceptor(response)
    }

    if (!response.ok) {
      throw createApiError(response.status, await response.text(), endpoint)
    }

    return response.json() as Promise<T>
  }

  const client = { addRequestInterceptor, addResponseInterceptor, request }
  return client
}

export type ApiError = Error & { name: 'ApiError'; status: number; body: string; endpoint: string }

export const createApiError = (status: number, body: string, endpoint: string): ApiError => {
  const error = new Error(`[${status}] ${endpoint}`) as ApiError
  error.name = 'ApiError'
  error.status = status
  error.body = body
  error.endpoint = endpoint
  return error
}

export const isApiError = (error: unknown): error is ApiError =>
  error instanceof Error && error.name === 'ApiError'

export const apiClient = createApiClient()
```

### 인터셉터 등록 (`src/lib/api/interceptors.ts`)

```typescript
import { apiClient } from './client'

// Auth 토큰 주입
apiClient.addRequestInterceptor((config) => ({
  ...config,
  headers: {
    'Content-Type': 'application/json',
    ...config.headers,
    Authorization: `Bearer ${getAccessToken()}`,
  },
}))

// 401 → 토큰 갱신 후 재시도
apiClient.addResponseInterceptor(async (response) => {
  if (response.status !== 401) return response

  await refreshAccessToken()
  // 재시도는 호출부에서 처리 (무한 루프 방지)
  return response
})

const getAccessToken = (): string => {
  // 토큰 저장소 (cookie, memory 등) 에서 읽기
  // 구현은 프로젝트 인증 방식에 따라 결정
  return ''
}

const refreshAccessToken = async (): Promise<void> => {
  // 갱신 로직
}
```

---

## 2. Query Factory 패턴

쿼리 키와 쿼리 함수를 도메인별로 한 파일에 모아서 관리한다. 쿼리 키가 분산되면 무효화(invalidate)와 캐시 관리가 어려워지기 때문이다.

### 파일 위치

```text
src/
└── [domain]/
    └── queries/
        └── [domain]Queries.ts   ← 쿼리 팩토리
```

### 팩토리 패턴

```typescript
// src/user/queries/userQueries.ts
import { apiClient } from '@/lib/api/client'
import type { User, UserFilters } from '../types'

const fetchUser = (id: string) =>
  apiClient.request<User>(`/users/${id}`)

const fetchUsers = (filters?: UserFilters) =>
  apiClient.request<User[]>(`/users${toQueryString(filters)}`)

export const userQueries = {
  // 계층적 키 구조: invalidate 범위를 조절할 수 있다
  all: () => ({ queryKey: ['users'] as const }),

  lists: () => ({
    ...userQueries.all(),
    queryKey: [...userQueries.all().queryKey, 'list'] as const,
  }),

  list: (filters?: UserFilters) => ({
    ...userQueries.lists(),
    queryKey: [...userQueries.lists().queryKey, filters] as const,
    queryFn: () => fetchUsers(filters),
  }),

  details: () => ({
    ...userQueries.all(),
    queryKey: [...userQueries.all().queryKey, 'detail'] as const,
  }),

  detail: (id: string) => ({
    ...userQueries.details(),
    queryKey: [...userQueries.details().queryKey, id] as const,
    queryFn: () => fetchUser(id),
  }),
}
```

### Invalidation 예시

계층 구조 덕분에 범위를 골라서 무효화할 수 있다.

```typescript
// 모든 user 쿼리 무효화
queryClient.invalidateQueries(userQueries.all())

// 목록만 무효화
queryClient.invalidateQueries(userQueries.lists())

// 특정 유저만 무효화
queryClient.invalidateQueries(userQueries.detail(userId))
```

---

## 3. TanStack Query 사용 패턴

### useQuery

```typescript
import { useQuery } from '@tanstack/react-query'
import { userQueries } from '../queries/userQueries'

const useUser = (id: string) =>
  useQuery({
    ...userQueries.detail(id),
    enabled: !!id,            // id 없으면 실행 안 함
    staleTime: 1000 * 60 * 5, // 5분간 fresh
  })
```

### useMutation

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { userQueries } from '../queries/userQueries'

const useUpdateUser = () => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (data: UpdateUserInput) =>
      apiClient.request<User>(`/users/${data.id}`, {
        method: 'PATCH',
        body: JSON.stringify(data),
      }),

    // 성공 시 관련 쿼리 무효화
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries(userQueries.detail(variables.id))
      queryClient.invalidateQueries(userQueries.lists())
    },
  })
}
```

### 낙관적 업데이트 (Optimistic Update)

응답을 기다리지 않고 UI를 먼저 업데이트해서 빠른 체감을 준다.

```typescript
useMutation({
  mutationFn: updateUser,
  onMutate: async (newData) => {
    await queryClient.cancelQueries(userQueries.detail(newData.id))
    const previous = queryClient.getQueryData(userQueries.detail(newData.id).queryKey)
    queryClient.setQueryData(userQueries.detail(newData.id).queryKey, newData)
    return { previous }
  },
  onError: (_, newData, context) => {
    // 실패 시 롤백
    queryClient.setQueryData(userQueries.detail(newData.id).queryKey, context?.previous)
  },
  onSettled: (_, __, variables) => {
    queryClient.invalidateQueries(userQueries.detail(variables.id))
  },
})
```

---

## 4. QueryClient 설정

```typescript
// src/lib/api/queryClient.ts
import { QueryClient } from '@tanstack/react-query'
import { ApiError } from './client'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,    // 1분 기본
      retry: (failureCount, error) => {
        // 4xx는 재시도 안 함 (클라이언트 오류)
        if (isApiError(error) && error.status < 500) return false
        return failureCount < 2
      },
    },
    mutations: {
      retry: false,
    },
  },
})
```

---

## 5. 에러 처리

에러 상태는 컴포넌트에서 직접 처리하거나 Error Boundary로 위임한다.

```typescript
const UserProfile = ({ id }: { id: string }) => {
  const { data, isLoading, error } = useUser(id)

  if (isLoading) return <Skeleton />

  if (isApiError(error)) {
    if (error.status === 404) return <NotFound />
    if (error.status >= 500) return <ServerError />
  }

  if (error) return <GenericError />

  return <Profile user={data} />
}
```

전역 에러는 `QueryClient`의 `onError` 대신 React Error Boundary로 처리한다. (onError는 deprecated)

---

## 유틸

### Query String 변환

```typescript
// src/lib/api/utils.ts
export function toQueryString(params?: Record<string, unknown>): string {
  if (!params) return ''
  const filtered = Object.entries(params).filter(([, v]) => v != null)
  if (!filtered.length) return ''
  return '?' + new URLSearchParams(filtered.map(([k, v]) => [k, String(v)])).toString()
}
```
