<details>
<summary>초기 로컬 세팅</summary>

1. AWS CLI 설치
  - AWS Command Line Interface(AWS CLI)는 AWS 서비스를 관리하는 통합 도구, 도구 하나만 다운로드하여 구성하면 여러 AWS 서비스를 명령줄에서 제어하고 스크립트를 통해 자동화 가능
2. kubectl & eksctl 설치
  - https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-kubectl.html
      - 쿠버네티스 클러스터에 명령을 실행할 수 있는 커맨드 라인 도구
  - https://eksctl.io/installation/
      - Amazon EKS에서 Kubernetes 클러스터를 생성 및 관리하기 위한 간단한 명령줄 유틸리티인 eksctl
3. aws-cli에서 aws 계정 연결
  - https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/what-is-eks.html
  - aws cli에서 특정 사용자로 로그인하는 방법
      
    ```markdown
    1. 먼저 IAM에서 각 계정에서 access key를 생성해야 함(생성하면 cvs 파일로 다운로드 받을 수 있음)
    2. `aws configure` 명령으로 아래 항목들을 설정한다. access key 발급 받으거를 아래와 같이 입력하면 됨
        AWS Access Key ID [****************6LW5]: A
        AWS Secret Access Key [****************JEwz]: t
        Default region name: [us-east-1]
        Default output format: [json]
        
    3. `aws configure list`로 정보 확인
    ```

</details>

<details>
<summary>EKS 구성</summary>

1. EKS 구성(클러스터 생성)
```
eksctl create cluster --name hyeon-cluster --region us-east-1 --without-nodegroup
# 버전명시
eksctl create cluster --name 클러스터 이름 --region 사용 리전 --version 버전 (선택) --without-nodegroup
```

2. eksctl에서 eks 조작을 위해 context 생성 : eksctl로 생성한 경우 context가 자동으로 추가됨
`aws eks update-kubeconfig --region 사용 리전 --name 사용할 클러스터 이름`
    - 현재 context 확인 코드
     `kubectl config current-context`

    - aws eks update-kubeconfig(미니큐브일 떄)
      - 현재 시스템의 Kubernetes 설정 파일 (~/.kube/config) 을 업데이트해서, 해당 클러스터에 kubectl이 접근할 수 있도록 함.


3. EKS node group 생성(aws 콘솔)
    ```markdown
    IAM role 생성 (AmazonEKSNodeRole) -> 콘솔에서 GUI로 추가 가능
                - AmazonEKSWorkerNodePolicy
                - AmazonEC2ContainerRegistryReadOnly
                - AmazonEKSServicePolicy
                - AmazonEKS_CNI_Policy
    -> IAM 역할에서 AmazonEKSNodeRole 해당 역할에 위의 정책들이 있는지 확인
    ```
    - eksctl로도 node group 생성 가능(비추천)
        
        ```
          eksctl create nodegroup \
              --cluster cluster이름 \
              --name nodegroup이름\
              --node-type t3.medium \
              --nodes 2 \
              --nodes-min 1 \
              --nodes-max 3 \
              --managed \
              --region 리전명
        ```
        
4. EFS CSI 드라이버 설치
    - 콘솔
        - EKS → 추가 기능 → 추가 기능 가져오기 →  EFS CSI 드라이버 선택 후 생성
    - eksctl 가능은 함
        ```
        ## CSI 드라이버가 설치되지 않은 경우 설치 필요함
        kubectl get csidrivers
        helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
        helm repo update
        helm upgrade --install aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
          --namespace kube-system \
          --set image.repository=602401143452.dkr.ecr.us-west-2.amazonaws.com/eks/aws-efs-csi-driver
        kubectl get pods -n kube-system | grep efs
        ```
        
