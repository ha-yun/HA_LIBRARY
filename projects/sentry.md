# StreamSets
<details>
<summary>StreamSets</summary>

- StreamSets는 데이터 파이프라인을 GUI로 손쉽게 설계하고 실행할 수 있는 ETL 도구
    - ETL = Extract(추출) + Transform(변환) + Load(적재)
    - Kafka, JSON, 파일, DB 등 다양한 소스로부터 데이터를 받아서 가공하고, 원하는 곳으로 보내는 걸 하나의 파이프라인 시각적으로 구성

#### 📦 StreamSets 주요 특징
- GUI 기반 : 드래그 앤 드롭으로 파이프라인 구성 가능
- 실시간 스트리밍 : Kafka, HTTP 등 스트리밍 데이터 처리 가능
- 배치 처리도 OK : 파일, DB처럼 정적인 소스도 처리 가능
- 풍부한 커넥터 : Kafka, S3, JDBC, Elasticsearch, Snowflake 등 다양한 연결 지원
- 변환 처리 : Expression Evaluator, Field Renamer, Field Splitter 등 다양한 프로세서 제공
- 에러 핸들링 : 개별 레코드 처리 중 오류 발생 시 에러 레코드 추적 및 경로 분리 가능

#### 기본 구성요소
1. Origin (입력) : 데이터를 가져오는 시작점
    - Kafka Consumer
    - Directory (로컬 파일)
    - HTTP Client
    - JDBC Query 등

2. Processor (처리) : 데이터 변환/가공 처리
    - Expression Evaluator: 필드 값 계산, 조건 처리
    - Field Remover, Field Renamer, Field Masker
    - JavaScript Evaluator: 복잡한 로직 직접 작성 가능
    - JSON Parser: JSON flatten 등

3. Destination (출력) : 데이터를 보내는 곳
    - Kafka Producer
    - Elasticsearch
    - JDBC Producer
    - S3 등

4. Error Stream : 처리 실패한 레코드가 따로 분리되어 저장되는 곳 (에러 디버깅에 유용!)

#### 기타
- JSON 구조가 복잡하면 Flatten 옵션으로 JSON Parser 사용하기
- 테스트 데이터는 Raw Data Preview 기능으로 바로 확인 가능
- Expression Evaluator에서는 ${record:value('/fieldName')} 이런 식으로 값 접근



</details>


### 백엔드에서 프론트로 실시간 데이터 전송하는 방법
1. WebSocket(웹소켓)
    - 가장 전형적인 방법
    - 서버와 클라이언트 간의 양방향 통신 지원
    - 한 번 연결을 수립하면 지속적으로 데이터를 주고받을 수 있음
    - 실시간 채팅, 게임, 주식 거래 등 빠른 반응이 필요한 애플리케이션에 적합

2. Server-Sent Events (SSE)
    - 서버에서 클라이언트로 단방향 실시간 전송
    - HTTP 연결을 유지한 채 서버가 계속 푸시
    - 가볍고 심플한 실시간 알림(예: 뉴스 속보, 공지사항)에 적합
    
3. HTTP Long Polling
    - 단방향
    - 오래된 방법이지만 여전히 쓰임
    - 클라이언트가 서버에 요청을 보내고, 서버는 데이터가 생길 때까지 응답을 안 돌려줌.
    - 데이터 생기면 응답 보내고, 클라이언트는 다시 요청
    - WebSocket 지원이 어려운 환경에서 대체용으로 씀

4. 라이브러리 활용
    - Socket.IO (WebSocket 기반 + fallback 제공)
    - 백엔드: socket.emit()
    - 프론트엔드: socket.on()
    - Firebase Realtime Database, Supabase 같은 것도 실시간 데이터 제공 기능 내장.


- SSE 예시:
    - 하나의 클라이언트가 연결되면, 서버는 여러 종류의 데이터를 다른 이벤트로 클라이언트에 전송
    - 예를 들어, 교통 데이터와 주차 데이터를 동시에 해당 클라이언트에 보내는 방식
    - 이를 위해, 각 데이터는 각기 다른 이벤트 이름으로 구분하여 클라이언트로 전달

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/traffic-sse")
public class SseTrafficController {

