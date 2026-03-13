# 3. CI/CD
작성자: donaldos
작성일: 2026-02-12


[← 목차로 돌아가기](./README.md)

---

## 구성 개요

```
GitHub Actions  → CI/CD 실행 주체
Slack Webhook   → 각 단계 알림 공유
```

---

## 전체 흐름

```
개발자 push / PR 생성
      ↓
GitHub Actions 실행
      ↓
각 단계마다 Slack 알림
      ↓
결과 팀 전체 공유
```

---

## 단계별 GitHub Actions + Slack 알림 설계

| 단계 | 트리거 | GitHub Actions | Slack 알림 |
|------|--------|----------------|------------|
| **코드 push** | feature 브랜치 push | 린트 · 유닛테스트 실행 | ✅ 성공 / ❌ 실패 |
| **PR 생성** | PR open | 린트 · 빌드 · 테스트 실행 | PR 생성 알림 + 결과 |
| **Dev 배포** | develop merge | 자동 빌드 · 배포 | 배포 시작 / 완료 / 실패 |
| **Staging 배포** | staging merge | 자동 빌드 · 배포 | 배포 시작 / 완료 / 실패 |
| **Production 배포** | 수동 승인 | 승인 요청 → 승인 후 배포 | 승인 요청 / 배포 완료 / 실패 |

---

## GitHub Actions 파일 구조

```
.github/
└── workflows/
    ├── ci.yml              # PR 시 린트 · 테스트
    ├── deploy-dev.yml      # Dev 자동 배포
    ├── deploy-staging.yml  # Staging 자동 배포
    └── deploy-prod.yml     # Production 수동 승인 배포
```

### ci.yml (PR 시 자동 실행)

```yaml
name: CI

on:
  pull_request:
    branches: [develop]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: 린트 실행
        run: make lint
      - name: 테스트 실행
        run: make test
      - name: Slack 알림 (성공/실패)
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### deploy-prod.yml (수동 승인 배포)

```yaml
name: Deploy Production

on:
  workflow_dispatch:  # 수동 실행

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # GitHub Environment 승인 설정
    steps:
      - uses: actions/checkout@v3
      - name: Production 배포
        run: make deploy-prod
      - name: Slack 알림
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK_PROD }}
```

---

## Slack 채널 구성

> 알림을 한 채널에 모두 넣으면 노이즈가 심해집니다. 반드시 채널을 분리하세요.

| 채널 | 용도 | 대상 |
|------|------|------|
| `#deploy-dev` | Dev 배포 알림 | 개발자 |
| `#deploy-staging` | Staging 배포 알림 | QA · 개발자 |
| `#deploy-production` | Production 배포 알림 | 전체 |
| `#ci-alerts` | 빌드 · 테스트 실패 알림 | 개발자 |

---

## Slack 알림 메시지 포맷

### 배포 성공

```
✅ [Staging] 백엔드 배포 완료
브랜치  : staging
커밋    : abc1234 - "로그인 API 수정"
배포자  : 홍길동
시간    : 2026-03-11 14:32
```

### 배포 실패

```
❌ [Staging] 백엔드 배포 실패
브랜치    : staging
커밋      : abc1234
실패 단계 : Docker 빌드
로그      : [바로가기 링크]
```

### Production 승인 요청

```
🚀 [Production] 배포 승인 요청
서비스    : 백엔드
배포자    : 홍길동
변경사항  : [PR 링크]
👉 승인하기 : [GitHub Actions 링크]
```

---

## Production 배포 승인 흐름

```
staging → main merge
      ↓
CI/CD 배포 준비 완료
      ↓
Slack #deploy-production 알림
"Production 배포 승인 요청"
      ↓
DevOps or 팀 리드 → GitHub Actions 승인 클릭
      ↓
자동 배포 실행
      ↓
Slack 배포 완료 알림
```

---

## DevOps 역할 명확화

```
✅ DevOps가 해야 할 것
──────────────────────────────
CI/CD 파이프라인 구축 및 관리
GitHub Actions workflow 작성
Slack Webhook 연동 설정
Production 배포 승인

❌ DevOps가 직접 하지 않아도 되는 것
──────────────────────────────
Staging 배포 직접 실행 (CI/CD가 자동 처리)
각 팀 코드 빌드 직접 실행
```

---

[← 목차로 돌아가기](./README.md) | [다음 : 브랜치 전략 →](./04_branch_strategy.md)
