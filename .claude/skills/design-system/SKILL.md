---
name: design-system
description: 디자인 시스템 구축 및 프로젝트 적용 스킬. Design.md가 없으면 인터뷰를 통해 토큰을 만들고, Design.md가 있으면 프로젝트를 분석해서 실제 코드에 입힌다. "디자인 시스템 만들어줘", "톤앤매너 잡아줘", "디자인 입혀줘", "토큰 만들어줘", "컬러 시스템", "타이포그래피", "반응형 설정", "브랜드 색상", "디자인 적용해줘" 등의 요청에 반드시 사용한다. 디자인 관련 작업이라면 이 스킬을 먼저 확인해라.
---

# 디자인 시스템 스킬

## 시작: Design.md 존재 여부 확인

```bash
find .claude/skills/design-system/docs -name "*Design.md" 2>/dev/null
```

- **없으면** → [인터뷰 모드](#인터뷰-모드-디자인-시스템-구축)
- **있으면** → [적용 모드](#적용-모드-프로젝트에-디자인-입히기)

---

## 인터뷰 모드: 디자인 시스템 구축

Design.md가 없을 때 실행한다. 인터뷰로 방향을 잡고, 토큰을 제안하고 확인받은 뒤 Design.md를 만든다.

### 인터뷰 질문 (순서대로)

한 번에 다 묻지 않는다. 대화하듯 하나씩 물어보고 답변을 받은 뒤 다음으로 넘어간다.

**1. 브랜드 방향**

> "이 서비스/브랜드를 표현하는 키워드를 3개 정도 말해주세요. (예: 신뢰감 있는, 가볍고 빠른, 따뜻한)"

답변 기반으로 컬러 톤(채도/명도 방향)과 타이포 무게감 방향을 미리 잡는다.

**2. 레퍼런스**

> "디자인이 마음에 드는 사이트나 앱이 있다면 하나만 말해주세요. 없어도 괜찮아요."

**3. 사용자 · 플랫폼**

> "주 사용자층과 주로 사용하는 플랫폼이 어떻게 되나요? (예: 직장인 / 데스크탑 위주, 일반 소비자 / 모바일 위주)"

- B2B 데스크탑 → 정보 밀도 높게, 타이포 작게
- B2C 모바일 → 여백 넓게, 터치 영역 크게, 타이포 크게

**4. 기존 컬러**

> "이미 정해진 브랜드 컬러가 있나요? 있다면 hex 코드나 색 이름을 알려주세요."

없으면 키워드 기반으로 Claude가 제안하고 확인받는다.

**5. 다크모드**

> "다크모드 지원이 필요한가요?"

필요하면 토큰을 light/dark 두 벌로 구성한다.

---

### 인터뷰 후: 토큰 제안 → 확인 → Design.md 저장

인터뷰 답변 기반으로 토큰 값을 실제 숫자로 채워서 제안한다. placeholder는 쓰지 않는다.

```
# [프로젝트명] 디자인 시스템 제안

## 브랜드 컬러
brand-500: #XXXXXX (선택 이유: 키워드 "..." 반영)
brand 스케일: 50 / 100 / 200 / 300 / 400 / 500 / 600 / 700 / 800 / 900 / 950

## 시맨틱 컬러
success / warning / error / info

## 타이포그래피
...

이 방향으로 진행할까요? 조정하고 싶은 값이 있으면 말씀해주세요.
```

사용자 확인 후 → Design.md 생성.

---

### Design.md 저장 위치

**경로**: `.claude/skills/design-system/docs/[프로젝트명]Design.md`

프로젝트명은 현재 디렉터리명 또는 `package.json`의 `name` 필드를 사용한다.

---

### Design.md 구조 (템플릿)

```markdown
# [프로젝트명] Design System
_generated: YYYY-MM-DD_

## Brand Identity
- **Keywords**: [키워드1], [키워드2], [키워드3]
- **Tone**: [설명]
- **Reference**: [레퍼런스 또는 "없음"]
- **Platform**: [B2B/B2C, 데스크탑/모바일]
- **Dark mode**: [yes / no]

## Color Tokens

### Brand
brand 컬러의 전체 스케일 (50~950). 500이 기준색.

| Token | Value |
| ----- | ----- |
| brand-50 | #XXXXXX |
| brand-100 | #XXXXXX |
| brand-200 | #XXXXXX |
| brand-300 | #XXXXXX |
| brand-400 | #XXXXXX |
| brand-500 | #XXXXXX |
| brand-600 | #XXXXXX |
| brand-700 | #XXXXXX |
| brand-800 | #XXXXXX |
| brand-900 | #XXXXXX |
| brand-950 | #XXXXXX |

### Semantic
고정값. 인터뷰 내용과 무관하게 항상 포함.

| Token | Value | Usage |
| ----- | ----- | ----- |
| success | #22C55E | 성공 상태 |
| warning | #F59E0B | 경고 |
| error | #EF4444 | 에러 |
| info | #3B82F6 | 정보 |

### Neutral
| Token | Value |
| ----- | ----- |
| neutral-50 | #FAFAFA |
| neutral-100 | #F5F5F5 |
| neutral-200 | #E5E5E5 |
| neutral-400 | #A3A3A3 |
| neutral-700 | #404040 |
| neutral-900 | #171717 |

## Typography
- **Font Family**: [폰트명 또는 "시스템 폰트"]
- **Base size**: 16px

| Scale | Size | Weight | Line height | Usage |
| ----- | ---- | ------ | ----------- | ----- |
| display | 36px | 700 | 1.2 | 랜딩 헤드라인 |
| h1 | 28px | 700 | 1.3 | 페이지 제목 |
| h2 | 22px | 600 | 1.4 | 섹션 제목 |
| h3 | 18px | 600 | 1.4 | 카드 제목 |
| body-lg | 16px | 400 | 1.6 | 주요 본문 |
| body | 14px | 400 | 1.6 | 기본 본문 |
| caption | 12px | 400 | 1.5 | 보조 텍스트 |

## Spacing Scale
4px 기반 배수 시스템.

| Token | Value |
| ----- | ----- |
| space-1 | 4px |
| space-2 | 8px |
| space-3 | 12px |
| space-4 | 16px |
| space-6 | 24px |
| space-8 | 32px |
| space-12 | 48px |
| space-16 | 64px |

## Border Radius
| Token | Value | Usage |
| ----- | ----- | ----- |
| radius-sm | 4px | 인풋, 뱃지 |
| radius-md | 8px | 버튼, 카드 |
| radius-lg | 12px | 모달, 큰 카드 |
| radius-full | 9999px | 알약형, 아바타 |

## Shadows
| Token | Value | Usage |
| ----- | ----- | ----- |
| shadow-sm | 0 1px 3px rgba(0,0,0,0.1) | 카드 |
| shadow-md | 0 4px 12px rgba(0,0,0,0.1) | 드롭다운 |
| shadow-lg | 0 8px 24px rgba(0,0,0,0.12) | 모달 |

## Breakpoints
| Name | Value | Target |
| ---- | ----- | ------ |
| sm | 640px | 모바일 가로 |
| md | 768px | 태블릿 |
| lg | 1024px | 작은 데스크탑 |
| xl | 1280px | 데스크탑 |
| 2xl | 1536px | 와이드 |

## Component Decisions
주요 컴포넌트의 디자인 결정사항. 적용 모드에서 채워진다.
```

---

## 적용 모드: 프로젝트에 디자인 입히기

"디자인 입혀줘", "디자인 시스템 적용해줘" 요청 시 실행한다.

### 1. Design.md 읽기

```bash
cat .claude/skills/design-system/docs/*Design.md
```

### 2. 프로젝트 스택 분석

```bash
# Tailwind 여부
cat tailwind.config.* 2>/dev/null | head -20

# 현재 CSS 파일 구조
find . -name "globals.css" -o -name "index.css" -o -name "variables.css" | grep -v node_modules
```

### 3. CSS 레이어에 토큰 주입 (기본)

**`css-layers` 스킬을 참조한다.** CSS 변수는 `@layer theme`에 배치하는 것이 이 프로젝트의 기본 패턴이다.

```css
@layer theme {
  :root {
    /* Brand */
    --color-brand-50: #XXXXXX;
    --color-brand-500: #XXXXXX;
    --color-brand-950: #XXXXXX;

    /* Semantic */
    --color-success: #22C55E;
    --color-warning: #F59E0B;
    --color-error: #EF4444;
    --color-info: #3B82F6;

    /* Neutral */
    --color-neutral-50: #FAFAFA;
    --color-neutral-900: #171717;

    /* Typography */
    --font-size-h1: 28px;
    --font-size-body: 14px;

    /* Spacing */
    --space-4: 16px;

    /* Radius */
    --radius-md: 8px;
  }
}
```

다크모드가 있으면 같은 레이어 안에 이어서 작성한다.

```css
@layer theme {
  :root { ... }

  [data-theme="dark"] {
    --color-neutral-50: #0A0A0A;
    --color-neutral-900: #FAFAFA;
    /* 반전이 필요한 토큰만 재정의 */
  }
}
```

### 4. Tailwind 병행 (tailwind.config 있을 때)

CSS 변수를 Tailwind theme에 연결한다. 값을 중복 정의하지 않고 CSS 변수를 참조하는 방식으로 한다.

```typescript
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      brand: {
        50: 'var(--color-brand-50)',
        500: 'var(--color-brand-500)',
        950: 'var(--color-brand-950)',
      },
      success: 'var(--color-success)',
      warning: 'var(--color-warning)',
      error: 'var(--color-error)',
      info: 'var(--color-info)',
    },
  },
}
```

### 5. 기존 컴포넌트에 토큰 적용

주요 컴포넌트 파일을 찾아서 하드코딩된 색상/크기 값을 토큰으로 교체한다.

```bash
find . -name "Button.*" -o -name "Input.*" -o -name "Card.*" | grep -v node_modules | head -10
```

적용한 결정사항은 Design.md의 **Component Decisions** 섹션에 기록한다.

---

## Design.md 업데이트 원칙

적용 모드에서 새로운 결정을 내릴 때마다 Design.md에 반영한다.

- 토큰 값 조정 → 해당 행 수정
- 새 컴포넌트 결정 → Component Decisions에 추가
- 새 색상 추가 → Color Tokens에 추가

Design.md는 이 프로젝트 디자인의 단일 진실 공급원(single source of truth)이다.
