# SSD MSA 구현 로드맵

## 1단계. 조직과 저장소 분리

- GitHub 조직 `capstone-ssd` 아래 MSA 전환용 저장소 생성
- 기존 `ssd-server` 저장소와 완전 분리
- 문서, 인프라, 매니페스트, 서비스 저장소를 각각 독립 관리

## 2단계. 인프라 코드 작성

- Terraform 기반 `ssd-platform-infra` 작성
- 기존 VPC와 RDS 재사용 구조 반영
- EKS 클러스터와 노드 그룹 생성 코드 작성
- ECR, IAM, IRSA, Secrets Manager 기준 추가

## 3단계. 쿠버네티스 배포 구조 작성

- `ssd-k8s-manifests`에 Helm chart 작성
- API Gateway와 서비스 공통 배포 템플릿 정리
- Ingress, Service, Deployment, ConfigMap, ExternalSecret 정의

## 4단계. 서비스 골격 작성

- `ssd-api-gateway` Spring Cloud Gateway 구성
- 각 서비스 Spring Boot 골격 작성
- 헬스체크, 기본 설정, Dockerfile, CI 추가

## 5단계. 배포 파이프라인 작성

- GitHub Actions로 빌드 및 이미지 푸시 작성
- Helm 배포 워크플로 작성
- 환경 변수와 비밀 값 관리 방식 문서화

## 6단계. 검증 및 회수

- Terraform apply로 dev 환경 생성
- 서비스 이미지 빌드 및 배포 확인
- ALB endpoint를 통한 동작 확인
- 검증 종료 후 `terraform destroy`로 회수

## 7단계. 블로그 및 발표 정리

- 주차별 학습 내용과 실제 구현 결과 연결
- 서비스 분리 이유, EKS 구성, 배포 흐름, 시크릿 전략, 트러블슈팅 정리
