---
name: project-briefing
description: 프로젝트 현황을 종합 진단하고 docs/에 브리핑 문서를 생성하는 스킬. "프로젝트 현황 브리핑해줘", "프로젝트 상태 체크", "현재 프로젝트 얼마나 잘 되어있어?", "프로젝트 진단해줘", "monorepo 점검", "설정 리뷰해줘" 등의 요청 시 반드시 사용한다. 환경 설정, 아키텍처, 보안, 성능, 개발자 경험, AI/스킬 설정까지 전 영역을 점검해 docs/project-briefing.md 파일로 출력한다.
---

# Project Briefing Skill

현재 프로젝트 전체를 진단해 `docs/project-briefing.md`에 브리핑 문서를 생성한다.

---

## 진단 원칙

**단순히 파일 목록을 나열하지 않는다.** 각 항목에 대해 "잘 되어 있는가 / 문제가 있는가 / 모호한가"를 판단해야 한다. 판단 기준은 이 프로젝트에 존재하는 스킬(`.claude/skills/`), `CLAUDE.md`, 기존 아키텍처 문서를 우선 참조한다.

**우선순위를 매긴다.** 모든 이슈를 동등하게 나열하지 말고, 실제 영향도(보안 취약점 > 빌드 실패 위험 > 개발 불편 > 스타일 불일치)에 따라 High/Medium/Low로 분류한다.

**현재 상태 기반으로 작성한다.** 파일을 직접 읽고 명령어로 확인한 것만 사실로 기록한다. 추측은 "추정" 또는 "확인 필요"로 표시한다.

---

## Step 1: 컨텍스트 수집

아래 파일들을 읽어 프로젝트의 의도와 규칙을 파악한다. 이미 대화 컨텍스트에 있으면 재확인 불필요.

```bash
# 프로젝트 규칙
cat .claude/CLAUDE.md

# 워크스페이스 구조
cat pnpm-workspace.yaml
cat turbo.json
cat mise.toml 2>/dev/null || cat .nvmrc 2>/dev/null || echo "버전 고정 파일 없음"

# 루트 패키지
cat package.json
```

`.claude/skills/` 디렉터리를 확인해 어떤 스킬이 활성화되어 있는지 파악한다 — 스킬이 존재한다는 것은 해당 영역에 팀 컨벤션이 있다는 뜻이다.

---

## Step 2: 전체 구조 스캔

```bash
# 앱 목록 및 패키지 목록
ls apps/ packages/

# 각 앱/패키지의 package.json에서 이름과 의존성 파악
for f in apps/*/package.json packages/*/package.json; do
  echo "=== $f ===" && cat "$f"
done

# 소스 파일 구조 (깊이 제한)
find apps -type f -name "*.ts" -o -name "*.tsx" | sort
find packages -type f -name "*.ts" -o -name "*.tsx" | sort
```

---

## Step 3: 영역별 진단

각 영역을 아래 기준으로 점검한다. 확인 명령어는 참고용이며 실제 구조에 맞게 조정한다.

### 3-1. 개발자 경험 & 툴링

| 항목 | 확인 방법 |
|------|-----------|
| TypeScript strict | `tsconfig.json`에서 `strict`, `noUncheckedIndexedAccess` 여부 |
| ESLint 설정 | flat config(v9) 여부, `no-explicit-any` 설정 |
| Prettier | 공유 설정 유무, `.prettierignore` 유무 |
| pre-commit | Husky/lint-staged 설정 여부 및 실행 내용 |
| CI/CD | `.github/workflows/` 존재 여부 |
| Turborepo 캐싱 | `turbo.json`의 `inputs`, `outputs`, `cache` 설정 적절성 |

```bash
find . -name ".husky" -o -name "lint-staged.config*" | head -5
ls .github/workflows/ 2>/dev/null || echo "CI 없음"
cat turbo.json
```

### 3-2. 아키텍처 일관성

`.claude/skills/monorepo-architecture/SKILL.md`가 존재하면 그 규칙을 기준으로 점검한다.

- 컴포넌트 레이어 구분이 지켜지는가 (`packages/ui` → `src/common/` → `src/app/[domain]/`)
- import 경로 규칙 준수 (`@repo/`, `@/common/`, 상대경로)
- 파일 명명 컨벤션 (PascalCase 컴포넌트, camelCase 훅/유틸)
- `src/lib/`과 `src/common/`의 역할이 명확히 구분되는가

```bash
# 도메인 컴포넌트 구조 확인
find apps -path "*/app/*/components" -type d
find apps -path "*/common/*" -type f
```

### 3-3. 테스트

- 테스트 프레임워크 설정 존재 여부
- 테스트 파일이 소스 파일 옆에 co-locate 되어 있는가
- 실제 테스트 파일 수 vs 소스 파일 수 (커버리지 감각)
- `turbo.json`에서 test task가 빌드에 과도하게 의존하는가

