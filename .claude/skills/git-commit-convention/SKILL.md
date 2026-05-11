---
name: git-commit-convention
description: Git 커밋 메시지 컨벤션 스킬. 커밋 메시지를 작성하거나 검토할 때 반드시 사용한다. "커밋해줘", "커밋 메시지 작성해줘", "git commit", "어떤 타입 써야 해", "커밋 컨벤션", "커밋 메시지 어떻게 써", "커밋 타입", "feat이야 fix야" 등의 요청 시 반드시 사용한다. 코드 변경 후 커밋 단계에서 항상 이 스킬을 참조한다.
---

# Git 커밋 컨벤션

## 형식

```
<type>: <제목>

[본문 - 선택사항]
```

- 제목은 한글 또는 영어로 작성한다.
- 제목은 50자 이내, 마침표 없이 끝낸다.
- 명령형으로 쓴다. ("추가한다" X → "추가" O, "Added" X → "Add" O)
- 본문이 필요하면 제목과 빈 줄 하나로 구분한다.

---

## 커밋 타입

| 타입 | 언제 쓰나 |
|------|-----------|
| `feat` | 새로운 기능 추가 |
| `fix` | 버그 수정 |
| `refactor` | 기능 변경 없는 코드 개선 |
| `docs` | 문서 수정 (README, 주석 등) |
| `style` | 포맷팅, 세미콜론 등 코드 의미 변화 없는 수정 |
| `test` | 테스트 코드 추가 또는 수정 |
| `chore` | 빌드 설정, 패키지 매니저, 기타 잡무 |
| `perf` | 성능 개선 |
| `ci` | CI/CD 설정 변경 |
| `revert` | 이전 커밋 되돌리기 |

### 타입 선택 기준

- **feat vs refactor**: 사용자가 체감할 수 있는 변화가 있으면 `feat`, 내부 코드만 바뀌면 `refactor`
- **fix vs refactor**: 잘못된 동작을 고치면 `fix`, 동작은 같고 코드만 개선하면 `refactor`
- **chore vs ci**: GitHub Actions, Jenkins 등 CI 파이프라인이면 `ci`, 나머지 설정(package.json scripts, .gitignore 등)은 `chore`
- **style vs refactor**: 자동 포맷터(Prettier)가 바꾼 것 → `style`, 사람이 판단해서 바꾼 구조 개선 → `refactor`

---

## 예시

```
feat: 소셜 로그인 추가
fix: 모달 스크롤 버그 수정
refactor: UserCard props 정리
docs: API 연동 가이드 추가
style: Prettier 포맷 적용
test: useAuth 로그아웃 테스트 추가
chore: eslint-plugin-import 추가
perf: 이미지 lazy loading 적용
ci: PR 체크 워크플로 추가
revert: feat: 다크모드 토글 추가
```

---

## 스코프 (선택)

변경 범위를 명확히 할 때 타입 뒤에 괄호로 추가한다.

```
feat(auth): 이메일 인증 기능 추가
fix(api): 토큰 만료 시 재발급 로직 수정
```

모노레포라면 패키지명을 스코프로 쓴다.

```
feat(web): 알림 센터 UI 추가
chore(ui): 의존성 업데이트
```
