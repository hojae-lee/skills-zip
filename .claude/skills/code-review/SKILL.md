---
name: code-review
description: 코드 리뷰 스킬. 코드를 붙여넣거나, 파일 경로를 지정하거나, "방금 수정한 거 리뷰해줘"처럼 말하면 서브에이전트가 자동으로 언어·프레임워크를 감지하고 최적화·버그·사이드 이펙트·개선 방안을 점검한다. 리뷰 전 어떤 코드를 리뷰할지 먼저 안내한다. "코드 리뷰해줘", "이 코드 봐줘", "review this code", "코드 검토해줘", "버그 있어?", "이 코드 괜찮아?", "리뷰 부탁해", "방금 수정한 거 봐줘", "변경사항 리뷰", "review my changes" 등의 요청 시 반드시 사용한다. 한국어·영어 모두 지원.
---

# Code Review

코드를 서브에이전트가 깊이 분석해서 실제 문제와 개선 방안을 보고한다.

## Step 1: 리뷰 대상 파악 (리뷰 전 항상 먼저 수행)

유저 요청에서 리뷰할 코드를 파악한다:

| 입력 방식 | 처리 방법 |
|-----------|-----------|
| 채팅에 코드 직접 붙여넣기 | 붙여넣은 코드 그대로 사용 |
| 파일 경로 명시 | 해당 파일 Read |
| "방금 수정한 거", "변경사항" 등 | `git diff HEAD` 또는 `git diff --staged` 로 파악 |
| PR 번호 | `gh pr diff <번호>` 로 diff 가져오기 |
| 파일 언급 없이 모호한 경우 | `git status`로 변경된 파일 목록 확인 후 판단 |

**리뷰 대상을 파악한 뒤, 서브에이전트 실행 전에 반드시 유저에게 먼저 안내한다:**

```
리뷰 대상:
- 파일: src/api/users.ts, src/hooks/useUsers.ts
- 범위: git diff HEAD 기준 변경된 부분
- 예상 리뷰 항목: [언어] [프레임워크] 코드

리뷰를 시작하겠습니다.
```

유저가 "맞다 / 진행해줘 / ok" 같은 확인 없이도 바로 진행한다. 안내 후 즉시 서브에이전트를 디스패치한다.

## Step 2: 서브에이전트 디스패치

`general-purpose` 타입 서브에이전트를 열고 아래 프롬프트를 사용한다.

`{CODE}` 자리에 Step 1에서 파악한 코드(또는 diff)를 채운다.

---

### [서브에이전트 프롬프트]

You are an expert code reviewer. Review the provided code thoroughly and concisely.

**RESPONSE LANGUAGE**: Reply in Korean if the user's original request was in Korean, English if in English.

---

#### Step 1: 언어 및 환경 감지

- **Language**: TypeScript, JavaScript, Python, Go, Rust, Java, Kotlin, Swift, SQL 등
- **Domain**: frontend / backend / fullstack / mobile / CLI / infra
- **Framework**: React, Next.js, Vue, Express, FastAPI, Django, Spring 등
- **Signals**: import 패턴, 파일명, 설정 파일, API 패턴 등

---

#### Step 2: 관련 가이드라인 적용

감지된 언어/프레임워크에 맞는 가이드라인을 리뷰에 적용한다.

**전체 공통 (General conventions)**
- Arrow function 우선 (`function` 키워드 지양, TypeScript/JS)
- Early return으로 중첩 최소화
- 하드코딩 금지 — 문자열/URL/경로는 상수 또는 env var로
- `const`/`let`만, `var` 금지
- 의미있는 이름 — `a`, `temp`, `data2`, `foo` 금지
- DRY — 중복 로직은 함수로 추출
- 매직 넘버 금지 — 이름 있는 상수 사용
- async/await 우선 (Promise 체인 지양)
- 함수는 한 가지 역할만

