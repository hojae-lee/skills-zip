---
name: nextjs-app-router
description: Use this skill whenever writing, creating, or modifying any Next.js or React code using the App Router — including pages, components, hooks, types, utilities, data fetching, or routing. Enforces RSC/Client Component boundaries, Suspense-based streaming, hydration safety, domain-based folder structure, TypeScript conventions, and TDD workflow. Trigger on requests like "페이지 만들어줘", "컴포넌트 추가해줘", "훅 작성해줘", "데이터 페칭", "로딩 UI", "서버 컴포넌트", "클라이언트 컴포넌트", "하이드레이션 에러", "Suspense", "API 라우트", or any code modification inside an app's src/ directory. Turborepo 모노레포 환경이라면 turborepo-workspace 스킬도 함께 사용한다.
---

# Next.js 16 App Router 컨벤션

## 앱 구조

```text
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── loading.tsx               # Suspense 기반 로딩 UI
│   ├── error.tsx                 # 에러 바운더리 ('use client' 필수)
│   ├── not-found.tsx
│   ├── globals.css
│   ├── api/                      # Route Handlers
│   │   └── [resource]/
│   │       └── route.ts
│   └── [domain]/
│       ├── page.tsx
│       ├── layout.tsx
│       ├── loading.tsx
│       ├── error.tsx
│       ├── components/
│       ├── hooks/
│       └── types/
├── common/
│   ├── components/
│   ├── hooks/
│   ├── types/
│   └── utils/
├── lib/
└── providers/
```

## Server Component vs Client Component

모든 컴포넌트는 기본적으로 Server Component다. 클라이언트 기능이 필요할 때만 `'use client'`를 추가한다.

### Server Component 사용 (기본)

서버에서 렌더링되며 번들 크기에 포함되지 않는다. DB 접근, 민감한 데이터, 무거운 라이브러리는 서버에서 처리한다.

```tsx
const UserList = async () => {
  const users = await fetchUsers()
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>
}
```

### Client Component (`'use client'` 필요한 경우)

- 이벤트 핸들러 (`onClick`, `onChange` 등)
- React hooks (`useState`, `useEffect`, `useRef` 등)
- 브라우저 전용 API (`window`, `localStorage` 등)

```tsx
'use client'

const Counter = () => {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

### 컴포지션 패턴

`'use client'`는 가능한 한 leaf 컴포넌트까지 밀어내려 서버 영역을 최대화한다.

```tsx
// ✅ 인터랙션이 필요한 부분만 클라이언트로 분리
const Page = async () => {
  const data = await fetchData()
  return (
    <main>
      <StaticContent data={data} />   {/* Server Component */}
      <InteractiveWidget />           {/* Client Component */}
    </main>
  )
}
```

Client Component 안에 Server Component를 직접 import하면 클라이언트 번들에 포함된다. 대신 `children` prop으로 전달한다.

## Suspense & 성능

### Streaming with Suspense

느린 컴포넌트를 `<Suspense>`로 감싸 빠른 부분 먼저 렌더링한다.

```tsx
const Page = () => (
  <main>
    <Header />
    <Suspense fallback={<ProductSkeleton />}>
      <ProductDetail />
    </Suspense>
    <Suspense fallback={<ReviewSkeleton />}>
      <Reviews />
    </Suspense>
  </main>
)
```

### 데이터 페칭 병렬화

순차 페칭은 waterfall을 유발한다. `Promise.all`로 병렬 처리한다.

```tsx
// ❌ waterfall
const user = await fetchUser(id)
const posts = await fetchPosts(id)

// ✅ 병렬
const [user, posts] = await Promise.all([fetchUser(id), fetchPosts(id)])
```

### `loading.tsx` vs `<Suspense>` 직접 사용

- `loading.tsx` — route segment 전체의 로딩 UI, 페이지 진입 시 자동 적용
- `<Suspense>` — 컴포넌트 단위 로딩, 더 세밀한 제어

데이터 페칭 속도가 다른 영역은 별도 컴포넌트로 분리해 독립적인 Suspense 경계를 만든다.

## Hydration & UX

서버 HTML과 클라이언트 렌더링 결과가 다르면 hydration 에러가 발생한다.

```tsx
// ❌ 서버/클라이언트에서 값이 달라짐
const Component = () => <div>{Date.now()}</div>
const Component = () => <div>{window.innerWidth}</div>
```

**useEffect로 클라이언트 전용 값 처리**
```tsx
const Component = () => {
  const [width, setWidth] = useState(0)
  useEffect(() => { setWidth(window.innerWidth) }, [])
  return <div>{width}</div>
}
```

**`dynamic`으로 SSR 비활성화** — 브라우저 API에 의존하는 서드파티 컴포넌트
```tsx
import dynamic from 'next/dynamic'
const Map = dynamic(() => import('./Map'), { ssr: false })
```

**`suppressHydrationWarning`** — 불가피한 불일치 (날짜, locale 등)
```tsx
<time suppressHydrationWarning>{new Date().toLocaleString()}</time>
```

## 데이터 페칭

### API Route Handlers

`app/api/` 아래에 리소스 단위로 파일을 구성한다.

```text
app/api/
└── users/
    ├── route.ts          # GET /api/users, POST /api/users
    └── [id]/
        └── route.ts      # GET /api/users/:id, PATCH, DELETE
