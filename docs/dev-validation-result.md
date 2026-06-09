# dev 검증 결과

## 1. 검증 목적

- Terraform으로 생성한 EKS dev 환경이 실제로 동작하는지 확인한다.
- 분리한 MSA 서비스가 클러스터 내부 통신과 외부 진입점을 모두 만족하는지 확인한다.
- 검증이 끝나면 `terraform destroy`로 회수 가능한 구조인지 확인한다.

## 2. 검증 범위

- EKS 클러스터 생성
- ECR 저장소 생성
- 서비스별 Docker 이미지 빌드 및 푸시
- Helm 기반 Kubernetes 배포
- 게이트웨이 외부 노출
- 클러스터 내부 서비스 라우팅

## 3. 실제 구성 결과

- 클러스터 이름: `ssd-dev-eks`
- 리전: `ap-northeast-2`
- 네임스페이스: `ssd-dev`
- 외부 진입점: `ssd-api-gateway`
- 내부 서비스:
  - `ssd-auth-service`
  - `ssd-member-service`
  - `ssd-document-service`
  - `ssd-review-service`
  - `ssd-ai-orchestrator`

## 4. 검증 중 확인한 사실

- 각 서비스 Pod는 모두 `Running`, `Ready 1/1` 상태로 기동했다.
- 게이트웨이에서 각 서비스로의 내부 HTTP 라우팅이 정상 동작했다.
- 외부 노출은 처음 `8080` 포트 기준 Classic ELB로 생성되었으나, 사용자 측 접근이 timeout 되었다.
- 게이트웨이 Service 외부 포트를 `80`으로 조정한 뒤 ELB를 통해 `200 OK` 응답을 확인했다.

## 5. 왜 이런 결과가 나왔는가

- Kubernetes `Service type=LoadBalancer`는 AWS에서 기본적으로 외부 로드밸런서를 생성한다.
- 처음에는 ELB 프론트 포트가 `8080`이었고, 내부 NodePort로 트래픽을 전달했다.
- 애플리케이션과 클러스터 내부 경로는 정상인데 외부에서만 timeout이 발생했으므로, 서비스 자체가 아니라 외부 진입 포트 선택이 원인 후보였다.
- 표준 HTTP 포트인 `80`으로 변경한 뒤 바로 응답이 확인되었으므로, 1차 검증 관점에서는 `80` 노출 구성이 더 안정적이었다.

## 6. 현재 남아 있는 후속 작업

- ALB Ingress Controller 설치 및 Ingress 기반 구조로 전환
- External Secrets Operator 설치 후 Kubernetes Secret placeholder 제거
- 실제 기존 SSD 코드 이식
- 공유 DB를 사용하는 상태에서 서비스 경계별 스키마/접속 정책 정리

## 7. 블로그에 적을 핵심 포인트

- "EKS에 올렸다"가 아니라 "실제로 외부에서 진입 가능한 상태까지 검증했다"는 점
- Gateway 하나만 외부 공개하고 나머지 서비스는 ClusterIP로 숨긴 점
- 외부 포트 `8080` timeout을 `80` 포트 전환으로 해결한 트러블슈팅 과정
- ALB, IRSA, External Secrets는 설계에 반영했지만 1차 검증은 최소 구성으로 끝낸 점