**프론트엔드 (React/Next.js)**
- 불필요한 리렌더 방지: useMemo/useCallback 사용 검토
- Next.js App Router: Client Component보다 Server Component 우선
- Prop drilling 2-3단계 초과 시 context 또는 상태관리 도입 검토
- 배럴 임포트 지양 — 직접 경로로 임포트해서 번들 최적화
- 독립적인 fetch는 Promise.all() 병렬 처리 (워터폴 방지)
- 파생 가능한 데이터는 useEffect 대신 직접 계산
- 리스트 key는 안정적인 값 사용 (배열 인덱스 금지)

**백엔드 (API/서버)**
- 입력 검증은 시스템 경계에서만 (사용자 입력, 외부 API)
- 로그에 민감 데이터 노출 금지
- 올바른 HTTP status code 사용
- Mutation은 멱등성(idempotency) 고려
- N+1 쿼리 패턴 확인

**에러 처리**
- API 에러: status code 기반 처리, 문자열 매칭 금지
- 프론트엔드: 컴포넌트 레벨 에러 vs 페이지 레벨 Error Boundary 구분
- 사용자에게 raw 에러 메시지 노출 금지
- 프로덕션 에러는 로깅 처리

**최신 문법 우선, 호환성 차선**
- optional chaining `?.`, nullish coalescing `??`, logical assignment `??=`, `||=`
- `structuredClone()` (manual deep copy 대신)
- `Array.at(-1)` (`array[array.length - 1]` 대신)
- **호환성 이슈가 있다면**: 최신 문법을 강요하지 않고 "최신은 X이지만 현재 타겟 환경에서는 Y 권장" 형태로 안내

---

#### Step 3: 4가지 차원에서 검토

**🐛 버그 (Bugs)**
- 논리 오류, off-by-one, null/undefined 역참조
- Race condition, await 누락
- 데이터 형태에 대한 잘못된 가정

**⚡ 최적화 (Optimization)**
- 불필요한 연산/렌더링
- N+1 쿼리, 누락된 인덱스
- 번들 사이즈, 불필요한 임포트
- 직렬화 가능한 fetch를 병렬로 전환

**💥 사이드 이펙트 & 안전성 (Side Effects & Safety)**
- 공유 상태의 예기치 않은 뮤테이션
- 에러 처리 누락
- 보안 이슈: XSS, SQL injection, 시크릿 노출, CORS
- 호출자에 대한 breaking change

**💡 더 나은 방안 (Better Alternatives)**
- 같은 목표를 달성하는 더 간단한 방법
- 커스텀 코드를 대체할 수 있는 언어/프레임워크 내장 기능
- 확립된 패턴으로의 개선

---

#### Step 4: 출력 형식

**이슈가 없는 경우:**
```
✅ 이상 없음 / All clear

Language: [detected]
Domain: [frontend/backend]
Applied guidelines: [적용된 가이드라인]
Checked: [리뷰한 항목 요약]
```

**이슈가 있는 경우:**
```
Language: [detected]
Domain: [frontend/backend]
Applied guidelines: [적용된 가이드라인]

---

## 🐛 버그 (Bugs)

### [이슈 제목]
**코드**:
```[language]
[문제가 있는 코드 스니펫]
```
**문제**: [왜 문제인지 설명]
**수정**:
```[language]
[수정된 코드]
```
**심각도**: Critical / Important / Minor

## ⚡ 최적화 (Optimization)
[동일 형식]

## 💥 사이드 이펙트 (Side Effects)
[동일 형식]

## 💡 더 나은 방안 (Better Alternatives)
[설명 + 예시 코드]

---
**심각도 요약**: Critical X건 · Important X건 · Minor X건
```

심각도 기준:
- **Critical**: 프로덕션에서 버그나 보안 이슈를 유발 → 즉시 수정
- **Important**: 머지 전 수정 권장
- **Minor**: 선택적 개선 사항

---

**[리뷰할 코드]**

{CODE}

---

## Step 3: 결과 전달

서브에이전트 결과를 유저에게 그대로 전달한다. 추가 설명은 유저가 요청할 때만 덧붙인다.
