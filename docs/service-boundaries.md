# SSD 서비스 경계 정의

## 1. 공통 원칙

- 외부 공개 API는 우선 API Gateway를 통해 진입한다.
- 내부 서비스는 ClusterIP로 노출하고 Gateway를 통하지 않는 외부 직접 호출은 허용하지 않는다.
- 서비스 간 통신은 REST만 사용한다.
- 데이터 일관성은 우선 동기 호출 기반으로 유지한다.

## 2. 서비스별 책임

| 서비스 | 주요 책임 | 직접 의존성 |
| --- | --- | --- |
| api-gateway | 라우팅, 공통 헤더, CORS | auth, member, document, review, ai |
| auth-service | 인증, JWT, OAuth | Redis, member |
| member-service | 회원 조회/관리 | PostgreSQL |
| document-service | 문서, 폴더, 소유권, 메타데이터 | PostgreSQL, S3, ai, review |
| review-service | 체크리스트, 평가, 리뷰 결과 | PostgreSQL |
| ai-orchestrator | 외부 AI 요청, 응답 매핑 | External AI API |

## 3. 1차 API 경계 예시

### auth-service

- `POST /internal/auth/token`
- `GET /internal/auth/validate`
- `POST /internal/auth/oauth/kakao`

### member-service

- `GET /internal/members/{memberId}`
- `GET /internal/members/me`

### document-service

- `GET /internal/documents/{documentId}`
- `POST /internal/documents`
- `PATCH /internal/documents/{documentId}`

### review-service

- `GET /internal/reviews/{documentId}`
- `POST /internal/reviews/{documentId}/evaluation`

### ai-orchestrator

- `POST /internal/ai/evaluations`
- `POST /internal/ai/summaries`
- `POST /internal/ai/keywords`

## 4. 기존 프로젝트와의 연결

기존 `ssd-server`의 멀티모듈 구조는 아래와 같이 매핑한다.

- `ssd-auth` -> `auth-service`
- `ssd-api`의 member 관련 API -> `member-service`
- `ssd-domain`, `ssd-application`의 document 관련 기능 -> `document-service`
- 평가/체크리스트/요약 결과 중심 기능 -> `review-service`
- `ssd-external`의 외부 AI 연동 -> `ai-orchestrator`

## 5. 과도기 가정

- 기존 로직을 완전히 재구현하지 않고, 1차는 골격과 배포 구조 중심으로 정리한다.
- 서비스별 세부 도메인 로직은 추후 점진 이관 대상으로 본다.
- 현재 레포에서는 운영 가능한 구조와 책임 분리가 명확한 초기 코드를 우선 확보한다.
