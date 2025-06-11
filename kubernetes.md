# Kubernetes
1. Docker / Container / Pod
- Docker: 컨테이너를 실행하고 관리할 수 있는 플랫폼. 애플리케이션과 종속성을 패키징하여 이식성이 뛰어난 환경을 제공.
- Container: 독립적으로 실행되는 가벼운 가상화 환경. 애플리케이션과 필요한 라이브러리를 포함하며, OS 커널을 공유해 빠르고 효율적.
- Pod: Kubernetes에서 가장 작은 배포 단위. 하나 이상의 컨테이너를 그룹으로 묶어 동일한 네트워크와 볼륨을 공유하도록 구성됨.

2. 포드를 관리하는 방식
- Deployment: 일반적인 애플리케이션을 배포하는 객체. 원하는 개수의 Pod를 유지하며, 업데이트 및 롤백 기능을 제공. -> 병렬처리
    - 웹 서버, API 서버, 백엔드 서비스 등 **"상태를 유지할 필요 없는 애플리케이션"**을 배포할 때 사용
    - 롤링 업데이트, 롤백 가능 (새 버전 배포 시, 기존 버전 점진적으로 교체)
    - 각 Pod는 독립적 → 특정 Pod가 죽어도 다른 Pod가 계속 실행되므로 영향 없음.
    - 📌 예시: Node.js 백엔드 서버, React/Vue 프론트엔드 앱, Spring API 서버

- StatefulSet: 상태를 유지해야 하는 애플리케이션(예: 데이터베이스)을 배포하는 객체. -> 순차처리
    - 각 Pod에 고유한 ID(고유한 네트워크 식별자 및 스토리지)를 부여해 상태를 유지해야 하는 애플리케이션(예: DB) 실행 및 관리.
    - Pod가 실행되면 그 안의 컨테이너도 실행되고, Pod가 종료되면 그 안의 컨테이너도 종료
    - 💡 StatefulSet이 중요한 이유?
        - 데이터베이스 같은 애플리케이션은 각 인스턴스(Pod)가 고유한 데이터와 설정을 가져야함, 예를 들어 MongoDB, MySQL, Kafka 같은 서비스는 Pod가 재시작돼도 기존 데이터를 유지해야 하는데, StatefulSet이 이걸 보장해줌
    - 📌 예시: PostgreSQL, MySQL, Redis, Kafka, Zookeeper

- DaemonSet: "각 노드에 꼭 필요한 시스템 수준의 서비스"를 실행하기 위한 용도(모든 노드에 특정 Pod를 하나씩 자동 배포), 주로 로그 수집기나 모니터링 에이전트를 배포할 때 사용됨. 
    - 🚀 DaemonSet이 필요한 이유 : 시스템 전반에서 동일한 기능을 수행하는 서비스가 필요할 때!
    - 💡 DaemonSet이 자동으로 특정 Pod를 배포하는 대표적인 예시:
        - ✔ 로그 수집기 → Fluentd, Filebeat (각 노드의 로그를 중앙 서버로 전송)
        - ✔ 모니터링 에이전트 → Prometheus Node Exporter (각 노드의 CPU, 메모리 모니터링)
        - ✔ 네트워크 관리 → Calico, Cilium (네트워크 정책 적용)


3. Pod를 외부에서 접근하는 방식
- ClusterIP: 클러스터 내부에서만 접근 가능
    - Pod는 기본적으로 외부에서 접근할 수 없고, 클러스터 내부에서만 통신 가능.
    - 내부에서 DNS 기반 서비스 디스커버리를 이용해 다른 서비스와 연결할 때 사용됨
    - 외부에서 직접 접근 불가, 다른 Pod에서 http://<서비스이름>:<포트>로 접근 가능.
    ```
    apiVersion: v1
    kind: Service
    metadata:
    name: my-service
    spec:
    selector:
        app: my-app  # my-app 라벨이 붙은 Pod들과 연결
    ports:
        - protocol: TCP
        port: 80      # 서비스가 노출하는 포트
        targetPort: 8080  # 실제 Pod 내부 컨테이너의 포트
    type: ClusterIP  # 기본값 (내부에서만 접근 가능)
    ```    

- NodePort: 각 노드의 특정 포트를 열어 외부에서도 접근 가능.
    - Node의 IP와 할당된 포트를 통해 외부에서 직접 접근 가능.
    - 기본적으로 30000~32767 범위의 포트가 할당됨.
    ```
    apiVersion: v1
    kind: Service
    metadata:
    name: my-service
    spec:
    selector:
        app: my-app
    ports:
        - protocol: TCP
        port: 80
        targetPort: 8080
        nodePort: 31000  # 외부에서 접근할 노드 포트 (자동 할당 가능)
    type: NodePort  # 외부에서 접근 가능
    ```

