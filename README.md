# GitOps Repository

이 저장소는 ArgoCD를 사용하여 Kubernetes 애플리케이션을 관리하는 GitOps 저장소입니다. Infrastructure as Code 패턴을 사용하여 모든 Kubernetes 리소스를 선언적으로 정의하고 관리합니다.

## 📁 프로젝트 구조

```
.
├── README.md                    # 프로젝트 설명서 (이 파일)
├── CLAUDE.md                    # Claude Code 작업 지침서
├── apps/                        # 애플리케이션 매니페스트 디렉토리
│   ├── traefik/                # Traefik 인그레스 컨트롤러 설정
│   │   ├── kustomization.yaml  # Traefik 리소스 통합 관리
│   │   ├── namespace.yaml      # traefik-system 네임스페이스 정의
│   │   ├── deployment.yaml     # Traefik 컨테이너 배포 설정
│   │   ├── service.yaml        # Traefik LoadBalancer 서비스
│   │   ├── rbac.yaml          # Traefik 권한 설정 (ServiceAccount, ClusterRole, ClusterRoleBinding)
│   │   └── argocd-ingressroute.yaml # ArgoCD UI 접근을 위한 IngressRoute
│   └── monitoring/             # 모니터링 스택 (향후 확장 예정)
│       ├── kustomization.yaml  # 모니터링 리소스 통합 관리
│       └── namespace.yaml      # monitoring 네임스페이스 정의
├── argocd-apps/                # ArgoCD 애플리케이션 정의 디렉토리
│   ├── app-of-apps.yaml       # 루트 애플리케이션 (다른 모든 앱을 관리)
│   ├── traefik-app.yaml       # Traefik 애플리케이션 정의
│   └── monitoring-app.yaml    # 모니터링 애플리케이션 정의
└── infrastructure/             # 핵심 인프라 구성요소 (향후 확장 예정)
```

## 🔧 각 파일 설명

### ArgoCD 애플리케이션 정의 (`argocd-apps/`)

#### `app-of-apps.yaml`
- **목적**: App of Apps 패턴을 구현하는 루트 애플리케이션
- **기능**: 다른 모든 ArgoCD 애플리케이션을 자동으로 관리
- **특징**: 자동 동기화, 자동 정리(prune), 자가 치유(self-heal) 활성화

#### `traefik-app.yaml` 
- **목적**: Traefik 인그레스 컨트롤러를 배포하는 ArgoCD 애플리케이션
- **소스**: `apps/traefik` 디렉토리
- **네임스페이스**: `traefik-system` (자동 생성)

#### `monitoring-app.yaml`
- **목적**: 모니터링 스택을 배포하는 ArgoCD 애플리케이션
- **소스**: `apps/monitoring` 디렉토리  
- **네임스페이스**: `monitoring` (자동 생성)

### Traefik 애플리케이션 (`apps/traefik/`)

#### `kustomization.yaml`
- **목적**: Kustomize를 사용한 리소스 통합 관리
- **포함 리소스**: namespace, deployment, service, rbac, argocd-ingressroute
- **장점**: 리소스 관리 단순화, 일관된 배포

#### `namespace.yaml`
- **목적**: `traefik-system` 네임스페이스 생성
- **기능**: Traefik 관련 모든 리소스의 논리적 분리

#### `deployment.yaml`
- **목적**: Traefik v3.0 컨테이너 배포 설정
- **주요 기능**:
  - HTTP(80), HTTPS(443), 관리 UI(8080) 포트 제공
  - Kubernetes CRD 및 Ingress 지원
  - Let's Encrypt ACME 자동 SSL 인증서 발급
  - 스테이징 환경용 SSL 설정

#### `service.yaml`
- **목적**: Traefik을 외부에 노출하는 LoadBalancer 서비스
- **포트**: 80(HTTP), 443(HTTPS), 8080(관리 UI)

#### `rbac.yaml`
- **목적**: Traefik이 Kubernetes 리소스에 접근하기 위한 권한 설정
- **구성요소**:
  - ServiceAccount: traefik 서비스 계정
  - ClusterRole: 필요한 API 리소스 접근 권한
  - ClusterRoleBinding: 서비스 계정과 역할 바인딩

#### `argocd-ingressroute.yaml`
- **목적**: ArgoCD UI에 외부에서 접근할 수 있도록 하는 Traefik IngressRoute
- **경로**: `/argocd` 프리픽스로 ArgoCD 서버 접근
- **미들웨어**: URL 경로에서 `/argocd` 프리픽스 제거

### 모니터링 애플리케이션 (`apps/monitoring/`)

#### `kustomization.yaml`
- **목적**: 모니터링 관련 리소스 통합 관리
- **현재 상태**: 네임스페이스만 포함 (향후 Prometheus, Grafana 등 추가 예정)

#### `namespace.yaml`
- **목적**: `monitoring` 네임스페이스 생성
- **기능**: 모니터링 도구들의 논리적 분리

## 🤖 Kustomization이란?

Kustomization은 Kubernetes 리소스를 선언적으로 관리하는 도구입니다:

### 주요 특징
1. **템플릿 없는 사용자 정의**: YAML 파일을 직접 수정하지 않고 오버레이 적용
2. **리소스 통합**: 여러 YAML 파일을 하나의 단위로 관리
3. **환경별 설정**: 개발/스테이징/프로덕션 환경별 설정 관리 용이
4. **네이티브 지원**: `kubectl apply -k` 명령으로 직접 적용 가능

### 본 프로젝트에서의 활용
- 각 애플리케이션 디렉토리에서 관련 모든 리소스를 하나로 묶음
- ArgoCD가 Kustomization을 인식하여 자동으로 모든 리소스 배포
- 리소스 관리와 버전 관리 단순화

## 🚀 사용법

### 1. 클러스터 관리 명령어

```bash
# ArgoCD 상태 확인
kubectl get pods -n argocd

# ArgoCD UI 접근 (포트 포워딩)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# ArgoCD 초기 관리자 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 2. ArgoCD CLI 사용법

```bash
# 애플리케이션 수동 동기화
argocd app sync <app-name>

# 애플리케이션 상태 확인
argocd app get <app-name>

# 모든 애플리케이션 목록 조회
argocd app list
```

### 3. 개발 워크플로우

1. `apps/` 디렉토리의 애플리케이션 매니페스트 수정
2. Git 저장소에 변경사항 커밋 및 푸시
3. ArgoCD가 자동으로 변경사항 감지 및 동기화 (자동 동기화 활성화 시)
4. 수동 동기화가 필요한 경우: ArgoCD UI 또는 CLI 사용

## 🔧 주요 설정

### 자동 동기화 정책
- **Automated Sync**: 모든 애플리케이션에서 활성화
- **Prune**: 삭제된 리소스 자동 정리
- **Self Heal**: 드리프트 발생 시 자동 복구

### 네임스페이스 관리
- **CreateNamespace=true**: 대상 네임스페이스가 없으면 자동 생성
- 각 애플리케이션별로 독립적인 네임스페이스 사용

### 보안
- Traefik RBAC: 최소 권한 원칙에 따른 클러스터 접근 권한 설정
- SSL/TLS: Let's Encrypt를 통한 자동 인증서 관리

## 📝 추가 정보

이 저장소는 GitOps 모범 사례를 따르며, 모든 인프라 변경사항을 Git을 통해 추적 가능하도록 설계되었습니다. 새로운 애플리케이션을 추가하거나 기존 설정을 변경할 때는 반드시 Git 워크플로우를 통해 진행하시기 바랍니다.
