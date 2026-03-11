# 4. 브랜치 전략

[← 목차로 돌아가기](./README.md)

---

## 브랜치 전략이란?

**팀원 여러 명이 동시에 개발할 때, Git 브랜치를 어떻게 나누고 관리할 것인가에 대한 규칙입니다.**

```
개발자 A → 로그인 기능 개발 중
개발자 B → 결제 기능 개발 중
개발자 C → 버그 수정 중

→ 규칙 없이 같은 브랜치에 push하면 코드 충돌 · 엉킴 발생
→ 브랜치 전략으로 이를 방지
```

---

## 브랜치 ↔ 환경 매핑

| 브랜치 | 환경 | 배포 방식 |
|--------|------|-----------|
| `feature/*` | 로컬 개발 | 없음 |
| `develop` | Dev 서버 | 자동 배포 |
| `staging` | Staging 서버 | 자동 배포 |
| `main` | Production 서버 | 수동 승인 후 배포 |

---

## 브랜치 종류 및 네이밍 규칙

| 브랜치 | 용도 | 규칙 | 예시 |
|--------|------|------|------|
| `main` | Production 배포 | 고정 | `main` |
| `staging` | Staging 배포 | 고정 | `staging` |
| `develop` | Dev 배포 | 고정 | `develop` |
| `feature/*` | 기능 개발 | `feature/기능명` | `feature/login-api` |
| `bugfix/*` | 버그 수정 | `bugfix/버그명` | `bugfix/login-null-error` |
| `hotfix/*` | 긴급 수정 | `hotfix/이슈명` | `hotfix/payment-crash` |
| `release/*` | 배포 준비 | `release/버전` | `release/v1.2.0` |

---

## 네이밍 세부 규칙

```
✅ 좋은 예
feature/user-login-api
feature/voice-recognition-model
bugfix/token-expiry-error
hotfix/critical-payment-bug

❌ 나쁜 예
feature/홍길동-작업    → 한글 사용 금지
feature/fix            → 너무 모호함
feature/test1          → 의미 없는 이름
my-branch              → 분류 없이 생성
```

### Jira 이슈 번호 연동

```
feature/DEV-123-login-api
bugfix/DEV-456-token-expiry

→ GitHub PR과 Jira 이슈 자동 연결
→ merge 시 Jira 이슈 자동 Done 처리
```

---

## 개발자 작업 흐름

```
1. develop에서 feature 브랜치 생성
   git checkout -b feature/DEV-123-login-api

2. 작업 후 push
   git push origin feature/DEV-123-login-api

3. Pull Request 생성
   → develop 브랜치로 merge 요청
   → 코드 리뷰 → 승인 → merge

4. develop merge 완료
   → Dev 자동 배포
   → Slack #deploy-dev 알림

5. develop → staging merge
   → Staging 자동 배포
   → QA 검증

6. staging → main merge
   → Production 배포 승인 요청
   → DevOps 승인 후 배포
```

---

## hotfix 브랜치 특별 처리

Production 긴급 버그 발생 시 일반 흐름과 다르게 처리합니다.

```
Production 긴급 버그 발생
      ↓
main에서 hotfix 브랜치 생성
git checkout -b hotfix/critical-payment-bug main
      ↓
긴급 수정 후 main으로 직접 merge
→ Production 긴급 배포
      ↓
develop에도 동일하게 merge
→ 코드 동기화 (누락 방지)
```

---

## Pull Request 규칙

### PR 생성 시 필수 포함 사항

```
1. 제목    : [타입] 작업 내용 요약
            [Feature] 사용자 로그인 API 추가
            [Bugfix] 토큰 만료 오류 수정
            [Hotfix] 결제 크래시 긴급 수정

2. 설명    : 무엇을, 왜 변경했는지

3. 테스트  : 어떻게 테스트했는지

4. 스크린샷: UI 변경 시 필수
```

---

## 코드 리뷰 규칙

### 리뷰어 지정 규칙

| 상황 | 리뷰어 |
|------|--------|
| 일반 기능 개발 | 동일 팀 개발자 1명 이상 |
| 핵심 기능 · 아키텍처 변경 | 팀 리드 필수 포함 |
| Hotfix | 팀 리드 즉시 리뷰 |

### 리뷰어 행동 규칙

```
✅ 승인 (Approve)
- 코드에 문제 없음
- merge 가능

⚠️ 수정 요청 (Request Changes)
- 반드시 이유와 개선 방향 명시
- "이렇게 고치세요" 가 아닌
  "이런 이유로 이 방향이 좋을 것 같습니다"

💬 단순 코멘트 (Comment)
- 승인 · 거절 아닌 의견 제시
- 반영 여부는 작성자 판단
```

### merge 가능 조건

```
✅ merge 가능 조건
──────────────────────────────
1. 리뷰어 최소 1명 승인
2. GitHub Actions CI 통과 (빌드 · 테스트 성공)
3. 충돌(Conflict) 없음

❌ merge 금지 조건
──────────────────────────────
1. CI 실패 상태
2. 수정 요청(Request Changes) 미해결
3. 리뷰어 승인 없음
```

---

## 전체 PR 흐름

```
feature 브랜치 작업 완료
      ↓
PR 생성 (제목 · 설명 · 테스트 작성)
      ↓
GitHub Actions 자동 실행 (린트 · 테스트)
      ↓              ↓
    실패            성공
      ↓              ↓
Slack 알림        리뷰어 코드 리뷰
수정 후              ↓         ↓
재push            승인      수정요청
                    ↓         ↓
                  merge     수정 후 재리뷰
                    ↓
                Dev 자동 배포
                Slack 알림
```

---

[← 목차로 돌아가기](./README.md) | [다음 : 협업 툴 →](./05_collaboration_tools.md)