- LoadBalancer: 클라우드 제공자의 로드 밸런서를 활용해 외부 트래픽을 관리.
    - NodePort와 ClusterIP를 포함하고, 외부에서 더 쉽게 접근할 수 있도록 지원.
    - 클라우드 환경(AWS, GCP, Azure)에서만 사용 가능.
    ```
    apiVersion: v1
    kind: Service
    metadata:
    name: my-service
    spec:
    selector:
        app: my-app
    ports:
        - protocol: TCP
        port: 80
        targetPort: 8080
    type: LoadBalancer  # 클라우드 로드밸런서 사용
    ```

- Ingress: 하나의 진입점으로 여러 서비스(Pod)로 트래픽을 라우팅하는 역할. → 도메인 기반 트래픽 라우팅
    - Ingress 컨트롤러(Nginx Ingress, Traefik 등)를 이용해 하나의 진입점(도메인)으로 여러 서비스에 트래픽을 분배.
    - example.com/api → api-service, example.com/web → web-service 처럼 설정 가능.
    - SSL/TLS 인증서 적용 가능 (HTTPS 지원).
    - Ingress 자체는 Service가 아니고, Ingress Controller가 필요함.
    ```
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: my-ingress
    spec:
    rules:
        - host: myapp.example.com  # 특정 도메인으로 요청 받음
        http:
            paths:
            - path: /api
                pathType: Prefix
                backend:
                service:
                    name: api-service
                    port:
                    number: 80
            - path: /web
                pathType: Prefix
                backend:
                service:
                    name: web-service
                    port:
                    number: 80
    ```

|--|Ingress (Kubernetes)|	Eureka API Gateway (Spring Cloud Gateway, Netflix Zuul)|
|--|--|--|
|주요 기능	|HTTP 트래픽을 여러 서비스(Pod)로 라우팅|	마이크로서비스 간 요청을 프록시 및 로드밸런싱|
|프로토콜	|HTTP/HTTPS	|HTTP, WebSocket|
|부하 분산	|Ingress 컨트롤러(Nginx, Traefik 등)가 담당	|API Gateway 내부적으로 수행|
|서비스 디스커버리	|Kubernetes 자체의 서비스 디스커버리 사용|	Eureka와 연동하여 서비스 찾기|
|SSL/TLS	|기본적으로 지원 (Let's Encrypt 사용 가능)	|Spring Security 등과 연동 필요|
|로드밸런싱	|Ingress 컨트롤러가 수행	|API Gateway가 수행|
|인증/인가	|기본적으로 없음, 별도 설정 필요|	JWT, OAuth 등 지원|
|요청 변환 (Filter)	|제한적 (Nginx 설정)	|강력한 필터 기능 지원 (Spring Cloud Gateway)|





## Minikube

1. 미니큐브 클러스터 실행 확인
    - 먼저 미니큐브 클러스터가 실행 중인지 확인해야 합니다. 미니큐브 클러스터가 실행 중이 아니라면 다음 명령어로 클러스터를 시작할 수 있습니다.
    ```
        minikube start

        kubectl get nodes

        minikube dashboard  # 대시보드
    ```

2. kubectl 설정 확인
    - 미니큐브를 실행하면 자동으로 kubectl이 해당 클러스터를 가리키게 됩니다. kubectl이 제대로 설정되었는지 확인하려면 다음 명령어를 입력해봅니다
    ```
        kubectl config current-context
    ```

3. YAML 파일 준비
    ```
    ---
    # 1. MetalLB 설치
    apiVersion: v1
    kind: ConfigMap
    metadata:
    namespace: metallb-system
    name: config
    labels:
        app: metallb

    data:
    config: |
        address-pools:
        - name: default
        protocol: layer2
        addresses:
        - 192.168.49.100-192.168.49.110
    ---
    # 2. Nginx Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-deployment
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
    ---
    # 3. Nginx LoadBalancer Service
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx-service
    spec:
    selector:
        app: nginx
    ports:
        - protocol: TCP
        port: 80
        targetPort: 80
    type: LoadBalancer
    ```
    ```
    kubectl apply -f deployment.yaml
    ```

4. 리소스 확인
    ```
    kubectl get deployments

    kubectl get pods
    ```

5. 로컬에서 서비스 접근
    ```
    minikube service nginx-service
    ```

6. 리소스 삭제
    ```
    kubectl delete -f deployment.yaml
    kubectl delete -f service.yaml
    ```


-------
- kubectl scale 명령어로 replicas 수정하기
    ```
    kubectl scale deployment nginx-deployment --replicas=3
    ```
- kubectl get으로 상태 확인
    ```
    kubectl get deployment nginx-deployment
    ```



- 기존 LoadBalancer 확인
    ```
    kubectl get svc -A
    ```

## Minikube(로컬도커)를 통해서 Rancher 설정하는 법

#### MetalLB 설정
1. minikube addons enable metallb
2. kubectl apply -f metallb-config.yaml
3. kubectl get services

#### Cert 설치
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
```

#### Rancher 설치
```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher --namespace cattle-system --create-namespace --set replicas=1 --set bootstrapPassword=admin --set hostname=[minikube ip 입력].nip.io
```

#### 실행
```
kubectl port-forward -n cattle-system svc/rancher 8443:443
```

- rancher에서 Cert-manager를 먼저 설치진행하라고 함