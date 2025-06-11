# 🧠 Elasticsearch란?
- 대용량 데이터를 빠르게 검색하고 분석할 수 있는 실시간 검색 엔진

- 💡 쉽게 말하면?
    - 구글처럼 → "문서, 로그, 데이터"에 대해 검색할 수 있게 해주는 도구
    - DB처럼 → 데이터를 저장하고, 복잡한 조건으로 필터링·집계·분석할 수 있음
    - 실시간 검색 + 분석을 동시에 잘함

- 🔧 기술적으로는?
    - REST API 기반
    - **JSON 형식으로 데이터 저장**
        - 내부 구조는 JSON → 역색인(Vectors, Tokens) 으로 변환
    - 분산 구조 → 데이터 많아도 빠름, 확장성 Good

- 🧩 관련 키워드
    - Index: 데이터 저장 공간 (RDB의 테이블 느낌)
    - Document: 저장되는 JSON 데이터 하나 (RDB의 row 느낌)
    - Field: JSON 안의 하나하나 key (RDB의 column 느낌)
    - Query DSL: JSON으로 쿼리하는 문법


### REST API 기반 Elasticsearch 호출
<details>
<summary>상세 내용</summary>

1. RestTemplate 으로 호출하기
    - 의존성 (Spring Boot 3.x부터는 수동 등록 필요)
        ```java
        @Bean
        public RestTemplate restTemplate() {
            return new RestTemplate();
        }
        ```
    - ex) 검색 API 호출
        ```java
        @Autowired
        private RestTemplate restTemplate;

        public String searchByArea(String area) {
            String esUrl = "http://localhost:9200/seoul_citydata_road/_search";

            String requestJson = """
                {
                "size": 10,
                "query": {
                    "match": {
                    "road_traffic.area_nm": "%s"
                    }
                }
                }
                """.formatted(area);

            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);

            HttpEntity<String> entity = new HttpEntity<>(requestJson, headers);

            ResponseEntity<String> response = restTemplate.exchange(
                    esUrl,
                    HttpMethod.POST,
                    entity,
                    String.class
            );

            return response.getBody(); // JSON String
        }

        ```
2. WebClient 으로 호출하기 (비동기 & 리액티브)
    - 의존성
        ```java
        @Bean
        public WebClient webClient() {
            return WebClient.builder().baseUrl("http://localhost:9200").build();
        }
        ```
    - ex) 검색 API 호출
        ```java
        @Autowired
        private WebClient webClient;

        public Mono<String> searchByAreaReactive(String area) {
            String body = """
                {
                "size": 10,
                "query": {
                    "match": {
                    "road_traffic.area_nm": "%s"
                    }
                }
                }
                """.formatted(area);

            return webClient.post()
                    .uri("/seoul_citydata_road/_search")
                    .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                    .bodyValue(body)
                    .retrieve()
                    .bodyToMono(String.class);
        }
        ```

- ES의 검색 APi 호출이랑 저장 API 호출 차이
    1. 검색 API 호출 (_search) : 이미 저장되어 있는 문서를 찾아오는 것 (조회용)
        - HTTP POST (또는 GET 가능하지만 POST가 일반적)
        - 쿼리 DSL을 JSON으로 전달해서 검색
        - _search endpoint 사용 (/index/_search)
        - 출력 : 검색 결과 (hits 등)
    2. 저장 API 호출 (_doc, 또는 PUT/POST) : 새로운 데이터를 Elasticsearch 인덱스에 저장하거나 갱신하는 것
        - PUT (ID를 지정해서 저장/업데이트) 또는 POST (ID 자동 생성)
        - _doc endpoint 사용 (/index/_doc/{id})
        - 출력 : 저장 성공 여부, ID 등

- 참고
    - RestTemplate은 동기식으로 간단하고 익숙함.
    - WebClient는 비동기 기반으로 리액티브하게 사용 가능 (Spring WebFlux에 적합).
    - index, _doc, _search 등의 endpoint는 Elasticsearch REST API 문서 기반으로 구성됨.

</details>



## ES에서 10,000건(limit) 이상은 기본 설정으로 한 번에 가져올 수 없다. 이걸 "deep pagination 문제"
- es가 만건 넘게 검색되면 데이터를 안줌 → 쿼리에다가 search sort를 하게 해서 다음 페이지로 넘어가서  오도록? 이런 처리들이 필요함
- 기본 문제 상황
    - ES는 from + size 방식으로 pagination을 하는데, from이 커질수록 성능 저하
    - 기본 설정으로는 최대 10,000건까지만 가져올 수 있어 (index.max_result_window = 10000)
- ✅ 해결 방법 – search_after 사용
    - ES에서 정렬을 기준으로 "다음 페이지"를 가져오기 위해 사용하는 게 바로 search_after 쿼리야.
    - search_after는 sort 기준으로 마지막 값을 기억해두고, 그 이후 데이터를 이어서 가져오는 방식이야.

```
    {
    "size": 5,
    "query": {
        "term": {
        "congestion.area_nm.keyword": "교대역"
        }
    },
    "sort": [
        {
        "congestion.ppltn_time": {
            "order": "desc"
        }
        },
        {
        "_id": "desc"
        }
    ],
    "search_after": ["2024-05-16T08:00:00", "document_id_123"]
    }

```
- search_after의 값은 이전 페이지 마지막 문서의 sort 필드 값들이야. 복수 정렬 조건이 있을 경우 모두 넣어야 해.