```bash
find apps packages -name "*.test.ts" -o -name "*.test.tsx" | wc -l
find apps packages -name "*.ts" -o -name "*.tsx" | grep -v test | wc -l
```

### 3-4. 보안

- 미들웨어/인증 설정 (`proxy.ts` 또는 `middleware.ts`) 존재 여부와 역할
- 환경변수 처리 방식 (`.env.example` 존재, 하드코딩 여부)
- JWT/인증 로직에서 검증 우회 가능 경로 존재 여부
- `NEXT_PUBLIC_*` 변수가 민감한 값을 노출하지 않는가
- 외부 패키지 의존성에 알려진 취약점 여부 (npm audit 수준)

```bash
find apps -name "proxy.ts" -o -name "middleware.ts"
find apps -name ".env*" | head -10
grep -r "NEXT_PUBLIC_" apps --include="*.ts" --include="*.tsx" -h | sort -u | head -20
# 하드코딩된 시크릿 패턴 확인
grep -rn "secret\|password\|token\|api_key" apps --include="*.ts" -i | grep -v "\.test\." | grep -v "env\." | head -10
```

### 3-5. 성능

- Next.js Image 최적화 (`<Image>` vs `<img>` 사용 현황)
- Turborepo 빌드 캐시 설정이 합리적인가 (`outputs` 범위)
- 번들 크기 우려 (대용량 의존성 직접 import 등)
- `test → ^build` 처럼 불필요한 직렬 의존이 있는가

```bash
grep -r "<img " apps --include="*.tsx" | grep -v "\.test\." | head -10
cat turbo.json | grep -A3 '"test"'
```

### 3-6. 문서 & 지식 관리

- `README.md` 존재 및 실질적 내용 여부
- `docs/` 폴더 내 문서 현황
- `CLAUDE.md`가 실제 작업 지침을 충분히 담고 있는가
- 아키텍처 결정 기록(ADR) 존재 여부

```bash
ls docs/
wc -l README.md 2>/dev/null || echo "README 없음"
```

### 3-7. AI/Skills 설정

- `.claude/skills/`에 존재하는 스킬 목록과 역할
- 스킬과 실제 프로젝트 규칙이 일치하는가 (스킬이 있는데 현실이 다른 경우)
- `CLAUDE.md`가 최신 상태인가

```bash
ls .claude/skills/
```

---

## Step 4: 브리핑 문서 작성

위 진단 결과를 아래 형식으로 `docs/project-briefing.md`에 작성한다. 기존 파일이 있으면 덮어쓴다.

### 문서 형식

```markdown
# 프로젝트 현황 브리핑

> 점검일: YYYY-MM-DD
> 대상: [프로젝트명] — [한 줄 설명]

---

## 1. 전체 구조 개요
[실제 트리 구조 + 각 앱/패키지 역할 설명]

## 2. 기술 스택
[주요 라이브러리/도구 버전 테이블]

## 3. 앱별 구조
[각 앱의 src/ 구조 + 주요 기능 설명]

## 4. 공유 패키지 상세
[각 패키지 역할, exports, 주요 기능]

## 5. 개발자 경험 & 툴링
[Turborepo, lint, format, pre-commit, CI/CD 현황]

## 6. 아키텍처 일관성
[스킬/규칙 대비 실제 코드베이스 준수 현황]

## 7. 보안
[인증, 환경변수, 미들웨어, 취약점 현황]

## 8. 성능
[빌드 캐싱, 이미지 최적화, 번들 관련 현황]

## 9. AI/Skills 설정
[활성 스킬 목록, CLAUDE.md 품질, docs 현황]

## 10. 잘 되어 있는 부분
[표: 항목 | 상태 | 비고]

## 11. 개선 필요 사항

### 🔴 High
[표: 항목 | 내용 | 권장 조치]

### 🟡 Medium
[표: 항목 | 내용 | 권장 조치]

### 🟢 Low
[표: 항목 | 내용 | 권장 조치]

## 12. 미결 사항 (Open Questions)
[설계가 모호하거나 결정이 필요한 항목 — 이슈가 아닌 "결정 필요" 항목]

## 13. 종합 평가
[2-3문장: 전반적 상태 + 가장 시급한 과제]
```

### 작성 기준

- **잘 된 부분**: 의도적으로 잘 설계된 항목. 당연한 것("파일이 존재함")은 포함하지 않는다.
- **High**: 보안 취약점, 빌드 실패 위험, 팀 생산성을 크게 해치는 항목
- **Medium**: 아키텍처 불일치, 누락된 편의 도구, 기술 부채
- **Low**: 스타일 불일치, IDE 편의성, 미래 확장 시 문제가 될 수 있는 것
- **미결 사항**: "잘못됐다"가 아니라 "어느 방향으로 갈지 결정이 필요하다"는 항목

---

## Step 5: 완료 보고

파일 저장 후 주요 발견사항만 3-5줄로 요약해 사용자에게 보고한다. 문서 전체를 다시 출력하지 않는다.
