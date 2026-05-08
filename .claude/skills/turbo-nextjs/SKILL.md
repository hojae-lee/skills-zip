---
name: turbo-nextjs
description: Use this skill whenever writing, creating, or modifying any Next.js or React code in this project — including new pages, components, hooks, types, or utilities. Enforces Turborepo + Next.js 16 monorepo architecture (packages/ui for atomic components, src/common for app-level shared code, src/app/ for domain-based features), TypeScript conventions, import rules, and TDD workflow with Vitest + React Testing Library. Trigger on any request like "add a page", "create a component", "add a hook", "set up a new feature", "refactor this file", "write a test", or any code modification in apps/ or packages/.
---

# Turborepo + Next.js 16 Monorepo Architecture

## 기술 스택

- **Turborepo** — 모노레포 태스크 오케스트레이션 (`turbo.json` 파이프라인)
- **Next.js 16** — App Router, React Server Components, PPR(Partial Prerendering)
- **pnpm workspaces** — 패키지 관리
- **Vitest + React Testing Library** — 테스트

## 전체 구조

```text
apps/<app-name>/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   └── [domain]/                 # 기능 도메인별 폴더
│   │       ├── page.tsx
│   │       ├── layout.tsx            # optional
│   │       ├── components/
│   │       ├── hooks/
│   │       └── types/
│   ├── common/                       # 앱 내 공통 코드 (app/ 외부)
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── types/
│   │   └── utils/
│   ├── lib/
│   └── providers/
├── public/
├── package.json
└── tsconfig.json

packages/
└── ui/                               # 아토믹 UI 컴포넌트 (@repo/ui)
    └── src/
        └── components/

turbo.json                            # Turbo 파이프라인 정의
```

## 컴포넌트 레이어

- **`packages/ui` (`@repo/ui`)** — Button, Input 등 아토믹 컴포넌트. 비즈니스 로직 없음.
- **`src/common/components/`** — 여러 도메인에서 공유하는 앱 레벨 컴포넌트.
- **`src/app/[domain]/components/`** — 도메인 컴포넌트. `@repo/ui`를 가져와 비즈니스 로직과 조합.

도메인 컴포넌트는 `@repo/ui`의 아토믹 컴포넌트를 조합해서 만든다. 아토믹 컴포넌트를 직접 수정하지 않는다.

## Import 규칙

- 아토믹 UI 컴포넌트 → `import { Button } from '@repo/ui'`
- 앱 공통 코드 → `import { ... } from '@/common/...'`
- 도메인 내부 → `import { ... } from './components/...'` (상대 경로)
- 다른 도메인 → 직접 import 금지, `common/`으로 올리기

import 시 `tsconfig.json`에 alias가 정의되어 있으면 그것을 따른다.

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

## 웹 접근성 원칙

`@repo/ui`의 Radix UI 기반 컴포넌트는 ARIA 기본값을 내장하고 있다. 그 위에 아래 원칙을 지킨다.

- **시맨틱 HTML 우선** — 클릭 이벤트를 `<div>`에 붙이지 않는다. 버튼은 `<button>`, 링크는 `<a>`, 목록은 `<ul>/<ol>`
- **아이콘 전용 버튼** — 텍스트 없이 아이콘만 있는 버튼에는 반드시 `aria-label` 추가
- **이미지 `alt`** — 의미 있는 이미지는 내용을 설명, 장식용 이미지는 `alt=""`
- **폼 입력** — `<input>`마다 `<label>`을 연결하거나 `aria-label` 지정. placeholder로 대체 금지
- **색상만으로 구분 금지** — 상태(에러, 성공)를 색상 외에 텍스트나 아이콘으로도 표현
- **focus 스타일** — `outline: none` 제거 시 대체 포커스 스타일 필수 (`focus-visible:ring-*`)
- **heading 계층** — `h1` → `h2` → `h3` 순서 유지, 레벨 건너뜀 금지

성능·접근성 종합 점검은 `web-performance` 스킬(Lighthouse 기반)로 실행한다.

## 코드 변경 후

하나라도 실패하면 완료로 간주하지 않는다.

```bash
# 루트에서 전체 실행 (Turbo 캐시 활용)
pnpm lint
pnpm format
pnpm test

# 특정 앱만 실행
pnpm --filter <app-name> test

# watch 모드
pnpm --filter <app-name> test:watch

# Turbo로 빌드 파이프라인 실행
turbo run build
turbo run lint test --filter=<app-name>
```