    private final List<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    @GetMapping("/stream")
    public SseEmitter streamTraffic() {
        SseEmitter emitter = new SseEmitter(0L); // 타임아웃 없음
        emitters.add(emitter);

        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));
        emitter.onError((e) -> emitters.remove(emitter));

        return emitter;
    }

    // 주기적으로 데이터를 전송
    public void sendToClient(Object trafficData, Object parkingData) {
        for (SseEmitter emitter : emitters) {
            try {
                // 'traffic-update' 이벤트로 교통 데이터 전송
                emitter.send(SseEmitter.event().name("traffic-update").data(trafficData));

                // 'parking-update' 이벤트로 주차 데이터 전송
                emitter.send(SseEmitter.event().name("parking-update").data(parkingData));
                
            } catch (IOException e) {
                emitter.completeWithError(e);
                emitters.remove(emitter);
            }
        }
    }
}
```

```java
    @Component
    @RequiredArgsConstructor
    public class PushScheduler {

        private final WeatherEsService weatherEsService;
        private final ESRoadService roadService;
        private final ParkEsService parkEsService;
        private final StreamController streamController;

        // 5분마다 날씨 데이터를 SSE 클라이언트에 push
        @Scheduled(fixedRate = 300_000) // 5분 = 300초
        public void pushWeatherToClients() {
            var weatherList = weatherEsService.getAllWeatherFromES();
            var trafficList = roadService.getTrafficData();
            var parkList = parkEsService.getAllParkFromES();
            streamController.sendToClients(weatherList, trafficList, parkList);
        }

    }
```

- 핵심 포인트:
    - **두 가지 다른 데이터 (교통 데이터와 주차 데이터)**를 동일한 클라이언트에게 다양한 이벤트 이름(traffic-update, parking-update)을 사용하여 전송
    - 클라이언트는 각 이벤트 이름에 따라 서버에서 전송한 데이터를 구분할 수 있음
    - 클라이언트에서 수신하는 방식: 클라이언트에서는 서버로부터 받은 데이터를 이벤트 이름을 기준으로 처리할 수 있음

```javascript
const eventSource = new EventSource("/traffic-sse/stream");

eventSource.addEventListener("traffic-update", function(event) {
    const trafficData = JSON.parse(event.data);
    console.log("교통 데이터:", trafficData);
});

eventSource.addEventListener("parking-update", function(event) {
    const parkingData = JSON.parse(event.data);
    console.log("주차 데이터:", parkingData);
});
```

- 결론
    - 하나의 클라이언트에게 여러 종류의 데이터를 보내기 위해, 서버에서는 각각 다른 이벤트 이름을 사용하여 데이터를 전송
    - 클라이언트에서는 각 이벤트에 맞는 데이터를 수신하고, 처리할 수 있음
    - 이렇게 하면, 하나의 클라이언트에 여러 가지 데이터를 전송할 수 있게 됩니다!



#### 레디스
- 데이터 조회



## 에러 모음
1. SSE → 배포 후 gateway에서 504 timeout 뜨는 에러 
    1.  로드밸런서 설정확인
        - 'Connection idle timeout'은 특정 연결이 어떠한 데이터도 전송하지 않고 유휴 상태(idle)로 있을 수 있는 최대 시간으로 클라이언트와 서버 사이의 연결이 활성 상태에서 아무런 데이터 교환이 없을 때, 이 연결을 얼마나 오래 유지할 것인지 결정하는 시간 설정이다. 설정된 시간이 경과한 후에는 유휴 상태의 연결이 자동으로 종료된다.
        -> 기본 60초 설정
        - 로드밸런서의 속성 -> Connection idle timeout이 60초로 되어 있음 -> SSE 통신 시간에 따라 시간을 늘려준다.

    2. GAteway timeout 늘리기
        `spring.cloud.gateway.httpclient.response-timeout=300000`

2. sse 처음 구독하고 5분동안 데이터가 안오는 문제 ⇒ 초기 데이터 전송 필요


3. SSE 초기 데이터 전송 시 발생하는 에러
    - 에러명
    `EventSource's response has a MIME type ("text/plain") that is not "text/event-stream". Aborting the connection.`

    - 초기 데이터를 보낼 떄
        - 오류 코드
            ```
                @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
                public SseEmitter stream() {
                SseEmitter emitter = new SseEmitter(0L); // 타임아웃 없음
                emitters.add(emitter);

                emitter.onCompletion(() -> emitters.remove(emitter));
                emitter.onTimeout(() -> emitters.remove(emitter));
                emitter.onError((e) -> emitters.remove(emitter));

                try {
                    var weatherList = weatherEsService.getAllWeatherFromES();
                    var trafficList = roadService.getTrafficData();
                    var parkList = parkEsService.getAllParkFromES();
                    var accidentList = accidentEsService.getAllAccidentsFromES();

                    sendToClients(weatherList, trafficList, parkList,accidentList);

                } catch (IOException | IllegalStateException e) {
                    emitter.completeWithError(e);
                    emitters.remove(emitter);
                }

                return emitter;
                }

                    // 주기적으로 클라이언트에게 push
                    public void sendToClients(List<Map<String, Object>> weatherList, JsonNode trafficList, List<Map<String, Object>> parkList, List<Map<String, Object>> accidentList) {
                        for (SseEmitter emitter : emitters) {
                            try {
                                emitter.send(SseEmitter.event()
                                        .name("weather-update")
                                        .data(weatherList));

                                emitter.send(SseEmitter.event()
                                        .name("traffic-update")
                                        .data(trafficList));

                                emitter.send(SseEmitter.event()
                                        .name("park-update")
                                        .data(parkList));

                                emitter.send(SseEmitter.event()
                                        .name("accident-alert")
                                        .data(accidentList));

                            } catch (IOException | IllegalStateException e) {
                                System.out.println("SSE 오류 발생");
                                emitter.completeWithError(e);
                                emitters.remove(emitter);
                            }
                        }
                    }
            ```

        - sendToClients(...) 메서드는 모든 emitters 리스트에 있는 emitter들을 순회하면서 데이터를 보내는 함수입니다.

            ```
                for (SseEmitter emitter : emitters) {
                emitter.send(...); // <- 이 emitter는 지금 streamWeather()에서 만든 emitter가 아님
                }
            ```
            - 이때 streamWeather() 안에서 생성한 emitter는 아직 클라이언트와의 연결이 완전히 수립되지 않은 상태일 수 있습니다. 그래서 sendToClients()로 보내려고 할 때 브라우저가 SSE 연결을 정식으로 맺기 전에 데이터를 받게 되며, 결과적으로 MIME type이 잘못된 상태로 처리되는 것입니다.

    - 🔁 emitter를 sendToClients() 안 말고, 직접 처리하면 해결됨!!
        - sendToClients()는 주기적 업데이트에서만 사용
            ```
                try {
                    var weatherList = weatherEsService.getAllWeatherFromES();
                    var trafficList = roadService.getTrafficData();
                    var parkList = parkEsService.getAllParkFromES();
                    var accidentList = accidentEsService.getAllAccidentsFromES();

                    emitter.send(SseEmitter.event()
                            .name("weather-update")
                            .data(weatherList));
                    emitter.send(SseEmitter.event()
                            .name("traffic-update")
                            .data(trafficList));
                    emitter.send(SseEmitter.event()
                            .name("park-update")
                            .data(parkList));
                    emitter.send(SseEmitter.event()
                            .name("accident-alert")
                            .data(accidentList));

                }
        ```


