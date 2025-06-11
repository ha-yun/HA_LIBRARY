#### etc
- data.sql 파일이 있으면, Spring Boot가 내장 DB(H2, HSQL, Derby) 또는 **설정된 RDBMS(MySQL, PostgreSQL 등)**에 대해 애플리케이션 실행 시점에 자동으로 실행.
    - ✅ 자동 실행 조건 정리
        1. src/main/resources/data.sql 경로에 있어야 함
        2. Spring Boot에서 해당 DB 연결이 정상적으로 설정되어 있어야 함
        3. spring.datasource.initialization-mode 값이 기본값(embedded), 또는 always로 설정돼 있어야 실행됨 (버전 2.5까지는)
            - `spring.datasource.initialization-mode: always`
            - Spring Boot 2.5 이상부터는 이 설정은 deprecated 되고, spring.sql.init.mode를 사용
        4. Docker로 실행할 경우에도, 컨테이너 안에서 Spring Boot 애플리케이션이 기동되면서 위 조건을 만족하면 실행됨.
    
    - Docker + Spring Boot + data.sql 시나리오 예시
        ```
        # application.yml 예시
            spring:
            datasource:
                url: jdbc:mysql://db:3306/mydb
                username: root
                password: password
            sql:
                init:
                mode: always

        ```


- `spring.jpa.show-sql=true`
    - 이 설정은 JPA가 실행하는 SQL 쿼리를 콘솔(log)에 출력 (예: select, insert, update, delete 쿼리 등)
