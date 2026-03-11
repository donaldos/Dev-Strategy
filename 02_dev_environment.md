# 2. 개발 환경

[← 목차로 돌아가기](./README.md)

---

## 환경 구성 개요

```
Dev → Staging → Production
```

| 환경 | 목적 | 배포 방식 | 담당 |
|------|------|-----------|------|
| **Dev** | 개발 중 기능 테스트 · 수시 배포 | 자동 배포 | CI/CD |
| **Staging** | 출시 전 최종 검증 · 운영과 동일 환경 | 자동 배포 | CI/CD |
| **Production** | 실 서비스 | 수동 승인 후 배포 | DevOps 승인 |

---

## 개발자 책임 범위

```
✅ 개발자가 해야 할 것
──────────────────────
코드 작성
Dockerfile 작성 및 관리
docker-compose.yml 작성 (로컬 개발용)
Dev 서버에서 정상 동작 확인
Pull Request 생성 + 코드 리뷰 참여
```

```
❌ 개발자가 안 해도 되는 것
──────────────────────────
Staging / Production 서버 설정
CI/CD 파이프라인 구성
클라우드 인프라 관리
Production 배포 승인
```

---

## Docker 구성

### docker-compose.yml 용도 구분

| 용도 | 작성 주체 |
|------|-----------|
| **로컬 개발용** (개발자 PC에서 띄우는 것) | 개발자 |
| **서버 운영용** (Staging · Production 환경) | DevOps |

> ⚠️ 개발자가 올리는 docker-compose.yml은 **로컬 및 Dev 환경 기준**입니다.
> Staging · Production용 환경 설정은 DevOps가 별도 관리합니다.

### Dockerfile 표준 합의 항목

DevOps와 초기에 반드시 합의해야 하는 항목입니다.

```
✅ 합의 필요 항목
──────────────────────────────
베이스 이미지 버전 통일
환경변수 명명 규칙
포트 규칙
로그 출력 형식
```

---

## 팀별 환경 구성

### 백엔드 / 프론트엔드

```
Dev → Staging → Production
```

- Staging이 Production과 **DB 스키마 · 인프라 스펙이 동일**해야 의미 있음
- 환경별 설정 파일 분리 필수

```
.env.dev
.env.staging
.env.production
```

---

### 앱 (iOS / Android)

```
Dev → Staging → Production
               ↓
          TestFlight (iOS)
          내부 트랙 (Android)
```

- 앱은 스토어 심사가 있어 웹과 배포 주기가 다름
- Staging = 내부 배포 (TestFlight / 구글 내부 트랙)
- **API 환경과 앱 버전을 매핑**하여 관리

---

### 음성인식 엔진 (클라우드 API)

```
Dev (실험) → Staging (통합테스트) → Production
                                      ├── 모델 v1
                                      └── 모델 v2  ← A/B 테스트
```

- 모델 배포는 코드 배포와 별도로 **모델 버전**이 추가 변수로 존재
- Production에서 여러 모델 버전 동시 서빙 가능
- MLflow · BentoML 등 **모델 레지스트리** 도입 권장

---

## 환경별 역할 분담

```
개발자   → 코드 + Dockerfile + docker-compose.yml + Dev 확인
QA       → Staging 기능 검증
DevOps   → 환경 구성 + 파이프라인 + Production 배포 승인
```

---

## 실제 배포 흐름

```
개발자
  ├── 코드 작성
  ├── Dockerfile 작성
  ├── docker-compose.yml 작성 (로컬/Dev용)
  ├── 로컬에서 테스트
  ├── Git push → Dev 자동 배포
  └── Dev 서버 동작 확인 후 PR 생성
         ↓
DevOps (CI/CD 파이프라인)
  ├── Staging 자동 배포
  └── Production 승인 후 배포
```

---

## 주의사항

### Staging DB 분리

```
❌ 잘못된 예
Staging이 Production DB를 바라봄
→ 테스트 데이터가 실서비스에 오염

✅ 올바른 예
별도 DB + 마스킹된 데이터 사용
```

### Production 배포 승인 체계

```
Production 배포 승인권자
  ├── DevOps 담당자
  └── 각 팀 리드 (백엔드 · 프론트 · 앱)
      → 본인 팀 서비스는 팀 리드도 승인 가능
```

---

[← 목차로 돌아가기](./README.md) | [다음 : CI/CD →](./03_cicd.md)
