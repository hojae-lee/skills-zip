# Skills Zip

Claude Code 커스텀 스킬 모음입니다. `.claude/skills/` 아래에 위치하며, 요청 내용에 따라 자동으로 트리거됩니다.

## 스킬 목록

| 스킬 | 설명 | 트리거 예시 |
| ---- | ---- | ----------- |
| [turbo-nextjs](#turbo-nextjs) | Turborepo + Next.js 16 모노레포 아키텍처 컨벤션 적용 | "컴포넌트 추가해줘", "페이지 만들어줘", "훅 작성해줘" |
| [css-layers](#css-layers) | CSS Cascade Layers 기반 스타일 관리 | "CSS 추가해줘", "스타일 수정해줘", "다크모드 색상", "globals.css" |
| [design-system](#design-system) | 디자인 시스템 구축 및 프로젝트 적용 | "디자인 시스템 만들어줘", "톤앤매너 잡아줘", "디자인 입혀줘" |
| [frontend-testing](#frontend-testing) | Vitest 기반 UI 테스트 / 유닛 테스트 작성 | "테스트 작성해줘", "컴포넌트 테스트", "훅 테스트" |
| [api-integration](#api-integration) | TanStack Query + fetch 클라이언트, Query Factory 패턴 | "API 연동해줘", "React Query", "fetch 설정", "인터셉터" |
| [error-handling](#error-handling) | API 에러(status 기반)와 UI 에러(Error Boundary) 분리 처리 | "에러 처리해줘", "Error Boundary", "toast 알림", "401 처리" |
| [web-performance](#web-performance) | Lighthouse 기반 성능 점검 및 최적화 | "성능 확인해줘", "lighthouse 돌려줘", "LCP 개선" |

---

## turbo-nextjs

Turborepo + Next.js 16 모노레포에서 코드를 작성하거나 수정할 때 사용한다. 아키텍처 컨벤션, TypeScript 규칙, import 경로, TDD 워크플로를 강제한다.

**트리거**: `apps/` 또는 `packages/` 안의 파일을 추가·수정하는 모든 작업. "컴포넌트 만들어줘", "페이지 추가", "훅 작성", "리팩터링", "테스트 작성" 등.

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

**트리거**: "테스트 작성해줘", "test 추가", "컴포넌트 테스트", "훅 테스트", "spec 작성" 등 테스트 관련 모든 요청.

---

## web-performance

Lighthouse를 실행해 성능 지표(LCP, CLS, FID, TBT)를 측정하고 개선 방향을 제시한다. 구현 완료 후 최종 점검 시 또는 성능 이슈 발생 시 사용한다.

**트리거**: "성능 확인해줘", "lighthouse 돌려줘", "최적화해줘", "bundle 크기 줄여줘", "LCP 개선", "Core Web Vitals" 등.
