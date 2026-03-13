# Git 브랜치 전략 및 워크플로우

## 브랜치 구조

```
main ← staging ← develop ← feature/*
                          ← bugfix/*
main ← hotfix/*
```

| 브랜치 | 역할 |
|--------|------|
| `main` | 실제 배포본 (직접 커밋 금지) |
| `staging` | QA 테스트 환경 |
| `develop` | 개발 통합 브랜치 |
| `feature/*` | 기능별 개발 |
| `bugfix/*` | QA 단계 버그 수정 |
| `hotfix/*` | 운영 중 긴급 버그 수정 |

---

## 워크플로우

### 1. 초기 프레임워크 구성
프로젝트 초기 코드 및 프레임워크를 로컬에서 구성한다.

### 2. main 브랜치 생성 및 초기 커밋
```bash
git init
git add .
git commit -m "init: 초기 프로젝트 세팅"
git remote add origin https://github.com/유저명/저장소명.git
git push -u origin main
```

### 3. develop 브랜치 생성 및 push
```bash
git checkout -b develop
git push -u origin develop
```

### 4. 기능별 브랜치 생성
develop 브랜치를 기반으로 기능별 브랜치를 생성한다.
```bash
git checkout develop
git checkout -b feature/기능명
git push -u origin feature/기능명
```

### 5. feature → develop Pull Request
기능 개발 완료 후 GitHub에서 PR을 생성한다.
- **base**: `develop`
- **compare**: `feature/기능명`
- 팀원 리뷰 후 병합

### 6. 기능별 담당자 일일 커밋/push
각 담당자는 담당 브랜치에 매일 작업 내용을 커밋/push한다.
```bash
git add .
git commit -m "feat: 작업 내용 설명"
git push origin feature/기능명
```

### 7. 기능 완료 후 병합 및 브랜치 정리
작업이 완료된 feature 브랜치는 develop으로 병합 후 삭제하거나 유지한다.
- GitHub PR 병합 후 **"Delete branch"** 로 정리 권장

### 8. staging 브랜치 생성
develop 브랜치에서 staging 브랜치를 생성한다.
```bash
git checkout develop
git checkout -b staging
git push -u origin staging
```

### 9. staging 브랜치에서 QA 진행
staging 환경에서 QA(품질 검증)를 수행한다.

### 10. QA 버그 발생 시 bugfix 브랜치 생성
QA 중 버그 발견 시 staging에서 bugfix 브랜치를 생성하고 수정 후 staging과 develop 양쪽에 반영한다.
```bash
git checkout staging
git checkout -b bugfix/버그명
git push -u origin bugfix/버그명
```
수정 완료 후:
- `bugfix/*` → `staging` PR 병합
- `bugfix/*` → `develop` PR 병합 (이력 동기화)

### 11. QA 완료 후 main 배포
staging QA가 완료되면 main으로 병합하여 배포한다.
- **base**: `main`
- **compare**: `staging`

---

## 운영 중 긴급 버그 (hotfix)
운영 배포 후 긴급 버그 발생 시 develop을 거치지 않고 main에서 직접 처리한다.
```bash
git checkout main
git checkout -b hotfix/긴급버그명
git push -u origin hotfix/긴급버그명
```
수정 완료 후:
- `hotfix/*` → `main` PR 병합 (즉시 배포)
- `hotfix/*` → `develop` PR 병합 (이력 동기화)

---

## 전체 흐름 요약

```
1. main (초기 커밋)
2. develop 분기
3. feature/* 분기 → 개발 → develop PR
4. staging 분기 → QA
5. QA 버그 → bugfix/* → staging + develop 반영
6. QA 완료 → main 배포
7. 운영 긴급 버그 → hotfix/* → main + develop 반영
```
