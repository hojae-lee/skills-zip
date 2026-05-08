---
name: web-performance
description: Lighthouse 기반 웹 성능 점검 스킬. 구현 완료 후 최종 검수 시, 또는 성능·최적화 관련 작업 시 반드시 사용한다. "성능 확인해줘", "lighthouse 돌려줘", "최적화해줘", "느린 것 같아", "bundle 크기 줄여줘", "LCP 개선", "Core Web Vitals", "빌드 최적화", "이미지 최적화", 코드 완성 후 점검 요청 시 트리거된다.
---

# Web Performance 점검

Lighthouse를 실행하고 결과를 분석해서 우선순위가 높은 항목부터 실행 가능한 개선안을 제시한다.

## 1단계: 서버 준비

Lighthouse는 실행 중인 서버에 접속해서 측정한다. **프로젝트의 `package.json` scripts를 먼저 확인**하고 빌드·실행 명령을 파악한다.

```bash
# 프로젝트 스크립트 확인
cat package.json | python3 -c "import json,sys; s=json.load(sys.stdin).get('scripts',{}); [print(f'  {k}: {v}') for k,v in s.items()]"
```

일반적인 패턴 (실제 스크립트에 맞게 조정):

```bash
# 프로덕션 빌드 기반 측정 (권장 — dev 서버는 최적화가 비활성화됨)
# build/start 스크립트명은 package.json에서 확인 후 사용
npm run build && npm run start &   # npm
pnpm build && pnpm start &         # pnpm
yarn build && yarn start &         # yarn
sleep 3  # 서버 시작 대기
```

모노레포 환경이라면 측정 대상 앱의 스크립트를 확인한다:

```bash
# 예: pnpm workspace
cat apps/<app-name>/package.json | python3 -c "import json,sys; s=json.load(sys.stdin).get('scripts',{}); [print(f'  {k}: {v}') for k,v in s.items()]"
```

로컬 개발 환경에서 서브도메인을 쓰려면 `/etc/hosts`에 아래 항목이 필요하다:

```
127.0.0.1 <subdomain>.localhost
```

## 2단계: Lighthouse 실행

```bash
# 기본 실행 (JSON + HTML 리포트 생성)
npx lighthouse http://localhost:3000 \
  --output json,html \
  --output-path ./lighthouse-report \
  --chrome-flags="--headless --no-sandbox" \
  --only-categories=performance,accessibility,best-practices,seo

# 서브도메인 기반 측정
npx lighthouse http://acme.localhost:3000 \
  --output json,html \
  --output-path ./lighthouse-report \
  --chrome-flags="--headless --no-sandbox"
```

결과 파일: `./lighthouse-report.report.json`, `./lighthouse-report.report.html`

## 3단계: 결과 분석

JSON 리포트에서 아래 항목을 추출해서 분석한다.

### 카테고리 점수 확인

| 카테고리       | 목표 점수 | 위치 (JSON)                                |
| -------------- | --------- | ------------------------------------------ |
| Performance    | 90+       | `categories.performance.score * 100`       |
| Accessibility  | 95+       | `categories.accessibility.score * 100`     |
| Best Practices | 95+       | `categories['best-practices'].score * 100` |
| SEO            | 90+       | `categories.seo.score * 100`               |

### Core Web Vitals 확인

```bash
# JSON에서 주요 지표 추출
cat lighthouse-report.report.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
audits = data['audits']
metrics = {
  'LCP': audits.get('largest-contentful-paint', {}).get('displayValue'),
  'TBT': audits.get('total-blocking-time', {}).get('displayValue'),
  'CLS': audits.get('cumulative-layout-shift', {}).get('displayValue'),
  'FCP': audits.get('first-contentful-paint', {}).get('displayValue'),
  'TTFB': audits.get('server-response-time', {}).get('displayValue'),
  'Speed Index': audits.get('speed-index', {}).get('displayValue'),
}
for k, v in metrics.items():
    print(f'{k}: {v}')
"
```

**목표 기준 (Good 범위):**

