---
name: web-accessibility
description: Use this skill for any web accessibility (a11y) work — auditing components, fixing accessibility issues, or building accessible UI from scratch. Covers semantic HTML, ARIA, keyboard navigation, focus management, color contrast, and screen reader support. Trigger on requests like "접근성 확인해줘", "a11y 개선", "스크린 리더", "키보드 내비게이션", "aria-label", "포커스 관리", "색상 대비", "웹 접근성", "WCAG", or when reviewing/building any interactive UI component.
---

# 웹 접근성 (a11y)

## 시맨틱 HTML 우선

의미에 맞는 HTML 요소를 사용하면 브라우저와 스크린 리더가 구조를 자동으로 이해한다. `div`/`span`에 role을 붙이기 전에 올바른 태그를 먼저 고려한다.

```tsx
// ❌
<div onClick={handleSubmit}>제출</div>
<div class="nav">...</div>

// ✅
<button onClick={handleSubmit}>제출</button>
<nav>...</nav>
```

| 용도 | 사용할 태그 |
|------|------------|
| 클릭 동작 | `<button>` |
| 페이지 이동 | `<a href="...">` |
| 목록 | `<ul>` / `<ol>` + `<li>` |
| 주요 콘텐츠 영역 | `<main>`, `<nav>`, `<header>`, `<footer>`, `<section>`, `<article>` |
| 테이블 데이터 | `<table>`, `<th>`, `<td>` |

## ARIA

시맨틱 태그로 의미를 전달할 수 없을 때만 ARIA를 추가한다. ARIA는 시맨틱 HTML을 대체하지 않는다.

### 레이블

```tsx
// 아이콘 전용 버튼 — 텍스트 없으면 aria-label 필수
<button aria-label="메뉴 닫기">
  <XIcon />
</button>

// 입력 필드 — placeholder는 레이블 대체 불가
<label htmlFor="email">이메일</label>
<input id="email" type="email" />

// 또는
<input type="email" aria-label="이메일" />
```

### 상태 전달

```tsx
// 토글 버튼
<button aria-pressed={isActive}>북마크</button>

// 확장/축소
<button aria-expanded={isOpen} aria-controls="menu">메뉴</button>
<ul id="menu" hidden={!isOpen}>...</ul>

// 로딩
<div aria-busy={isLoading} aria-live="polite">
  {isLoading ? '불러오는 중...' : content}
</div>

// 에러 메시지 연결
<input aria-describedby="email-error" aria-invalid={!!error} />
<span id="email-error" role="alert">{error}</span>
```

### Live Regions

동적으로 변경되는 콘텐츠를 스크린 리더에 알린다.

```tsx
// 중요 알림 (즉시 읽음)
<div role="alert">{errorMessage}</div>

// 일반 업데이트 (현재 읽기 완료 후)
<div aria-live="polite">{statusMessage}</div>
```

## 키보드 내비게이션

모든 인터랙션 요소는 키보드로 접근 가능해야 한다.

```tsx
// ❌ div는 기본 키보드 접근 불가
<div onClick={handleClick}>클릭</div>

// ✅ button은 Enter/Space 자동 지원
<button onClick={handleClick}>클릭</button>

// 커스텀 인터랙션이 불가피한 경우
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => { if (e.key === 'Enter' || e.key === ' ') handleClick() }}
>
  클릭
</div>
```

### Tab 순서

`tabIndex={0}` — 자연스러운 DOM 순서에 포함  
`tabIndex={-1}` — Tab으로 접근 불가, 프로그래밍 방식으로만 포커스  
`tabIndex={1+}` — 사용 금지 (자연 순서를 깨뜨림)

## 포커스 관리

### 포커스 스타일

`outline: none` 제거 시 반드시 대체 스타일을 제공한다.

```css
/* ❌ */
:focus { outline: none; }

/* ✅ */
:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

Tailwind:
```tsx
<button className="focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:outline-none">
```

### 모달/다이얼로그 포커스 트랩

모달이 열리면 포커스를 모달 내부에 가두고, 닫힐 때 원래 요소로 돌려보낸다.

```tsx
const Modal = ({ isOpen, onClose }) => {
  const previousFocus = useRef<HTMLElement | null>(null)

  useEffect(() => {
    if (isOpen) {
      previousFocus.current = document.activeElement as HTMLElement
      // 모달 내 첫 번째 포커스 가능 요소로 이동
    } else {
      previousFocus.current?.focus()
    }
  }, [isOpen])
}
```

Radix UI, Headless UI 등 접근성이 내장된 라이브러리를 사용하면 포커스 트랩을 자동으로 처리한다.

## 이미지

```tsx
// 의미 있는 이미지 — 내용을 설명
<Image src="/chart.png" alt="2024년 월별 매출 그래프" />

