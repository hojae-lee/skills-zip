---
name: css-layers
description: CSS 작업 시 반드시 사용한다. 이 프로젝트는 CSS Cascade Layers(@layer theme, base, components, utilities)로 스타일을 관리한다. CSS 변수 추가, 컴포넌트 스타일 작성, 유틸리티 클래스 추가, globals.css 수정, packages/ui/src/index.css 수정 등 CSS 파일을 건드리는 모든 작업에 이 스킬을 사용해라. "CSS 추가해줘", "스타일 수정해줘", "토큰 추가해줘", "다크모드 색상", "컴포넌트 스타일", "유틸리티 클래스", "globals.css", "index.css" 등의 요청 시 트리거된다.
---

# CSS Layer 구조

이 프로젝트는 CSS Cascade Layers로 스타일 우선순위를 명시적으로 관리한다. 레이어 순서는 `apps/<app>/src/app/globals.css` 상단에 선언되어 있다.

```css
@layer theme, base, components, utilities;
```

우선순위: `theme` < `base` < `components` < `utilities` < unlayered (뒤로 갈수록 강함)

## 레이어별 역할과 파일 위치

### `@layer theme` — CSS 변수 토큰

**파일:** `packages/ui/src/index.css`

`:root`와 `.dark` 안의 CSS 변수 정의. 브랜드 색상, 시맨틱 색상, 반지름 등 모든 디자인 토큰이 여기 있다. 모노레포 전체 앱이 공유한다.

```css
@layer theme {
  :root {
    --brand-600: oklch(0.488 0.243 275);
    --background: oklch(1 0 0);
    --radius: 0.625rem;
    /* ... */
  }
  .dark {
    --background: oklch(0.145 0 0);
    /* ... */
  }
}
```

새 토큰 추가 시 항상 `@layer theme` 안에, `:root`(라이트)와 `.dark`(다크) 양쪽에 넣는다.

### `@layer base` — 요소 리셋

**파일:** `apps/<app>/src/app/globals.css`

`*`, `body`, `a`, `html` 등 HTML 요소 기본 스타일. 클래스 없이 선택자로만 작성한다.

```css
@layer base {
  * {
    border-color: var(--border);
  }
  body {
    background-color: var(--background);
    color: var(--foreground);
  }
  a {
    color: inherit;
    text-decoration: none;
  }
}
```

### `@layer components` — 컴포넌트 클래스

두 곳에 나뉜다:

- **`packages/ui/src/index.css`** — 모든 앱이 쓰는 공통 컴포넌트 스타일
- **`apps/<app>/src/app/globals.css`** — 해당 앱 전용 컴포넌트 스타일

재사용 가능한 `.card`, `.badge` 같은 클래스가 여기 들어간다. 여러 속성을 조합한 스타일을 하나의 클래스로 묶을 때도 여기다.

### `@layer utilities` — 서비스별 유틸리티

**파일:** `apps/<app>/src/app/globals.css`

```css
@layer utilities {
  .hero-dot-grid {
    background-image: radial-gradient(...);
  }
}
```

## 새 스타일 추가 시 판단 기준

| 추가하려는 것                      | 넣을 레이어       | 넣을 파일                   |
| ---------------------------------- | ----------------- | --------------------------- |
| 새 색상 토큰 (`--color-new`)       | `theme`           | `packages/ui/src/index.css` |
| 다크모드 토큰 오버라이드           | `theme` > `.dark` | `packages/ui/src/index.css` |
| 요소 기본 스타일 (`h1`, `p`)       | `base`            | `apps/<app>/globals.css`    |
| 여러 앱이 공유하는 컴포넌트 클래스 | `components`      | `packages/ui/src/index.css` |
| 앱 전용 컴포넌트 클래스            | `components`      | `apps/<app>/globals.css`    |
| 페이지/섹션 전용 유틸리티          | `utilities`       | `apps/<app>/globals.css`    |

## 패키지 export

`packages/ui/src/index.css`는 `@repo/ui/index.css`로 import한다.

```css
@import "@repo/ui/index.css";
```

`packages/ui/package.json`의 exports에 등록되어 있다:

```json
{ "./index.css": "./src/index.css" }
```
