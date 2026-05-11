# Skills Zip

Claude Code 커스텀 스킬 모음입니다. `.claude/skills/` 아래에 위치하며, 요청 내용에 따라 자동으로 트리거됩니다.

## 스킬 목록

| 스킬                                                              | 설명                                                                                       | 트리거 예시                                                           |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| [git-commit-convention](#git-commit-convention)                   | feat/fix/refactor 등 커밋 타입 선택 기준과 메시지 형식                                     | "커밋해줘", "커밋 메시지 작성", "어떤 타입 써야 해", "feat이야 fix야"  |
| [react-package](#react-package)                                   | 독립 React/TypeScript 프로젝트 컨벤션 (컴포넌트, 훅, Zustand, 성능 최적화)                 | "컴포넌트 만들어줘", "훅 작성해줘", "Zustand 써줘", "리렌더링 줄여줘" |
| [nextjs-app-router](#nextjs-app-router)                           | Next.js 16 App Router 컨벤션 (RSC, Suspense, Hydration, 도메인 구조)                       | "페이지 만들어줘", "서버 컴포넌트", "클라이언트 컴포넌트", "Suspense" |
| [turborepo-workspace](#turborepo-workspace)                       | Turborepo + pnpm 모노레포 구조, 공통 패키지, CI/CD, Dockerfile                             | "모노레포 설정해줘", "새 패키지 추가", "CI 설정해줘", "도커파일"      |
| [web-accessibility](#web-accessibility)                           | 시맨틱 HTML, ARIA, 키보드/포커스, 색상 대비 적용 + Lighthouse·axe-core로 점수 측정         | "접근성 확인해줘", "aria-label", "키보드 내비게이션", "WCAG"          |
| [css-layers](#css-layers)                                         | CSS Cascade Layers 기반 스타일 관리                                                        | "CSS 추가해줘", "스타일 수정해줘", "다크모드 색상", "globals.css"     |
| [design-system](#design-system)                                   | 디자인 시스템 구축 및 프로젝트 적용                                                        | "디자인 시스템 만들어줘", "톤앤매너 잡아줘", "디자인 입혀줘"          |
| [frontend-testing](#frontend-testing)                             | Vitest 기반 UI 테스트 / 유닛 테스트 작성                                                   | "테스트 작성해줘", "컴포넌트 테스트", "훅 테스트"                     |
| [api-integration](#api-integration)                               | TanStack Query + fetch 클라이언트, Query Factory 패턴                                      | "API 연동해줘", "React Query", "fetch 설정", "인터셉터"               |
| [error-handling](#error-handling)                                 | API 에러(status 기반)와 UI 에러(Error Boundary) 분리 처리                                  | "에러 처리해줘", "Error Boundary", "toast 알림", "401 처리"           |
| [web-performance](#web-performance)                               | Lighthouse 기반 성능 점검 및 최적화                                                        | "성능 확인해줘", "lighthouse 돌려줘", "LCP 개선"                      |
| [lint-prettier-setup](#lint-prettier-setup)                       | ESLint + Prettier + Husky 설정을 프로젝트 표준에 맞게 세팅. 단일 프로젝트 및 monorepo 지원 | "lint 설정해줘", "eslint 추가해줘", "새 패키지 설정", "husky 설정"    |
| [brainstorming](#brainstorming)                                   | 구현 전 요구사항·설계 탐색, 2-3가지 접근법 제시, 스펙 문서 작성                            | 새 기능 만들기 전, 설계 논의, "어떻게 구현할지"                       |
| [writing-plans](#writing-plans)                                   | 스펙을 바탕으로 파일별·단계별 구현 계획 문서 작성                                          | 스펙 완성 후 구현 전, 멀티스텝 작업                                   |
| [executing-plans](#executing-plans)                               | 작성된 계획을 순서대로 실행, 블로커 발생 시 중단                                           | 계획 파일이 있을 때 실행 시작                                         |
| [test-driven-development](#test-driven-development)               | Red→Green→Refactor TDD 사이클 강제                                                         | 기능/버그픽스 구현 전                                                 |
| [systematic-debugging](#systematic-debugging)                     | 근본 원인 분석 → 가설 → 단일 수정 4단계 디버깅                                             | 버그, 테스트 실패, 예상치 못한 동작 발생 시                           |
| [requesting-code-review](#requesting-code-review)                 | 코드 리뷰어 서브에이전트 디스패치                                                          | 태스크 완료 후, 머지 전                                               |
| [receiving-code-review](#receiving-code-review)                   | 리뷰 피드백을 기술적으로 검증하고 반영                                                     | 코드 리뷰 피드백을 받았을 때                                          |
| [verification-before-completion](#verification-before-completion) | 완료 선언 전 검증 명령 실행 후 증거 확인                                                   | 완료/커밋/PR 생성 전                                                  |
| [subagent-driven-development](#subagent-driven-development)       | 태스크별 서브에이전트 디스패치 + 스펙·품질 2단계 리뷰                                      | 구현 계획 실행 시 (서브에이전트 가능한 환경)                          |
| [using-git-worktrees](#using-git-worktrees)                       | 격리된 워크트리 생성 후 클린 베이스라인 확인                                               | 피처 작업 시작 전, 계획 실행 전                                       |

---

## git-commit-convention

커밋 메시지를 작성할 때 타입을 선택하고 형식을 맞춘다. `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `perf`, `ci`, `revert` 10가지 타입과 선택 기준을 다룬다.

**트리거**: "커밋해줘", "커밋 메시지 작성해줘", "어떤 타입 써야 해", "feat이야 fix야", "커밋 컨벤션", "git commit" 등 커밋 관련 모든 요청.

---

## react-package

Next.js가 아닌 독립 React/TypeScript 프로젝트에서 사용한다. TypeScript 컨벤션, 컴포넌트 패턴, React Compiler 최적화, Zustand 상태 관리(도메인별 separate store), TanStack Query 커스텀 훅 래핑을 다룬다.

**트리거**: "컴포넌트 만들어줘", "훅 작성해줘", "Zustand 써줘", "리렌더링 줄여줘", "성능 개선해줘", "React 코드 리뷰해줘" 등 독립 React 프로젝트의 모든 작업. Next.js라면 `nextjs-app-router` 스킬을 사용한다.

---

## nextjs-app-router

Next.js 16 App Router 기반 코드를 작성하거나 수정할 때 사용한다. Server/Client Component 경계, Suspense 스트리밍, Hydration 패턴, 도메인 기반 폴더 구조, TypeScript 컨벤션, TDD 워크플로를 적용한다.

**트리거**: `src/` 내부 파일을 추가·수정하는 모든 작업. "페이지 만들어줘", "컴포넌트 추가", "서버 컴포넌트", "클라이언트 컴포넌트", "Suspense", "하이드레이션 에러", "API 라우트" 등. Turborepo 모노레포라면 `turborepo-workspace` 스킬도 함께 사용한다.

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

## frontend-testing

Vitest + React Testing Library 기반 테스트를 작성한다. UI 테스트(컴포넌트)와 유닛 테스트(함수·훅)를 구분하고, 테스트 환경이 이미 설정돼 있으면 기존 패턴을 그대로 따른다. 테스트가 불필요한 단순 컴포넌트는 작성하지 않는다.

**`test-driven-development`와 역할 분리**:

- `test-driven-development` → **기본 원칙** (언제, 어떤 순서로 — Red-Green-Refactor 사이클)
- `frontend-testing` → **규격** (어떻게 쓸지 — Vitest/RTL 패턴, 파일 구조, 쿼리 우선순위)

기능 구현 시 두 스킬을 함께 사용한다. `test-driven-development`가 사이클을 정의하고, 실제 테스트 코드는 이 스킬의 패턴을 따른다.

**트리거**: "테스트 작성해줘", "test 추가", "컴포넌트 테스트", "훅 테스트", "spec 작성" 등 테스트 관련 모든 요청.

---

## web-performance

Lighthouse를 실행해 성능 지표(LCP, CLS, FID, TBT)를 측정하고 개선 방향을 제시한다. 구현 완료 후 최종 점검 시 또는 성능 이슈 발생 시 사용한다.

**트리거**: "성능 확인해줘", "lighthouse 돌려줘", "최적화해줘", "bundle 크기 줄여줘", "LCP 개선", "Core Web Vitals" 등.

---

## lint-prettier-setup

새 프로젝트 또는 새 패키지/앱을 추가할 때 ESLint와 Prettier 설정을 이 프로젝트의 표준에 맞게 세팅한다. 단일 프로젝트와 monorepo(Turborepo/pnpm) 모두 지원하며, Husky pre-commit 훅도 설정할 수 있다.

**트리거**: "lint 설정해줘", "eslint 추가해줘", "prettier 설정", "새 패키지 설정", "프로젝트 초기 세팅", "새 앱 추가", "코드 품질 설정", "pre-commit 설정", "husky 설정".

---

## brainstorming

구현 전에 요구사항과 설계를 탐색한다. 프로젝트 맥락을 파악하고 질문을 통해 아이디어를 구체화한 뒤, 2-3가지 접근법을 제시하고 스펙 문서를 작성한다. 스펙 승인 후 `writing-plans` 스킬로 이어진다.

**트리거**: 새 기능 추가 전, 컴포넌트 설계 논의, "어떻게 구현할지", "뭘 만들어야 할지" 등 구현 이전 설계 단계.

---

## writing-plans

스펙을 바탕으로 엔지니어가 바로 실행할 수 있는 단계별 구현 계획 문서를 작성한다. 수정할 파일, 실제 코드, 실행 명령어, 예상 출력까지 포함한다. TDD 워크플로를 기본으로 한다.

**트리거**: 스펙 또는 요구사항이 있는 멀티스텝 작업 시작 전.

---

## executing-plans

작성된 구현 계획 파일을 로드하고 태스크를 순서대로 실행한다. 블로커 발생 시 즉시 중단하고 질문한다. 모든 태스크 완료 후 `finishing-a-development-branch`로 마무리한다.

**트리거**: 계획 파일이 있고 실행을 시작할 때.

---

## test-driven-development

테스트 먼저 작성하고, 실패를 확인하고, 최소 구현으로 통과시킨 뒤 리팩터링한다. 프로덕션 코드는 실패하는 테스트 없이 작성하지 않는다.

**트리거**: 새 기능 구현, 버그 수정, 리팩터링 전 언제나.

---

## systematic-debugging

무작위 수정 대신 근본 원인을 먼저 찾는다. 에러 분석 → 패턴 파악 → 단일 가설 검증 → 수정의 4단계를 따른다. 3번 이상 실패하면 아키텍처를 의심한다.

**트리거**: 버그 발생, 테스트 실패, 예상치 못한 동작, 빌드 실패 등 모든 기술적 문제.

---

## requesting-code-review

코드 리뷰어 서브에이전트를 디스패치해 작업 결과를 검증한다. git SHA 범위를 기준으로 리뷰를 요청하고 피드백을 반영한다.

**트리거**: 태스크 완료 후, 머지 전, 복잡한 버그 수정 후.

---

## receiving-code-review

리뷰 피드백을 감정적으로 수용하지 않고 기술적으로 검증한다. 불명확한 항목은 먼저 질문하고, YAGNI 원칙으로 불필요한 기능 추가를 거부한다. 항목별로 하나씩 구현하고 테스트한다.

**트리거**: 코드 리뷰 피드백을 받았을 때.

---

## verification-before-completion

완료를 선언하기 전에 반드시 검증 명령을 실행하고 결과를 확인한다. "아마 통과할 것", "이미 확인했음" 같은 가정으로 완료 선언 금지.

**트리거**: 완료 선언, 커밋, PR 생성, 다음 태스크로 이동 전 언제나.

---

## subagent-driven-development

구현 계획의 각 태스크를 독립적인 서브에이전트에 위임한다. 태스크별로 스펙 준수 리뷰 → 코드 품질 리뷰 2단계를 거친다. 인간 파트너 개입 없이 모든 태스크를 연속 실행한다.

**트리거**: 구현 계획이 있고 서브에이전트를 사용할 수 있는 환경에서 실행할 때.

---

## using-git-worktrees

피처 작업 전에 격리된 워크트리를 생성한다. 이미 격리된 환경이면 생성을 건너뛴다. 워크트리 생성 후 의존성 설치와 클린 베이스라인 테스트를 실행한다.

**트리거**: 피처 작업 시작 전, 구현 계획 실행 전.
