# Skills Zip

Claude Code 커스텀 스킬 모음입니다. `.claude/skills/` 아래에 위치하며, 요청 내용에 따라 자동으로 트리거됩니다.

## 스킬 목록

| 스킬                                            | 설명                                                                                       | 트리거 예시                                                           |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| [git-commit-convention](#git-commit-convention) | feat/fix/refactor 등 커밋 타입 선택 기준과 메시지 형식                                     | "커밋해줘", "커밋 메시지 작성", "어떤 타입 써야 해", "feat이야 fix야" |
| [coding-conventions](#coding-conventions)       | 코드 작성·리뷰·리팩토링 시 기본 컨벤션 적용                                                | "코드 작성해줘", "리뷰해줘", "이 코드 괜찮아?"                        |
| [react-package](#react-package)                 | 독립 React/TypeScript 프로젝트 컨벤션 (컴포넌트, 훅, Zustand, 성능 최적화)                 | "컴포넌트 만들어줘", "훅 작성해줘", "Zustand 써줘", "리렌더링 줄여줘" |
| [nextjs-app-router](#nextjs-app-router)         | Next.js App Router 컨벤션 (RSC, Suspense, Hydration, 도메인 구조)                          | "페이지 만들어줘", "서버 컴포넌트", "클라이언트 컴포넌트", "Suspense" |
| [turborepo-workspace](#turborepo-workspace)     | Turborepo + pnpm 모노레포 구조, 공통 패키지, CI/CD, Dockerfile                             | "모노레포 설정해줘", "새 패키지 추가", "CI 설정해줘", "도커파일"      |
| [web-accessibility](#web-accessibility)         | 시맨틱 HTML, ARIA, 키보드/포커스, 색상 대비 적용 + Lighthouse·axe-core로 점수 측정         | "접근성 확인해줘", "aria-label", "키보드 내비게이션", "WCAG"          |
| [css-layers](#css-layers)                       | CSS Cascade Layers 기반 스타일 관리                                                        | "CSS 추가해줘", "스타일 수정해줘", "다크모드 색상", "globals.css"     |
| [design-system](#design-system)                 | 디자인 시스템 구축 및 프로젝트 적용                                                        | "디자인 시스템 만들어줘", "톤앤매너 잡아줘", "디자인 입혀줘"          |
| [frontend-test](#frontend-test)                 | Vitest 기반 UI 테스트 / 유닛 테스트 작성                                                   | "테스트 작성해줘", "컴포넌트 테스트", "훅 테스트"                     |
| [api-integration](#api-integration)             | TanStack Query + fetch 클라이언트, Query Factory 패턴                                      | "API 연동해줘", "React Query", "fetch 설정", "인터셉터"               |
| [error-handling](#error-handling)               | API 에러(status 기반)와 UI 에러(Error Boundary) 분리 처리                                  | "에러 처리해줘", "Error Boundary", "toast 알림", "401 처리"           |
| [web-performance](#web-performance)             | Lighthouse 기반 성능 점검 및 최적화                                                        | "성능 확인해줘", "lighthouse 돌려줘", "LCP 개선"                      |
| [lint-prettier-setup](#lint-prettier-setup)     | ESLint + Prettier + Husky 설정을 프로젝트 표준에 맞게 세팅. 단일 프로젝트 및 monorepo 지원 | "lint 설정해줘", "eslint 추가해줘", "새 패키지 설정", "husky 설정"    |
| [project-briefing](#project-briefing)           | 프로젝트 현황 종합 진단 후 docs/에 브리핑 문서 생성                                        | "프로젝트 현황 브리핑해줘", "프로젝트 진단해줘", "설정 리뷰해줘"      |
| [code-review](#code-review)                     | 코드 리뷰 수행                                                                             | "코드 리뷰해줘", "PR 리뷰해줘"                                        |
| [grill-me](#grill-me)                           | 계획·설계를 집요하게 인터뷰하며 검증                                                       | "grill me", "계획 검증해줘", "설계 스트레스 테스트"                   |

---

## git-commit-convention

커밋 메시지를 작성할 때 타입을 선택하고 형식을 맞춘다. `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `perf`, `ci`, `revert` 10가지 타입과 선택 기준을 다룬다.

**트리거**: "커밋해줘", "커밋 메시지 작성해줘", "어떤 타입 써야 해", "feat이야 fix야", "커밋 컨벤션", "git commit" 등 커밋 관련 모든 요청.

---

## coding-conventions

코드 작성·리뷰·리팩토링 시 기본 컨벤션을 적용한다. `var` 금지, async/await 우선, early return, 하드코딩 금지, 중복 조건 제거 등 프로젝트 공통 규칙을 다룬다.

**트리거**: "코드 작성해줘", "리뷰해줘", "리팩토링해줘", "컨벤션 확인해줘", "이 코드 괜찮아?" 등 코드 관련 모든 요청.

---

## react-package

Next.js가 아닌 독립 React/TypeScript 프로젝트에서 사용한다. TypeScript 컨벤션, 컴포넌트 패턴, React Compiler 최적화, Zustand 상태 관리(도메인별 separate store), TanStack Query 커스텀 훅 래핑을 다룬다.

**트리거**: "컴포넌트 만들어줘", "훅 작성해줘", "Zustand 써줘", "리렌더링 줄여줘", "성능 개선해줘", "React 코드 리뷰해줘" 등 독립 React 프로젝트의 모든 작업. Next.js라면 `nextjs-app-router` 스킬을 사용한다.

---

## nextjs-app-router

Next.js App Router 기반 코드를 작성하거나 수정할 때 사용한다. Server/Client Component 경계, Suspense 스트리밍, Hydration 패턴, 도메인 기반 폴더 구조, TypeScript 컨벤션을 적용한다.

**트리거**: `src/` 내부 파일을 추가·수정하는 모든 작업. "페이지 만들어줘", "컴포넌트 추가", "서버 컴포넌트", "클라이언트 컴포넌트", "Suspense", "하이드레이션 에러", "API 라우트" 등.

---

## turborepo-workspace

Turborepo + pnpm 모노레포 구조를 설정하거나 관리할 때 사용한다. `packages/` 아래 공통 패키지(ui, utils, styles, tsconfig, eslint-config, prettier-config) 구성, turbo.json 파이프라인, 프로젝트별 명령어, GitHub Actions CI/CD, Dockerfile(turbo prune 기반 멀티 스테이지)을 다룬다.

**트리거**: "모노레포 설정해줘", "새 패키지 추가", "공통 설정 만들어줘", "CI 설정해줘", "도커파일 만들어줘", "@repo/ui", "pnpm workspace" 등 모노레포 루트 또는 `packages/` 관련 작업.

---

## web-accessibility

웹 접근성(a11y) 관련 작업에 사용한다. 시맨틱 HTML, ARIA 레이블·상태, 키보드 내비게이션, 포커스 관리(트랩 포함), 색상 대비, 스크린 리더 지원, heading 계층을 다룬다.

**트리거**: "접근성 확인해줘", "a11y 개선", "스크린 리더", "키보드 내비게이션", "aria-label", "포커스 관리", "색상 대비", "WCAG" 등 접근성 관련 모든 작업.

---

## css-layers

CSS 파일을 건드리는 모든 작업에 사용한다. `@layer theme / base / components / utilities` 구조로 스타일을 관리하며, 어느 레이어에 무엇을 넣을지 결정한다.

**트리거**: "CSS 추가해줘", "스타일 수정", "토큰 추가", "다크모드 색상", "globals.css", "index.css" 수정 요청.

---

## design-system

Design.md 유무에 따라 두 가지 모드로 동작한다.

- **Design.md 없음** → 인터뷰(브랜드 키워드, 레퍼런스, 플랫폼, 컬러, 다크모드)를 진행하고 토큰을 확정해 `.claude/skills/design-system/docs/[프로젝트명]Design.md`를 생성한다.
- **Design.md 있음** → 프로젝트 스택을 분석하고 `css-layers` 스킬을 참조해 `@layer theme`에 토큰을 주입한다. Tailwind가 있으면 병행 적용한다.

**트리거**: "디자인 시스템 만들어줘", "톤앤매너 잡아줘", "디자인 입혀줘", "컬러 시스템", "타이포그래피", "브랜드 색상" 등 디자인 관련 작업.

---

## frontend-test

Vitest + React Testing Library 기반 테스트를 작성한다. UI 테스트(컴포넌트)와 유닛 테스트(함수·훅)를 구분하고, react-i18next, framer-motion, Zustand, React Router, TanStack Query 등 주요 라이브러리 mock 패턴을 포함한다.

**트리거**: "테스트 작성해줘", "테스트 만들어줘", "컴포넌트 테스트", "훅 테스트", "unit test", "ui test" 등 테스트 관련 모든 요청.

---

## api-integration

TanStack Query + fetch 클라이언트, Query Factory 패턴으로 API를 연동한다. 인터셉터, 에러 처리, 캐싱 전략을 다룬다.

**트리거**: "API 연동해줘", "React Query", "fetch 설정", "인터셉터", "캐싱 전략" 등.

---

## error-handling

API 에러(HTTP status 기반)와 UI 에러(Error Boundary)를 분리 처리한다. toast 알림, 401 리다이렉트, 전역 에러 상태를 다룬다.

**트리거**: "에러 처리해줘", "Error Boundary", "toast 알림", "401 처리" 등.

---

## web-performance

Lighthouse를 실행해 성능 지표(LCP, CLS, FID, TBT)를 측정하고 개선 방향을 제시한다. 구현 완료 후 최종 점검 시 또는 성능 이슈 발생 시 사용한다.

**트리거**: "성능 확인해줘", "lighthouse 돌려줘", "최적화해줘", "bundle 크기 줄여줘", "LCP 개선", "Core Web Vitals" 등.

---

## lint-prettier-setup

새 프로젝트 또는 새 패키지/앱을 추가할 때 ESLint와 Prettier 설정을 표준에 맞게 세팅한다. 단일 프로젝트와 monorepo(Turborepo/pnpm) 모두 지원하며, Husky pre-commit 훅도 설정할 수 있다.

**트리거**: "lint 설정해줘", "eslint 추가해줘", "prettier 설정", "새 패키지 설정", "프로젝트 초기 세팅", "husky 설정".

---

## project-briefing

프로젝트 현황을 종합 진단하고 `docs/project-briefing.md`를 생성한다. 환경 설정, 아키텍처, 보안, 성능, 개발자 경험, AI/스킬 설정까지 전 영역을 점검한다.

**트리거**: "프로젝트 현황 브리핑해줘", "프로젝트 상태 체크", "프로젝트 진단해줘", "monorepo 점검", "설정 리뷰해줘".

---

## code-review

코드 변경 사항을 리뷰한다. PR 또는 특정 파일·함수를 대상으로 버그, 최적화, 컨벤션 위반을 점검한다.

**트리거**: "코드 리뷰해줘", "PR 리뷰해줘", "이 코드 봐줘".

---
