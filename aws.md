# AWS EKS(Amazon Elastic Kubernetes Service)
- AWS에서 제공하는 관리형 Kubernetes 서비스, 쿠버네티스 클러스터를 쉽게 만들고 운영할 수 있도록 AWS가 관리해 주는 서비스
- 자동화된 제어 플레인 관리 (Kubernetes API 서버 & etcd 운영)
- EC2, Fargate 노드에서 워커 노드 실행 가능
- AWS 서비스와 연동 쉬움 (IAM, VPC, ALB, CloudWatch 등)
- 자동 확장(Auto Scaling), 보안 관리, 네트워크 통합 제공

## AWS EKS 클러스터 생성 흐름
1. 클러스터 생성 (Control Plane)
    - AWS 콘솔, AWS CLI, Terraform 등을 사용해 EKS 클러스터를 생성
    - AWS가 자동으로 Kubernetes Control Plane을 관리(API 서버 & etcd 관리)
2. 워커 노드(Node) 추가
    - EC2 또는 AWS Fargate를 사용해 노드 그룹(Node Group) 생성 OR Fargate(서버리스)로 노드 실행
    - 노드가 클러스터에 자동으로 연결됨
3. 네트워크 & 보안 설정
    - VPC, 서브넷, 보안 그룹(Security Group) 설정
    - IAM 역할 및 권한 구성 (EKS가 노드를 제어할 수 있도록 설정)
4. kubectl 및 aws-cli 설정
    - kubectl & aws eks update-kubeconfig 명령어로 클러스터 연결
    - kubectl get nodes 로 정상 동작 확인
5. 애플리케이션 배포
    - kubectl apply -f deployment.yaml 명령어로 앱을 EKS에 배포
    - 서비스(Service), Ingress 등을 설정해 외부 접근 가능하도록 구성
6. AWS 서비스 연동
    - ALB(Application Load Balancer)로 외부 트래픽 관리
    - IAM을 활용한 보안 정책 적용
    - CloudWatch로 로깅 및 모니터링

## EKS의 주요 장점
- 완전 관리형 서비스(마스터 노드 관리 불필요) → 제어 플레인을 직접 운영할 필요 없음, AWS에서 자동 운영
- 자동 확장 (Auto Scaling) → 필요할 때만 노드 추가 가능
- AWS 서비스와 강력한 연동 → ALB, IAM, CloudWatch, RDS, VPC 네트워크 통합
- 보안 강화 → IAM 기반 인증, VPC 내부 배포 가능
- EC2 & Fargate 지원 → 서버리스 옵션(Fargate)으로 비용 절감 가능
    - Fargate 지원 (서버리스 Kubernetes) → 컨테이너만 실행, 노드 관리 불필요


- *) Rancher 설치
    - Rancher는 SSL 인증서 발급을 위해 cert-manager라는 도구를 사용합니다. cert-manager는 자동으로 SSL/TLS 인증서를 관리하는 Kubernetes 애플리케이션입니다. Rancher를 설치하기 전에 cert-manager를 먼저 설치해야 합니다.
    - LB생성 후 랜처 설치
    ```
        1. helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
        1-1. helm repo update
        2. kubectl create namespace cattle-system
        --> 로드밸런스 먼저 생성인줄 알았는데 랜처 설치하고 에러나도 무시하고 LB 붙이면 됨.

        3. Rancher 접속용 LoadBalancer 생성
            kubectl apply -f 0.rancher_lb.yaml

            #기존 LoadBalancer 확인
            kubectl get svc -A
            
        4. Rancher 설치
        helm install rancher rancher-latest/rancher --namespace cattle-system   --set hostname="{로드밸런스 IP}" --set replicas=1 --set bootstrapPassword=admin --set ingress.tls.source=secret
    ```
    - 랜처 설치 후 LB붙이기
    ```
       1. helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
        2. kubectl create namespace cattle-system
        3. helm install rancher rancher-latest/rancher --namespace cattle-system --set replicas=1 --set boo:tstrapPassword=admin --set ingress.tls.source=secret
        4. Rancher 접속용 LoadBalancer 생성
            kubectl apply -f 0.rancher_lb.yaml
            초기 패스워드 확인 법 : kubectl logs -n cattle-system rancher-5c7d9ffdd6-rc6pl | grep "Bootstrap Password:"
            패스워드 변경 admin / rookies1234!
    ```

### Rancher란?
Rancher는 Kubernetes(쿠버네티스) 클러스터를 쉽고 효율적으로 관리할 수 있는 오픈소스 플랫폼,
단순히 쿠버네티스 대시보드(GUI) 역할만 하는 게 아니라, 여러 개의 클러스터를 한 곳에서 통합 관리할 수 있는 기능 제공.

