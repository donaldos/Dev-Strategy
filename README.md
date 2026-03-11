# 연구개발부 개발 조직 및 시스템 가이드

## 목차

| 번호 | 항목 | 설명 |
|------|------|------|
| 1 | [팀 구조](./01_team_structure.md) | 연구개발부 팀 구성 및 역할 정의 |
| 2 | [개발 환경](./02_dev_environment.md) | Dev · Staging · Production 환경 구성 |
| 3 | [CI/CD](./03_cicd.md) | GitHub Actions + Slack Webhook 구성 |
| 4 | [브랜치 전략](./04_branch_strategy.md) | Git Flow · 네이밍 규칙 · 코드 리뷰 |
| 5 | [협업 툴](./05_collaboration_tools.md) | Jira · Confluence · Slack · Figma |
| 6 | [모니터링 및 장애 대응](./06_monitoring.md) | 클라이언트 · 서버 모니터링 · 장애 대응 프로세스 |

---

## 전체 개발 흐름 요약

```
기획팀 → Jira 이슈 등록
      ↓
개발자 → feature 브랜치 생성 후 개발
      ↓
코드 완성 → Git push
      ↓
Pull Request 생성 → 코드 리뷰
      ↓
GitHub Actions CI 실행 (린트 · 테스트)
      ↓
승인 → develop merge → Dev 자동 배포
      ↓
Staging merge → Staging 자동 배포
      ↓
QA 검증 완료
      ↓
Production 배포 승인 요청 → DevOps 승인
      ↓
Production 배포 완료
      ↓
모니터링 (Firebase · Sentry · Grafana)
```

---

## 역할 요약

| 역할 | 책임 |
|------|------|
| **개발자** | 코드 · Dockerfile · docker-compose.yml · Dev 확인 |
| **QA** | Staging 기능 검증 |
| **DevOps** | 환경 구성 · CI/CD 파이프라인 · Production 배포 승인 |
| **팀 리드** | 코드 리뷰 · PR 승인 · 장애 대응 |

---

> 문서 관련 문의는 Slack **#general** 채널을 이용해주세요.