5. ECR(Container Registry) 생성 & 이미지 push : [615299753054.dkr.ecr.ap-northeast-2.amazonaws.com](http://615299753054.dkr.ecr.ap-northeast-2.amazonaws.com/)
    - push 방법
        1. **AWS ECR(Amazon Elastic Container Registry)** 에 **Docker로 로그인 → AWS에 있는 Docker 이미지 저장소에 접근하기 위해 인증하는 과정**
            
            `aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin [777205220558.dkr.ecr.ap-northeast-2.amazonaws.com](http://777205220558.dkr.ecr.ap-northeast-2.amazonaws.com/)`
            
            ⇒ 결과 : 로그인 성공 
            
            - 주요 포인트
                - `-username`은 무조건 `AWS` (고정값)
                - `-password-stdin`으로 토큰을 stdin으로 받음 (보안상 안전)
        2. 도커에 이미지 생성
            
            `docker tag {develop/dsp-svc-control:latest} [777205220558.dkr.ecr.ap-northeast-2.amazonaws.com/{develop/dsp-svc-control:latest}](http://777205220558.dkr.ecr.ap-northeast-2.amazonaws.com/%7Bdevelop/dsp-svc-control:latest%7D)`
            
        3. 이미지 푸시
            - 주의 : repository를 미리 만들어 둬야 이미지 push가 가능함  ex) "develop/dsp-svc-control”
            
            `docker push [615299753054.dkr.ecr.ap-northeast-2.amazonaws.com/{develop/dsp-svc-control:latest}](http://615299753054.dkr.ecr.ap-northeast-2.amazonaws.com/%7Bdevelop/dsp-svc-control:latest%7D)`
            
6. EFS 생성
    > EFS 와 PVC 설정 부분을 자동화 진행 코드 존재(파이썬)
    아래쪽 “EFS 세팅 및 PVC” 부분 참고
    > 
    - aws 콘솔에서 efs 생성
    - 생성 후, EFS에 access point 설정 필요 : EFS Acces Point 생성정보.yaml 참조
        - `kubectl apply -f createpv.yaml` (예시)
            
            ```
            apiVersion: v1
            kind: PersistentVolume
            metadata:
              name: streamsets-data-pv
              labels:
                type: streamsets-data-pv
            spec:
              capacity:
                storage: 1Gi
              volumeMode: Filesystem
              accessModes:
                - ReadWriteMany
              persistentVolumeReclaimPolicy: Retain
              storageClassName: efs-sc
              csi:
                driver: efs.csi.aws.com
                volumeHandle: efs아이디:: Ecr해당이미지url
            ```
            
        
7. EFS 네트워크 > 관리 > eks cluster의 보안그룹을 추가해준다.
    - VPC: 사용중인 클러스터의 VPC
        - 해당 VPC에서 사용중인 서브넷 확인 필요 (a~d)
    - 탑재대상 추가
        - 사용중인 가용 영역 모두 추가
        - 서브넷 ID: EC2에서 각 인스턴스의 서브넷ID 확인하여 동일한 ID 추가
        - IP 주소: 공백
        - 보안그룹: 사용중인 클러스터 이름 들어간 보안그룹들 모두 추가
8. PersistentVolumeClaim 생성
    
    `kubectl apply -f 0.storageClass.yaml`
    
    `kubectl apply -f 1.createpv.yaml`
    
    - createpv 적용 시 확인
        - 각 엑세스 포인트 id가 알맞게 들어갔는지 확인
9. node label 작업
    - node 정보 가져오기
        
        `kubectl get node`
        
    
    ```
    kubectl label nodes {eks 노드1 id} dspESType=master
    kubectl label nodes {eks 노드1 id} dspSparkType=master
    
    kubectl label nodes {eks 노드2 id} dspESType=data1
    kubectl label nodes {eks 노드2 id} dspSparkType=worker
    
    kubectl label nodes {eks 노드3 id} dspESType=data2
    kubectl label nodes {eks 노드3 id} dspSparkType=worker
    kubectl label nodes {eks 노드3 id} dspDBType=postgre
    ```
    
    - node label 잘 붙었는지 확인
    
            `kubectl get nodes --show-labels`

</details>  

<details>
<summary> 인프라 설치</summary>

- ConfigMap
    1. ConfigMap 설정 추가하기
        
        `kubectl apply -f [configMap.yaml]`
        
    2. ConfigMap 정보 확인하기
        
        `kubectl get configmap`
            
    