- LCP ≤ 2.5s
- TBT ≤ 200ms (FID/INP 대리 지표)
- CLS ≤ 0.1
- FCP ≤ 1.8s
- TTFB ≤ 800ms

### 실패 항목 추출

```bash
cat lighthouse-report.report.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
failed = [
  (k, v.get('title'), v.get('displayValue'))
  for k, v in data['audits'].items()
  if v.get('score') is not None and v['score'] < 0.9
]
failed.sort(key=lambda x: x[0])
for audit_id, title, value in failed:
    print(f'- [{audit_id}] {title}: {value}')
"
```

## 4단계: Next.js 최적화 항목

점수가 낮은 카테고리에 따라 아래 항목을 우선 점검한다.

### Performance

**이미지 최적화**

- `<img>` 태그 직접 사용 금지 → `next/image` 사용
- `priority` prop: LCP 대상 이미지(뷰포트 상단)에 추가
- `sizes` prop: 반응형 이미지 크기 명시

**폰트 최적화**

- `next/font`로 폰트 로드 → layout shift 방지, 외부 요청 제거
- `display: 'swap'` 설정

**번들 크기**

```bash
# 번들 분석 — 패키지 매니저와 스크립트명은 package.json에서 확인 후 사용
ANALYZE=true npm run build   # npm
ANALYZE=true pnpm build      # pnpm (모노레포: pnpm --filter <app> build)
# 또는
npx @next/bundle-analyzer
```

- 큰 라이브러리는 dynamic import로 지연 로드
- `next/dynamic`으로 클라이언트 전용 컴포넌트 lazy load

**캐싱**

- Server Component 데이터 fetch에 `cache` 옵션 활용
- `revalidate` 설정으로 불필요한 재요청 방지
- `React.cache`로 동일 요청 내 중복 fetch 제거

**렌더링 전략**

- 정적 페이지는 SSG 유지 (동적 데이터 없는 경우)
- `loading.tsx`로 스트리밍 활성화 → TTFB 체감 개선

### Accessibility

주요 항목:

- 이미지 `alt` 텍스트
- 버튼/링크 레이블 (`aria-label`)
- 색상 대비 4.5:1 이상
- 키보드 네비게이션 (focus visible)

### Best Practices

- HTTPS 사용 (프로덕션)
- 콘솔 오류 없음
- 안전하지 않은 JS 패턴 제거 (`document.write` 등)
- 최신 라이브러리 사용 (취약점 있는 버전 업데이트)

### SEO

- `<title>`, `<meta name="description">` 설정
- `robots.txt`, `sitemap.xml` 존재 여부
- 링크에 `href` 속성 존재
- `viewport` meta 태그 설정

## 5단계: 리포트 형식

분석 결과를 아래 형식으로 출력한다.

```
## Lighthouse 결과

### 점수
- Performance: XX / 100
- Accessibility: XX / 100
- Best Practices: XX / 100
- SEO: XX / 100

### Core Web Vitals
- LCP: X.Xs (🟢 Good / 🟡 Needs Improvement / 🔴 Poor)
- TBT: XXXms
- CLS: X.XX
- FCP: X.Xs
- TTFB: XXXms

### 우선 개선 항목 (점수 영향 순)

**P0 — 즉시 수정**
- [audit-id] 문제 설명 → 구체적 수정 방법

**P1 — 다음 릴리즈 전**
- [audit-id] 문제 설명 → 구체적 수정 방법

**P2 — 여유 있을 때**
- [audit-id] 문제 설명 → 구체적 수정 방법
```

## 측정 팁

- **반복 측정**: Lighthouse 결과는 실행마다 약간 다르다. 3회 실행 후 평균을 사용한다.
- **네트워크 조건**: 기본값은 Slow 4G 시뮬레이션이다. 실제 사용자 환경에 맞게 `--throttling-method=provided`로 조정 가능.
- **모바일 vs 데스크톱**: 기본값은 모바일이다. 데스크톱 측정 시 `--preset=desktop` 추가.
- **프로덕션 vs dev**: `next dev`는 최적화가 꺼져 있어서 점수가 낮게 나온다. 항상 `next build && next start`로 측정한다.
