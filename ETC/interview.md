# ✅ DevOps 및 배포 관련 요약
- CI/CD 개념 설명
    - CI는 Continuous Integration의 약자로, 코드 변경 사항을 자주 통합하고 자동으로 테스트하는 과정입니다.
    - CD는 Continuous Delivery 또는 Continuous Deployment를 의미하며, CI 이후 단계에서 자동으로 배포까지 이어지는 과정을 말합니다. 
    - 이를 통해 코드의 안정성과 배포 효율을 높일 수 있습니다.
    - 예를 들어 GitHub Actions와 AWS/Docker/Vercel을 이용해 커밋 시마다 자동으로 테스트 후 배포한 경험이 있습니다.
- Git, GitHub에서 협업 시 브랜치 전략 (Git flow 등)
- .env 파일을 사용하는 이유
    - .env 파일은 API 키, DB 비밀번호, 포트 번호 등 민감한 정보를 코드에서 분리해 관리하기 위해 사용합니다. 이 방식은 보안성을 높이고, 환경별로 설정값을 분리할 수 있어 운영/개발 환경을 유연하게 나눌 수 있습니다.
    - Git에 커밋하지 않도록 .gitignore에 등록해 사용합니다.

--------------------

- 쿠버네티스(Kubernetes)
    - 쿠버네티스는 컨테이너 오케스트레이션 툴로, 컨테이너화된 애플리케이션을 자동으로 배포, 확장, 관리해줍니다.
    - 여러 대의 서버(노드)를 클러스터로 묶고, 파드(Pod) 단위로 서비스를 배포합니다.
        - 예를 들어 트래픽이 많을 때 자동으로 스케일링하고, 문제가 생기면 자동으로 재시작해주는 기능이 있어 대규모 서비스 운영에 유리합니다.
    - Pod, Service, Deployment 같은 개념
        - Pod : 컨테이너의 묶음 (가장 작은 배포 단위) / 보통 1 Pod = 1 컨테이너이지만, 꼭 그런 건 아님 / 같은 네트워크 공간(IP), 저장소 공유함
        - Service : Pod들에 접근하는 안정된 주소 (LoadBalancer 역할)
            - Pod는 언제든 죽고 새로 만들어질 수 있어 → IP가 바뀜
            - Service는 항상 고정된 IP/도메인 제공 → 클라이언트 입장에서 안정적
            - LoadBalancer / ClusterIP / NodePort 같은 유형이 있음
        - Deployment : Pod의 상태를 관리해주는 컨트롤러
            - 몇 개의 Pod를 유지할지 정의 (예: 항상 3개 유지)
            - Pod가 죽으면 자동으로 다시 생성
            - 버전 관리, 롤링 업데이트, 롤백 등도 지원함
    - ReplicaSet, ConfigMap, Ingress
        - ReplicaSet (RS) : Pod 개수 유지해주는 관리자
            - 같은 Pod를 지정된 개수만큼 복제해서 유지해줌 / Pod가 죽으면 자동으로 새로 생성함 / 단독으로 쓰기보다는 Deployment 내부에서 자동 생성됨
            - 실무 포인트: 보통 Deployment가 ReplicaSet을 포함하므로 따로 정의할 일은 많지 않음
        - ConfigMap : 설정값을 분리해서 관리하는 곳
            - 환경 변수, 설정 파일 등 앱의 설정 정보 저장 / 코드와 설정을 분리해서, 코드 수정 없이 설정만 바꿀 수 있음 / 컨테이너 안에 환경변수나 마운트된 파일 형태로 주입 가능
            -  비유: “앱이 실행되기 전에 설정을 알려주는 비서” → DB 주소, 외부 API 키 같은 거 여기 넣어두면 편함
        - Ingress : 외부 트래픽을 내부 서비스로 연결하는 입구
            - Service만으로는 외부에서 접속하려면 NodePort 등 써야 하고 좀 불편함
            - Ingress는 도메인 기반 라우팅, HTTPS 설정 등 가능
            - 하나의 IP에 여러 경로(도메인)로 여러 서비스 연결 가능
            - 실무 포인트: Ingress Controller (예: NGINX Ingress Controller) 필요함


- Docker
    - 도커(Docker)는 애플리케이션을 컨테이너(Container) 라는 단위로 간편하게 패키징하고 실행할 수 있게 해주는 플랫폼
        - Dockerfile: 컨테이너 환경을 정의하는 코드
        - 이미지: Dockerfile 기반으로 빌드된 '설명서 결과물'
        - 컨테이너: 이미지를 실행한 상태
- Docker vs VM의 차이
    - Docker는 OS 가상화가 아니라 프로세스 수준의 격리
        - VM은 하드웨어 위에 또 하나의 OS(운영체제)를 돌리는 것
            - 무겁다, OS 부팅 시간도 길고, 자원도 많이 잡아먹음 (RAM, CPU 등)
        - 도커는 전체 OS를 또 띄우는 게 아니라, 기존 OS 안에서 필요한 환경만 격리해서 앱을 돌림
            - 커널은 공유하고, 각 컨테이너는 자기만의 파일 시스템, 프로세스, 네트워크를 갖고 있는 것처럼 보임
                - 커널을 공유한다 -> 같은 OS 환경 안에서 돌아간다(지금 돌고 있는 OS의 커널(Linux 커널 등))
            - 실제론 OS 하나에서 여러 격리된 앱이 독립적으로 돌아가는 방식
                - 훨씬 가볍고 빠름 / 컨테이너 실행은 거의 "앱 실행" 수준으로 빠름
    - VM은 무겁고 느림, Docker는 가볍고 빠름

- Docker를 사용해 본 경험과 Dockerfile 작성 방법
    - 유레카 서버 기반 MSA 프로젝트를 Docker로 컨테이너화하고, docker-compose로 통합 관리 및 배포한 경험이 있습니다. 각 마이크로서비스에 맞는 Dockerfile을 작성하여 이미지 생성 및 서비스 간 연결 설정까지 구성했습니다.
    - Dockerfile은 기본적으로 FROM 명령어로 베이스 이미지를 설정하고, WORKDIR, COPY, RUN, CMD 또는 ENTRYPOINT 등을 사용해 애플리케이션을 실행할 수 있는 환경을 구성합니다
    ```
        FROM openjdk:17-jdk-alpine
        WORKDIR /app
        COPY build/libs/myapp.jar app.jar
        EXPOSE 8080
        ENTRYPOINT ["java", "-jar", "app.jar"]
    ```


--------------------