- ALB(AWS-LoadBalancer)를 사용한 ingress구현
    1.  **OIDC 공급자와 연결되어 있는지 확인 -> URL이 출력되면 연결되어 있는거!**
        
        `aws eks describe-cluster --name {클러스터이름} --query "cluster.identity.oidc.issuer" --output text`
        
        - **OIDC가 뭐지?**
            - OpenID Connect의 줄임말로, 현대적인 인증(Authentication)과 사용자 정보 확인(Identity)을 다루는 표준 프로토콜
                - oauth 2.0 기반 프로토콜 -> eks들이 efs등의 aws서비스 접근 권한 가질 수 있도록 도와줌
    2. **IAM 설정 + OIDC Provider 연결 / eks 외부 서비스에 접근할 수 있는 인증 기반 생성**
        
        `eksctl utils associate-iam-oidc-provider --region {리전명} --cluster {클러스터이름} --approve`
        
    3. **IAM 정책 파일 다운로드, 저장**
        
        `curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/install/iam_policy.json`
        
    4. **b에서 다운받은  json 파일 기반으로 IAM 정책 생성**
        
        `aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://iam_policy.json`
        
    5. **EKS에서 IRSA 방식을 사용하여 aws-load-balancer-controller AWS 리소스를 권한있게 사용할 수 있도록 IAM 역할 연결 -> d에서 만든 IAM 정책을 내 클러스터가 사용할 수 있도록  연결**
        
        `eksctl create iamserviceaccount \
        --cluster={클러스터이름} \
        --namespace=kube-system \
        --name=aws-load-balancer-controller \
        --attach-policy-arn=arn:aws:iam::777205220558:policy/AWSLoadBalancerControllerIAMPolicy \
        --override-existing-serviceaccounts \
        --region {리전명} \
        --approve`
        
    6. **ALB Ingress Controller 설치 & 실행 (Helm)**
        
        `helm repo add eks https://aws.github.io/eks-charts
        helm repo update`
        
        `helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
        -n kube-system \
        --set clusterName={클러스터이름} \
        --set serviceAccount.create=false \
        --set serviceAccount.name=aws-load-balancer-controller` 
        
    7.  **ALB 컨트롤러가 잘 배포되었는지 확인**
        
          `kubectl get deployment -n kube-system aws-load-balancer-controller`
        
    
    - **ingress 접속(로컬에서만 한다면)**
        - Ingress ip 찾기
        `dig +short k8s-default-demo1ing-c691c37f5b-1078457141.ap-northeast-2.elb.amazonaws.com`
        - 로컬  `sudo vi /etc/hosts`
            
            ⇒ 파일에 Ingress IP랑 Hostname을 추가한 이유는 바로 **도메인 이름으로 서비스에 접근할 수 있게 하기 위해서**
            
            - vi 실행 시 → i를 누르면 Insert로 진입, 맨 마지막 줄에
                
                ```perl
                `<INGRESS-IP>  HOSTS이름(demo1.sumits.cloud,kafkamanager.sumits.cloud,streamsets.sumits.cloud)`
                ```
                
                추가 후 esc > :wq 저장
                
            - `curl <http://demo1.sumits.cloud`로 확인 후 접속
    
    - [**Route53](https://us-east-1.console.aws.amazon.com/route53/v2/home?region=us-east-1)**
        
        Amazon ***Route 53***는 가용성과 확장성이 뛰어난 도메인 이름 시스템(DNS) 웹 서비스
        
        1. Route53에 “호스팅 영역” 이동
        2. “[seoultravel.life](https://us-east-1.console.aws.amazon.com/route53/v2/hostedzones?region=us-east-1#/ListRecordSets/Z01234593VVAOVTHSBEQA)” 클릭 진행
        3. 레코드 생성 클릭
            
            ![image.png](attachment:fa80bd7d-2d97-479e-820d-7ffdbe3e3958:image.png)
            
        4. 아래 사진과 같이 세팅
            
            ![image.png](attachment:02882dfc-ed8a-4bd1-bd05-58bd1af45832:image.png)
            
            - 레코드 이름은 “들어갈 웹사이트 설정”
            - 레코드 유형 CNAME (디폴트는 A레코드, A레코드는 IP주소로 입력해야함)
            - 값은 `kubectl get ingress` 나온 ADDRESS를 입력해야함

- racher
    1. helm에 racher 설치하기 위한 폴더를 추가
    `helm repo add rancher-latest https://releases.rancher.com/server-charts/latest`
    2. eks에 racher 설치할 namespace 생성
    `kubectl create namespace cattle-system`
    3. eks에 racher 설치
    `helm install rancher rancher-latest/rancher --namespace cattle-system --set replicas=1 --set bootstrapPassword=admin --set ingress.tls.source=secret`
    4. racher용 loadbalancer 생성
    `kubectl apply -f 0.rancher_lb.yaml`
        - kubectl get svc -A 로 lb의 외부 IP 확인
            - https://외부IP로 racher 접속
    5. 비밀번호 확인(초기 패스워드 확인 법 → admin 안될떄)
        - kubectl get pods -A → racher pod 이름 확인
        
          `kubectl logs -n cattle-system 래처 pod 이름 | grep "Bootstrap Password:”`
        
    6. 비밀번호 초기화(안될 시)
        
        `kubectl -n cattle-system exec -it 래처 pod 이름 -- reset-password`
            
- Volumemount
    
    <aside>
    💡
    > 아래 코드의 경로는 Final_project\yml\infra 에서 진행하였습니다.
    총 4개의 볼륨마운트를 설정해야 한다
    > 
    </aside>
    
    ### kibana
    
    1. yaml  파일 수정
        
        ```yaml
        # 1.demo1.yaml 파일에아래 내용 추가
        # 기본적으로 없을경우 만들고, 받은 코드에는 붙어있음
        spec:
          template:
            spec:
              containers:
                volumeMounts:
                  - name: kibana-config
                    mountPath: /usr/share/kibana/config
                volumes:
                  - name: kibana-config
                    persistentVolumeClaim:
                      claimName: kibana-config-pvc
        ```
        
    2.  배포를 통해 PVC를 볼륨마운트를 연결한다
        
        `kubectl apply -f 1.demo1.yaml`
        
    3. cp 명령어를 통해 복사하기
        
        `kubectl cp volumemount\kibana\config\. [demo1-pod이름]:/usr/share/kibana/config/kibana.yml` 
        
    
    ### es-data-1
    
    1. yaml  파일 수정
        
        ```yaml
        # 1.demo1.yaml 파일에아래 내용 추가
        # 기본적으로 없을경우 만들고, 받은 코드에는 붙어있음
        spec:
          template:
            spec:
              containers:
                volumeMounts:
                  - name: es-data1-config
                    mountPath: /usr/share/data1/config
                volumes:
                  - name: es-data1-config
                    persistentVolumeClaim:
                      claimName: es-data1-config-pvc
        ```
        
    2.  배포를 통해 PVC를 볼륨마운트를 연결한다
        
        `kubectl apply -f 1.demo1.yaml`
        
    3. cp 명령어를 통해 복사하기
        
        `kubectl cp volumemount\es-data-7.10.0-oss\config\. [demo1-pod이름]:/usr/share/data1/config/`
        
    
    ### es-data-2
    
    1. yaml  파일 수정
        
        ```yaml
        # 1.demo1.yaml 파일에아래 내용 추가
        # 기본적으로 없을경우 만들고, 받은 코드에는 붙어있음
        spec:
          template:
            spec:
              containers:
                volumeMounts:
                  - name: es-data2-config
                    mountPath: /usr/share/data2/config
                volumes:
                  - name: es-data2-config
                    persistentVolumeClaim:
                      claimName: es-data2-config-pvc
        ```
        
    2.  배포를 통해 PVC를 볼륨마운트를 연결한다
        
        `kubectl apply -f 1.demo1.yaml`
        
    3. cp 명령어를 통해 복사하기
        
        `kubectl cp volumemount\es-data-7.10.0-oss\config\. [demo1-pod이름]:/usr/share/data1/config/`
        
    
    ### es-master
    
    1. yaml  파일 수정
        
        ```yaml
        # 1.demo1.yaml 파일에아래 내용 추가
        # 기본적으로 없을경우 만들고, 받은 코드에는 붙어있음
        spec:
          template:
            spec:
              containers:
                volumeMounts:
                  - name: es-master-config
                    mountPath: /usr/share/master/config
                volumes:
                  - name: es-master-config
                    persistentVolumeClaim:
                      claimName: es-master-config-pvc
        ```
        
    2.  배포를 통해 PVC를 볼륨마운트를 연결한다
        
        `kubectl apply -f 1.demo1.yaml`
        
    3. cp 명령어를 통해 복사하기
        
        `kubectl cp volumemount\es-master-7.10.0-oss\config\. [demo1-pod이름]:/usr/share/master/config/`
        
    
    ```yaml
    # 4개의 볼륨마운트를 설정하면 아래와 같음 
    spec:
      ...
      template:
        ...
        spec:
          ...
            volumeMounts: # volumes에서 설정한 pvc를 실제 폴더로 연결
            ...
            - name: kibana-config # volumes에서 설정한 pvc 이름
              mountPath: /usr/share/kibana/config # 실제 pod내 마운트(연동) 된 경로 
            - name: es-master-config
              mountPath: /usr/share/master/config
            - name: es-data1-config
              mountPath: /usr/share/data1/config
            - name: es-data2-config
              mountPath: /usr/share/data2b/config 
          volumes: # 실제 pvc를 해당 pod에 연결해주는 코드
            ...
            - name: kibana-config # 이러한 이름으로 pvc를 pod에서 사용
              persistentVolumeClaim: # 사용할 pvc 이름
                claimName: kibana-config-pvc
            - name: es-master-config
              persistentVolumeClaim:
                claimName: es-master-config-pvc
            - name: es-data1-config
              persistentVolumeClaim:
                claimName: es-data1-config-pvc
            - name: es-data2-config
              persistentVolumeClaim:
                claimName: es-data2-config-pvc
    ```
    - 요약
      1. demo1 포드가 kibana-config PVC를 마운트 -> 즉, demo1 포드 안의 /usr/share/kibana/config 경로는 사실 PVC로 연결된 "외부 디스크"라고 보면 됨.
      2. kubectl cp로 demo1 포드에 파일을 복사 -> PVC 디스크 안으로 kibana.yml 파일이 저장(/usr/share/kibana/config가 PVC와 연결돼 있었기 때문에)
        - demo1 포드 = PVC에 파일을 넣는 용도 (파일을 "업로드"하는 통로)
      3. 이제 이 Kibana Pod도 /usr/share/kibana/config에서 PVC에 저장된 kibana.yml을 읽을 수 있게 됨.


- PostgreSQL
    - `kubectl apply -f 1.postgresql.yaml`
        
        ```perl
        apiVersion: v1
        kind: ReplicationController
        metadata:
          name: postgresql
        spec:
          replicas: 1
          template:
            metadata:
              labels:
                app: postgresql
            spec:
              containers:
              - name: postgresql
                image: 777205220558.dkr.ecr.ap-northeast-2.amazonaws.com/postgres:10.13-alpine
                env:
                  - name: POSTGRES_USER
                    value: postgres
                  - name: POSTGRES_PASSWORD
                    value: "example"
                  - name: POSTGRES_DB
                    value: postgres
                  - name: PGDATA
                    value: /var/lib/postgresql/data
                  - name: TZ
                    value: Asia/Seoul
                ports:
                  - containerPort: 5432
                volumeMounts:
                  - name: postgres
                    mountPath: /var/lib/postgresql/data
        #          - name: timezone
        #            mountPath: /etc/localtime
        #            readOnly: true
              volumes:
                - name: postgres
                  persistentVolumeClaim:
                    claimName: postgre-data-pvc
        #        - hostPath:
        #            path: /etc/localtime
        #          name: timezone
              nodeSelector:
                dspDBType : "postgre"
        ---
        apiVersion: v1
        kind: Service
        metadata:
          labels:
            component: postgres
          name: postgres-cs
        spec:
          ports:
            - port: 5432
              targetPort: 5432
              protocol: TCP
              name: pgql
          selector:
            app: postgresql
          type: ClusterIP
        ```
        
    - `kubectl apply -f 2.postgresql_lb.yaml`
        
        ```perl
        ---
        apiVersion: v1
        kind: Service
        metadata:
          labels:
            component: postgres
          name: postgres-lb
        annotations:
          service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
        spec:
          ports:
            - port: 5432
              targetPort: 5432
              protocol: TCP
              name: pgql
          selector:
            app: postgresql
          type: LoadBalancer
        ```
        
        - https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/service/annotations/#lb-scheme
            
            `annotations:
              service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing`
            
            - 이거를 추가해야 외부에서 접근가능
                - `service.beta.kubernetes.io/aws-load-balancer-scheme`NLB가 인터넷에 연결되는지, 내부에 연결되는지 여부를 지정합니다. 유효한 값은 `internal`, 입니다 `internet-facing`. 지정하지 않으면 기본값은 입니다 `internal`.
    - init.txt 파일내 실행
        
        ```perl
        kubectl exec -it {postgresql이름} -- mkdir /var/lib/postgresql/data ;sleep 5;
        kubectl exec -it {postgresql이름} -- mkdir /var/lib/postgresql/data/ts_temp ;sleep 5;
        kubectl exec -it {postgresql이름} -- chown -R postgres:postgres /var/lib/postgresql/data/ ;sleep 5;
        kubectl cp init.sql {postgresql이름}:/
        kubectl exec -it {postgresql이름} -- psql -U postgres -f "/init.sql" ;sleep 15;\
        kubectl exec -it {postgresql이름} -- psql -U postgres -c "DROP SCHEMA secuiot" ;sleep 5;\
        kubectl exec -it {postgresql이름} -- sh -c "export PGPASSWORD=secuiot1q2w" ;sleep 5;\
        kubectl exec -it {postgresql이름} -- psql secuiot -U secuiot -c "CREATE SCHEMA secuiot AUTHORIZATION secuiot"
        ```
        
        - init.sql 파일 같은 공간에 필요
            
            ```perl
            -- 사용자 생성
            CREATE USER secuiot WITH PASSWORD 'secuiot1q2w';
            
            -- 리눅스에서 사용
            CREATE TABLESPACE ts_temp OWNER secuiot location '/var/lib/postgresql/data/ts_temp';
            
            ------------------------------------------------------------------------------------
            --사용자 default_tablespace 설정
            ------------------------------------------------------------------------------------
            ALTER ROLE secuiot SET DEFAULT_TABLESPACE TO ts_temp;
            
            ------------------------------------------------------------------------------------
            --database 생성 및 default_tablespace 설정
            ------------------------------------------------------------------------------------
            CREATE DATABASE secuiot OWNER secuiot;
            
            ALTER DATABASE secuiot SET DEFAULT_TABLESPACE to ts_temp;
            
            --connect secuiot secuiot
            --------------------------------------------------------------------------------------------------------------
            -- DB를 secuiot로 재접속한다
            -- 0.실행 : secuiot(DB) secuiot(sechema) secuiot(계정)
            --------------------------------------------------------------------------------------------------------------
            CREATE SCHEMA secuiot AUTHORIZATION secuiot;
            
            ```
                
        
- kibana란?
    
    Elasticsearch의 데이터를 시각화하고 분석할 수 있는 대시보드 도구
    
    - Elasticsearch에 저장된 **로그, 메트릭, 트레이스 데이터를** 👉 **차트, 그래프, 대시보드**로 시각화해줌
    - **검색 + 필터링**으로 원하는 정보 빠르게 찾을 수 있음
    - 실시간으로 로그/데이터를 모니터링 가능

</details>       

### EFS 세팅 및 PVC
[makeefs2.py](attachment:4bbb8e59-1747-4756-bb6a-e7c34fc77cfe:makeefs2.py)

<aside>
💡 EFS Access Point 성성정보.yaml 파일과 1.createpv.yaml 파일을 같은 경로로 설정해주세요
***리전 서버 설정 해주세요*** 
</aside>

`python makeefs2.py` 
- 실행 명령어
- cli 명령어 나열
    - deploy로 생성된 pod 삭제
        
        `kubectl delete deploy postgresql`
        