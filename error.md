### API 에러 코드 목록

| Custom Code | HTTP Status | Message                     | 설명 |
|-------------|-------------|-----------------------------|------|
| 1000        | 400         | Invalid request             | 잘못된 요청, 필드 누락 등 |
| 1001        | 400         | Weather data not found      | 날씨 데이터 없음 |
| 2000        | 401         | Unauthorized                | 인증 실패 |
| 2001        | 403         | Forbidden                   | 접근 권한 없음 |
| 3000        | 404         | Resource not found          | 리소스 없음 |
| 3001        | 404         | User not found              | 사용자 없음 |
| 3002        | 404         | Weather data not found      | 날씨 정보 없음 |
| 3003        | 404         | Location not found          | 위치 정보 없음 |
| 9000        | 500         | Internal server error       | 서버 내부 오류 |
| 9001        | 503         | Service unavailable         | 서버 점검 중 |
