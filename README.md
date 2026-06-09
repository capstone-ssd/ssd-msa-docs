# SSD MSA 전환 문서

이 저장소는 기존 `ssd-server` 프로젝트를 AWS EKS 기반 MSA 구조로 전환하기 위한 아키텍처 문서, 구현 기준, 운영 메모를 관리한다.

## 목표

- 기존 Java Spring 기반 SSD 서비스를 MSA 구조로 재구성한다.
- AWS EKS 환경에 배포 가능한 Terraform, Helm, GitHub Actions 기준을 문서화한다.
- 기존 프로젝트의 핵심 설정과 도메인 개념을 유지하면서 서비스 경계를 재정의한다.
- 최종적으로 블로그 작성과 발표 자료 작성에 바로 활용할 수 있는 형태로 정리한다.

## 저장소 구성

- `docs/architecture-overview.md`
  전체 아키텍처와 서비스 경계 설명
- `docs/service-boundaries.md`
  각 서비스 책임, API 경계, 데이터 소유 범위 정리
- `docs/implementation-roadmap.md`
  실제 구현 순서와 검증 절차 정리
- `docs/dev-validation-result.md`
  실제 EKS dev 검증 결과와 트러블슈팅 기록

## 기본 원칙

- 기존 `ssd-server` 저장소는 수정하지 않는다.
- 새 MSA 레포는 모두 `capstone-ssd` GitHub 조직에서 독립적으로 관리한다.
- 환경은 우선 `dev` 하나만 사용한다.
- 서비스 간 통신은 우선 REST 기반으로 제한한다.
- 데이터베이스는 기존 RDS를 재사용하되, 장기적으로 서비스별 스키마 분리를 목표로 한다.
- 배포는 `GitHub Actions -> ECR -> Helm -> EKS` 흐름을 사용한다.
- 민감 정보는 Git 저장소에 커밋하지 않고 AWS Secrets Manager를 통해 관리한다.