- ✅ Rancher의 핵심 기능
    1. 멀티 클러스터 관리
        - AWS EKS, Google GKE, Azure AKS 같은 매니지드 쿠버네티스 클러스터를 한 곳에서 관리할 수 있음
        - 온프레미스 또는 클라우드에 직접 구축한 베어메탈 Kubernetes도 관리 가능
        - 여러 개의 클러스터를 통합해서 보기 쉽게 관리하는 멀티 클러스터 대시보드 제공
    2. GUI 기반 클러스터 운영
        - 터미널 없이 웹 대시보드에서 Kubernetes 클러스터를 쉽게 관리 가능
        - Pod, 서비스, Ingress, 네임스페이스 등 쿠버네티스 리소스를 클릭 몇 번으로 배포 가능
        - Helm 차트 및 애플리케이션을 쉽게 배포 가능
    3. 보안 및 접근 제어 (RBAC)
        - 역할 기반 액세스 제어(RBAC) → 사용자의 권한을 설정할 수 있음.
        - 특정 유저에게 읽기 전용, 배포 권한, 관리 권한 등을 세분화하여 부여 가능.
        - LDAP, Active Directory, GitHub 같은 외부 인증 시스템과 연동 가능.
    4. CI/CD 및 DevOps 지원
        - GitOps 및 CI/CD 파이프라인 구축 지원.
        - Kubernetes 환경에서 지속적인 배포(Continuous Deployment, CD) 관리 가능.
        - Istio, Linkerd 같은 **서비스 메시(Service Mesh)**와 통합 가능.
    5. 모니터링 및 로깅 기능 내장
        - Prometheus 기반의 클러스터 상태 모니터링 기능 제공.
        - Grafana와 연동하여 리소스 사용량을 대시보드에서 시각적으로 확인 가능.
        - Fluentd, Elastic Stack(ELK) 같은 로깅 시스템과 연동 가능.

- 📌 Rancher를 왜 사용할까?
    - 여러 개의 쿠버네티스 클러스터를 한 곳에서 관리하고 싶을 때
    - Kubernetes를 GUI 기반으로 쉽게 운영하고 싶을 때
    - 쿠버네티스 RBAC, 인증 시스템, 보안 정책을 쉽게 관리하고 싶을 때
    - CI/CD, 모니터링, 로깅 시스템을 통합하고 DevOps 환경을 구축할 때



### ERROR
- AlreadyExistsException: Stack [eksctl-cluster-cluster] already exists
    - AWS cloudformation에서 오류난 Stack들 삭제

- "Resource handler returned message: \"The maximum number of VPCs has been reached.
    - VPc 삭제~~

- helm install rancher rancher-latest/rancher --namespace cattle-system~~~ 했는데 Error: INSTALLATION FAILED: cannot re-use a name that is still in use 
    ```
    helm list -A
    helm uninstall rancher -n cattle-system
    ```


#### AWS ECR (Elastic Container Registry)란?
- **ECR(Amazon Elastic Container Registry)**는 AWS에서 제공하는 컨테이너 이미지 저장소
    - 쉽게 말하면, Docker Hub 같은 이미지 저장소의 AWS 버전이라고 생각하면 돼!
- 쿠버네티스나 ECS에서 컨테이너를 배포하려면 **컨테이너 이미지(Docker 이미지)**가 필요, 이 이미지를 안전하게 저장하고 관리할 수 있도록 AWS에서 제공하는 Docker 이미지 저장소가 ECR!

- ✅ ECR의 주요 기능
    1. 컨테이너 이미지 저장 및 관리
        - Docker 이미지, OCI(Open Container Initiative) 이미지 저장 가능
        - AWS IAM(Identity and Access Management) 기반으로 이미지 접근 제어 가능
    2. AWS 서비스와 완벽 연동
        - ECS(Amazon Elastic Container Service), **EKS(Amazon Elastic Kubernetes Service)**와 쉽게 연동 가능
        - AWS Lambda와도 연동 가능 (컨테이너 기반 Lambda 함수 실행 시 사용 가능)
    3. 보안 및 접근 제어
        - AWS IAM을 활용한 권한 관리 (특정 사용자 또는 역할(Role)에 따라 이미지 접근 제한 가능)
        - VPC 엔드포인트(VPC Endpoint) 지원 → 인터넷을 거치지 않고 내부 네트워크에서 직접 접근 가능
        - 이미지 스캔 기능 → 보안 취약점(예: CVE) 체크 가능
    4. 서버리스 및 관리형 서비스
        - 직접 서버를 운영할 필요 없이, AWS가 **완전 관리형(Managed Service)**으로 제공
        - 사용한 만큼만 비용을 지불하는 Pay-as-you-go 요금제

- 🔥 ECR 사용 흐름
    - ECR 저장소(Repository) 생성
    - Docker 이미지 빌드 및 태깅 (tagging)
    - AWS ECR에 로그인
    - Docker 이미지 푸시(push)
    - EKS, ECS 같은 서비스에서 이미지 가져와서 배포


##### 추가 설명
- demo1 configmap 추가할 때, volumeMounts에서 파일 가져올 때(EFS에서) mounttarget 오류 발생시,
    - EFS에서 네트워크 관리 > 보안그룹 추가 -> 해당 클러스터 아래 가용영역 abc 추가 -> 서브넷은 로드밸런스가 사용하는 서브넷(ec2 LB) + 보안그룹은 클러스터 default 빼고 클러스터에 연결된 거 추가

- 주키퍼 설치시, PodDisruptionBudget(PDB)은 Kubernetes에서 애플리케이션의 고가용성을 보장하기 위한 설정이에요. 쉽게 말해, 너무 많은 파드(Pod)가 동시에 중단되지 않도록 제한을 걸어주는 기능

- streamsets 계정 = admin/admin

- INGRESS 안되는거 -> http, https사용 안하는건 안됨. (ELASTICSEARCH,POSTGRESQL)

- POSTGRESQL 접속 안될 떄.. annotaions 추가
    - https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/service/annotations/#lb-scheme
    ```
    apiVersion: v1
    kind: Service
    metadata:
    name: postgres-lb
    labels:
        component: postgres
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
    spec:
    type: LoadBalancer
    selector:
        app: postgresql
    ports:
        - name: pgql
        protocol: TCP
        port: 5432
        targetPort: 5432
    ```