4. Gateway CORS 에러
    ```java
        @Configuration
        public class Cors {

            @Bean
            public CorsWebFilter corsWebFilter() {
                CorsConfiguration config = new CorsConfiguration();
                config.setAllowCredentials(true); // 인증 정보 포함 X
                config.setAllowedOrigins(List.of(
                        "http://localhost:5173",
                        "http://192.168.0.186:5173",
                        "http://58.127.241.84:5173",
                        "http://map.seoultravel.life",
                        "http://www.seoultravel.life"
                ));
                config.setAllowedHeaders(List.of("Origin", "Content-Type", "Accept", "Authorization"));
                config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));

                // expose Content-Type so it can be sent with the response
                config.setExposedHeaders(List.of("Content-Type", "Cache-Control"));

                UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
                source.registerCorsConfiguration("/**", config); // 모든 경로에 적용

                return new CorsWebFilter(source);
            }

        }
    ```
    - 다른 서비스에서도 CORS 방지 설정을 해두면, 실행은 되는데 코드 중복이라고 에러가 나온다



5. DB connection pool 너무 많이 쓰고 있는 문제 발생 (기존에는 100개정도..) => Connection Pool과 적절한 Size 설정하기 (with HikariCP)
    - https://colour-my-memories-blue.tistory.com/15
    - ⇒ 멘토님 say : DB Connection Pool을 사용하셔야 하고 대표적인게 HikariCP 입니다.

    ```java
        # 각 서비스의 application.properties에 추가한 내용
        spring.datasource.hikari.maximum-pool-size=8
        spring.datasource.hikari.minimum-idle=8

        # 우리 인프라 상황 : 각 개별 인스턴스마다 커넥션 풀 8개씩
        총 64개 (8개 x 서비스 4개 x 인스턴스 2개)
        ** postgresql이 허용하는 최대 커넥션 풀 개수는 200 **
    ```
    * ✅ **일반적인 권장사항**
        - **최대 커넥션 수의 80% 이하로 운영**하는 것이 **권장**됩니다.
        - 나머지 20%는 다음과 같은 이유로 여유로 남겨둡니다:
            - DBA 접속
            - 백업, 모니터링, 메인터넌스 등 내부 관리용 쿼리
            - 예기치 못한 스파이크 대응
        

    * **💙참고 팁**
        - PostgreSQL은 커넥션 하나당 리소스를 꽤 잡아먹기 때문에, 커넥션 수가 많아질수록 **메모리** 사용량이 증가합니다.
        - 고트래픽 서비스에서는 `pgbouncer`, `pgpool` 등의 **커넥션 풀러** 도입도 고려하세요.
        - DB 커넥션 누수, 트랜잭션 미종료 같은 **커넥션 고정** 문제가 있으면 max 커넥션보다 빨리 고갈되므로 주기적인 모니터링도 중요합니다.


    - HikariCP
        - 스프링 부트 2.0부터는 HikariCP를 기본 DataSource로 채택하고 있습니다. HikariCP는 빠르고 간편하며 오버헤드가 zero라고 합니다(HikariCP 피셜이며 자세한 이유는 모르겠네요).
        - HikariCP Property


