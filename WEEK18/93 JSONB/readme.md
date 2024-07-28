JSON을 바이너리로 표현한 데이터 바이너리로 변환하는 과정에 약간의 오버헤드는 있지만 데이터 처리가 쉬워진다.

![1](https://github.com/kauKoala/Backend-Study/assets/7845568/da036925-9c80-4a9a-8bc4-c5a3a5230297)

GIN 인덱스를 통해 데이터를 탐색한다. 인덱스 구조에 대해선 다음 주에 설명할 예정

PostgreSQL에선 json, jsonb 관련 명령어를 제공해 용도에 맞춰 데이터를 처리할 수 있다.

### JSON만을 써야 할 때

- 데이터를 원문 그대로 저장할 때
- 데이터에 형식이 있을 때
- JSON 표준 인코딩이 아닌 문자를 쓰는 경우
  - postgreSQL 경우, UTF가 아닌 유니코드(\uxxxx) 식이라면 오류를 반환한다.

### 활용 사례

**Hash Join 테이블 스캔 성능 개선**

![2](https://github.com/kauKoala/Backend-Study/assets/7845568/b0d1df70-7bbb-4ab0-9b38-b8733d3f2ca3)

두 테이블이 Hash Join 하는 과정에서 테이블 스캔에 병목 발생

문자열 형식의 JSON을 읽는 과정에서 수행 시간이 10초나 걸리는 문제 발생 JSONB를 사용해 문자열을 바이너리로 바꾼 결과 1초 이내로 해결

특수한 상황이라 DB에서 JSON 데이터를 처리한 거였지 대부분 DB 단에서 JSON 같은 데이터를 직접 처리할 상황은 없지 않을까라고 생각한다.