```

```ts
// app/api/users/route.ts
export const GET = async (req: Request) => {
  const users = await db.user.findMany()
  return Response.json(users)
}

export const POST = async (req: Request) => {
  const body = await req.json()
  const user = await db.user.create({ data: body })
  return Response.json(user, { status: 201 })
}
```

클라이언트에서의 데이터 페칭과 캐싱은 `api-integration` 스킬(TanStack Query)을 따른다.

### Server Component에서 직접 fetch

Server Component에서는 `async/await`로 직접 데이터를 가져올 수 있다.

```tsx
const ProductList = async () => {
  const res = await fetch(`${process.env.API_URL}/products`)
  const products = await res.json()
  return <ul>{products.map(p => <li key={p.id}>{p.name}</li>)}</ul>
}
```

### Server Actions

```tsx
'use server'

const createUser = async (formData: FormData) => {
  await db.user.create({ data: { name: formData.get('name') } })
  revalidatePath('/users')
}

<form action={createUser}>
  <input name="name" />
  <button type="submit">추가</button>
</form>
```

## Next.js 전용 컴포넌트

`<img>` 대신 `<Image>`, `<a>` 대신 `<Link>`를 사용한다.

```tsx
import Image from 'next/image'
import Link from 'next/link'

<Image src="/hero.png" alt="히어로" width={800} height={400} priority />
<Link href="/about">소개</Link>
```

## 라우팅

```text
app/products/[id]/page.tsx        → /products/123
app/blog/[...slug]/page.tsx       → /blog/a/b/c
```

```tsx
export const generateStaticParams = async () => {
  const products = await fetchProducts()
  return products.map(p => ({ id: p.id }))
}

export const generateMetadata = async ({ params }: Props): Promise<Metadata> => {
  const product = await fetchProduct(params.id)
  return { title: product.name, description: product.description }
}
```

## 환경변수

- `NEXT_PUBLIC_` prefix → 클라이언트 번들 포함, 브라우저 접근 가능
- prefix 없음 → 서버 전용, 클라이언트에 절대 노출 금지

## 컴포넌트 레이어

- **`src/common/components/`** — 여러 도메인에서 공유하는 앱 레벨 컴포넌트
- **`src/app/[domain]/components/`** — 도메인 컴포넌트
- 다른 도메인의 컴포넌트 직접 import 금지 → `common/`으로 올리기

## Import 규칙

- 앱 공통 코드 → `import { ... } from '@/common/...'`
- 도메인 내부 → `import { ... } from './components/...'` (상대 경로)
- 다른 도메인 → 직접 import 금지
- `tsconfig.json`에 alias가 정의되어 있으면 그것을 따른다

## TypeScript 컨벤션

- **`any` 금지** — `unknown`, 정확한 타입, 또는 제네릭 사용
- **Props** → `type` 사용 (`type CardProps = { ... }`)
- **데이터/도메인 객체** → `interface` 사용 (`interface User { ... }`)
- **컴포넌트** → 화살표 함수 (`const Foo = () => { ... }`)

### Export 규칙

- `page.tsx`, `layout.tsx` → default export (Next.js 요구사항)
- 도메인 컴포넌트 → default export
- 공통 유틸, 훅, 타입 → named export

## 파일 명명

- 컴포넌트 → PascalCase (`UserCard.tsx`)
- 훅 → camelCase + `use` 접두사 (`useUserData.ts`)
- 타입 → camelCase + `.types` 접미사 (`user.types.ts`)
- 유틸 → camelCase (`formatDate.ts`)
- 페이지 → `page.tsx` (Next.js 고정)

## TDD Workflow

테스트를 먼저 작성하고, 통과시키는 최소 구현 후, 리팩터링한다.

1. 실패하는 테스트 작성
2. 테스트를 통과시키는 최소 구현
3. 리팩터링 (테스트 통과 유지)
4. lint + format + test 실행

테스트 파일은 소스 파일 옆에 co-locate: `UserCard.tsx` → `UserCard.test.tsx`

### 테스트 대상

- 유틸 함수 → 입력/출력 단위 테스트
- 커스텀 훅 → `renderHook` + 상태 변화
- 컴포넌트 → 렌더링 + 사용자 인터랙션

### 명명 규칙

- 파일: `[대상].test.ts(x)`
- `describe`: 대상 이름
- `it`: `'~한다'` 형태로 행동 기술

## 코드 변경 후

하나라도 실패하면 완료로 간주하지 않는다.

```bash
pnpm lint
pnpm format
pnpm test
```