6. 프론트에서의 api연결은 되는데(cors 허용해둔 url) 내부 msa 백엔드 서비스 서버끼리의 REST 통신이 안되는 경우..
    - 일단 같은 DB를 사용하고 있어서 해당 DB에서 데이터 조회하는 걸로 수정해둠



#### spring boot 데이터 캐싱
컨트롤러에서 데이터를 넣고, 서비스에서 사용하는 경우



#### 정적 팩토리 메소드




### 질문 사항
- 멀티 모듈 프로젝트 구조
    - Gradle이나 Maven에서 하나의 **메인 프로젝트(루트 프로젝트)** 아래에 여러 개의 **서브모듈(모듈)**을 포함시키는 구조
    - 서브모듈은 단순히 "하위 프로젝트"
    | 이유             | 설명                                                                 |
    |------------------|----------------------------------------------------------------------|
    | 코드 재사용성    | 공통 기능(예: 인증, DB 연결, 유틸 등)을 다른 모듈에서도 재사용할 수 있음 |
    | 개발 분리        | 기능 단위로 모듈을 분리해서 각각 독립적으로 개발/테스트 가능            |
    | 빌드 속도 향상   | 변경된 모듈만 다시 빌드하여 전체 빌드 시간을 줄일 수 있음              |
    | 책임 분리        | 도메인 별로 코드 분리하여 유지보수성과 테스트 용이성 향상              |
    | 배포 전략 다양화 | 일부 모듈만 따로 jar로 배포하는 것도 가능                              |



    - 각 모듈에 대해 공통 모듈은 다른 모듈들이 implementation(project(":sumits.jdbc.postgre")) 처럼 가져다 쓸 수 있습니다.
    - 공통 DB 설정도 공유 가능
    - 이 구조는 보통 도메인 구분이 확실하거나, 회사/팀 프로젝트에서 유지보수와 협업이 중요할 때 사용하는 구조입니다. Spring Boot 기반의 기업용 프로젝트에서 자주 사용하는 구조이며, 단일 프로젝트보다 훨씬 확장성과 재사용성이 뛰어납니다.

- 멀티모듈 프로젝트 구조와 **MSA(Microservices Architecture)**는 목적과 구조가 다르지만, 적절히 조합해서 사용할 수 있습니다.
    | 항목        | 멀티모듈 (Multi-Module)              | MSA (Microservices Architecture)         |
    |-------------|---------------------------------------|------------------------------------------|
    | 구조        | 하나의 프로젝트 안에서 모듈 분리     | 완전히 독립된 서비스로 분리              |
    | 배포        | 하나의 어플리케이션으로 빌드/배포     | 각 서비스 독립 빌드 및 배포              |
    | 데이터베이스| DB 공유도 가능                        | 보통 서비스마다 독립된 DB 사용           |
    | 장점        | 코드 재사용, 개발 편의성              | 유연한 확장성, 독립 배포, 장애 격리      |
    | 단점        | 컴파일/빌드 의존성 존재               | 복잡한 인프라 필요 (게이트웨이, 통신 등) |

- 멀티모듈 + MSA 조합 예시 구조도 도식



- ✅ PostgreSQL 서브모듈을 사용하는 목적은?
    - 목적: PostgreSQL 설정을 별도 Git 프로젝트로 분리해서 Git Submodule로 포함함으로써:
    - DB 초기 설정, 테이블 생성 스크립트, 데이터 시드 등을 중앙화하여 재사용할 수 있음
    - 다른 MSA 서비스들이 이 DB 설정을 공유해서 일관된 인프라 환경을 구성할 수 있음
    - 예: Docker로 DB 컨테이너 올릴 때, /databases/postgresql 서브모듈을 사용



- configmap이 우선으로 먹어서 application.properties 냅둬도 된다.


- 집킨(Zipkin)
    - 의존성 주입




