# 프로젝트 아키텍처 개요

## 아키텍처 다이어그램


## 아키텍처 설명

이 프로젝트는 Kubernetes 기반 마이크로서비스 아키텍처를 채택하여 확장성, 유연성 및 견고성을 제공합니다. 주요 구성 요소는 다음과 같습니다:

### 인프라
- **AWS EKS**: Kubernetes 클러스터 관리
- **AWS S3**: 사용자 업로드 파일, 정적 콘텐츠, 로그 저장.
- **CloudFront**: 콘텐츠 전송 네트워크(CDN)로 글로벌 성능 향상
- **Route 53**: DNS 및 트래픽 라우팅
- **Load Balancer Controller**: k8s ingress를 위한 Application Load Balancer

### 모니터링 및 로깅
- **Prometheus**: 시스템 메트릭 모니터링
- **Grafana**: 시각화 및 대시보드
- **Loki**: 로그 집계 시스템
- **Promtail**: 로그 수집기
- 
## 클러스터 구성
- **노드 타입**: t3.medium 인스턴스 4개 
- **환경**: 프로덕션 VPC에 단일 클러스터
- **확장**: hpa로 pod 자동 확장, eks 노드 그룹 autoscaling으로 자원 확장.
- **특징**: Prometheus, Grafana, Loki, Promtail 스택 및 EKS 애드온을 활용하여 kubernetes 클러스터를 관리하고 모니터링

## 리소스 선택 이유

### AWS EKS
- **선택 이유**: 관리형 Kubernetes 서비스로 클러스터 관리 복잡성 감소
- **대안**: ECS는 관리가 더 용이하지만 Kubernetes의 에코시스템과 유연성이 부족하다고 생각

### PostgreSQL
- **선택 이유**: 추후 팀프로젝트에서 pgvector를 활용해 벡터 db로도 활용하기 위해서 선택

### 모니터링 스택 (Prometheus, Grafana, Loki, Promtail)
- **선택 이유**: 오픈소스, 추가 비용 x, 포괄적인 모니터링 및 로깅 시스템 등 다양한 대시보드를 통해 높은 가시성 제공, helm 차트 등 이미 정의된 메니페스트 활용시 추가로 작업하고 설정해야 하는 시간 줄어듬

## 고민내용

1. **비용 최적화**: 이렇게 한달만 서버를 운영해도 70만원이라는 budget을 거의 다 소비해야 될 것 같다. 스팟 인스턴스 활용할 수 있을지 검토해보고 비용절감을 위해 dev 환경에서는 노드 인스턴스를 어떻게 구성해야 할 지 아직 더 고민중
2. **멀티 클러스터**: 보안, 장애 복구, 이 외에도 dev, stage, production 환경을 분리하기 위해서 vpc 및 클러스터가 여러개 필요할 것으로 예상됨, 현재는 dev와 stage를 하나의 vpc및 클러스터에서 별도 네임스페이스로 개발하는 것을 생각중
3. **보안 강화**: 애플리케이션 보안, 취약점 검사를 위한 이미지 스캐닝, Sigstore Cosign을 통한 이미지 서명 추가. 이 외에도 아직 해당 아키텍쳐의 보안성이 production level에 적합한지 좀 더 고민이 필요 할 것 같음


### 장점

1. **확장성**: Kubernetes를 통한 자동 확장 및 부하 분산
2. **안정성**: EKS의 관리형 서비스 결합
3. **고가용성**: 여러 가용 영역에 분산된 서비스로 고가용성 보장
4. **모니터링**: Prometheus, Grafana, Loki를 통한 포괄적인 모니터링 및 로깅
5. **유지보수**: ArgoCD를 기반으로 한 CI/CD 파이프라인으로 gitops을 기반의 개발

### 단점

1. **복잡성**: Kubernetes와 MSA의 운영이 복잡함
2. **비용**: EKS 관리 비용 및 모니터링 도구를 위한 고사양 노드 필요
3. **리소스 요구사항**: Prometheus, Grafana, Loki 등이 기본 리소스를 차지하고 있음, 지금은 간단한 React, Spring boot app을 각각 pod 하나씩만 배포 했지만 팀 프로젝트에서는 애플리케이션도 replica가 필요할 것이고, LLM 처리하는 fastapi도 추가 되면 매우 높은 양의 리소스가 요구 될 것으로 보임