// 장식용 이미지 — alt 빈 문자열 (스크린 리더가 건너뜀)
<Image src="/decoration.png" alt="" />

// 아이콘이 텍스트와 함께 있을 때
<span aria-hidden="true"><StarIcon /></span>
<span>즐겨찾기</span>
```

## 색상 & 시각적 정보

색상만으로 상태를 구분하면 색각 이상자에게 의미가 전달되지 않는다.

```tsx
// ❌ 색상만으로 에러 표시
<input className="border-red-500" />

// ✅ 텍스트/아이콘으로도 표현
<input className="border-red-500" aria-invalid />
<span className="text-red-500">
  <ErrorIcon aria-hidden />
  이메일 형식이 올바르지 않습니다
</span>
```

**색상 대비** — 일반 텍스트 4.5:1 이상, 큰 텍스트(18px+ 또는 bold 14px+) 3:1 이상 (WCAG AA 기준)

## Heading 계층

`h1` → `h2` → `h3` 순서를 유지하고 레벨을 건너뛰지 않는다. 페이지당 `h1`은 하나.

```tsx
// ❌
<h1>페이지 제목</h1>
<h3>섹션</h3>  {/* h2 건너뜀 */}

// ✅
<h1>페이지 제목</h1>
<h2>섹션</h2>
<h3>서브섹션</h3>
```

시각적 크기가 계층과 맞지 않으면 CSS로 스타일만 조정한다.

## 체크리스트

컴포넌트 완성 후 확인한다.

- [ ] 키보드만으로 모든 인터랙션 가능한가
- [ ] 포커스 스타일이 잘 보이는가
- [ ] 이미지에 alt 텍스트가 있는가
- [ ] 폼 입력에 레이블이 연결되어 있는가
- [ ] 동적 콘텐츠 변경이 스크린 리더에 전달되는가
- [ ] 색상 외에 다른 방식으로도 상태를 전달하는가
- [ ] heading 계층이 올바른가

## 접근성 점수 측정

접근성 작업이 끝나면 반드시 점수를 측정하고 결과를 보고한다. 두 가지 방법을 순서대로 실행한다.

### 1. Lighthouse — 페이지 단위 점수

dev 서버가 실행 중이어야 한다. 없으면 먼저 시작한다.

```bash
# dev 서버 확인 후 실행
npx lighthouse <url> \
  --only-categories=accessibility \
  --output=json \
  --output-path=./lighthouse-a11y.json \
  --chrome-flags="--headless"

# 점수만 빠르게 확인
node -e "
const r = require('./lighthouse-a11y.json');
const score = r.categories.accessibility.score * 100;
const audits = r.categories.accessibility.auditRefs
  .map(ref => r.audits[ref.id])
  .filter(a => a.score !== null && a.score < 1)
  .map(a => \`  ✗ \${a.title}: \${a.displayValue || ''}\`);
console.log(\`접근성 점수: \${score}/100\`);
if (audits.length) console.log('개선 필요:\n' + audits.join('\n'));
"
```

목표 점수: **90 이상**. 미달 시 실패 항목을 수정하고 재측정한다.

### 2. axe-core — 컴포넌트 단위 자동 감사

Playwright가 있으면 실행한다. 없으면 설치 후 진행한다.

```bash
npm install -D @axe-core/playwright
# 또는
pnpm add -D @axe-core/playwright
```

```ts
// a11y.test.ts
import { test, expect } from '@playwright/test'
import AxeBuilder from '@axe-core/playwright'

test('접근성 위반 없음', async ({ page }) => {
  await page.goto('http://localhost:3000')
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze()

  if (results.violations.length > 0) {
    console.log(
      results.violations.map(v =>
        `[${v.impact}] ${v.description}\n  → ${v.nodes.map(n => n.target).join(', ')}`
      ).join('\n\n')
    )
  }

  expect(results.violations).toHaveLength(0)
})
```

```bash
npx playwright test a11y.test.ts
```

### 결과 보고 형식

측정 후 아래 형식으로 결과를 정리해서 보고한다.

```markdown
## 접근성 점수 결과

| 항목 | 점수 |
|------|------|
| Lighthouse 접근성 | 94/100 |
| axe-core 위반 | 0건 |

### 수정한 항목
- 아이콘 버튼 aria-label 추가 (3개)
- 폼 입력 label 연결

### 잔여 이슈 (수동 확인 필요)
- 없음
```

점수가 90 미만이거나 axe 위반이 있으면 수정 후 재측정해서 통과할 때까지 반복한다.
