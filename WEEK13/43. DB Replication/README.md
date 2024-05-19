# 43. DB Replication



## DB Replication이란?

**동일한 데이터**를 한 데이터베이스 서버에서 **다른 서버들로 복사하거나 배포**하여 DB의 부하를 분산하는 기술



## DB Replication의 기본원리



Master DB에는 쓰기 관련 작업만을 수행하고 이에 대한 수정을 Slave DB에 반영한다.

Query의 대부분이 Select인점(대략 DB 요청의 60~80%)을  감안하여 Slave DB는 읽기 작업만을 수행하며 부하를 분산시킨다.

![image](https://github.com/kauKoala/Backend-Study/assets/26706925/5e46ce79-4334-4524-9435-e85378d6024f)


## Replication 방법

로그기반 Replication은 MySQL, PostgreSQL, Oracle 등등 여러 DB에 걸쳐 사용되고 있음

- MySQL
  - Statement Based
    - 로그에 저장할때 SQL문 자체를 저장함. Slave에서는 로그에 저장된 SQL문을 실행함
    - SQL문을 저장하기에 로그가 비교적 사이즈가 작아짐
    - 실행할때마다 결과가 바뀌는 쿼리인경우 일관성이 깨질 수 있음(현재시간, UUID)
  - Row based
    - Row가 변경될것을 직접 로그에 저장함
    - 결과를 저장하고 Slave에 반영하기에 일관성이 지켜짐
    - 변경되는 데이터가 많아질수록 로그 사이즈가 커짐
  - Mixed 
    - 변경되는 데이터가 많을 경우 Statement Based, 일관성이 깨질경우 Row based와 같이 상황에 맞게 혼용함
- PostgreSQL
  - WAL(Write-Ahead Logging)
    - 데이터베이스에서 발생하는 모든 변경 사항을 로그로 남기는 방법으로,
      데이터 변경을 **디스크에 기록하기 전에 로그에 먼저 기록**
    - WAL Record - DB의 변경사항 하나
    - WAL File - 여러 개의 WAL Record를 하나로 묶은 것
  - File-based Log Shipping
    - WAL File을 주기적으로 Slave에 전송함 (정해진 파일크기, 시간)
    - 구성이 간단함
    - 실시간으로 Slave에 동기화되지 않기에 데이터가 일관적이지 않을 수 있음
    - File이 차기전에 Master에 장애 발생시 로그 손실
  - Streaming Replication
    - WAL Record를 거의 실시간으로 Slave에 전송함
    - Clustering과 거의 비슷한 가용성 효과를 얻을 수 있음
    - 하지만 높은 동기화 비용 (모든 변경사항 복사)
  - Logical Replication
    - Streaming Replication과 거의 비슷하지만 DML 발생시 WAL Record를 **Logical Decoding** 처리 후 전송
    - PostgreSQL 10버전부터 도입 
    - 다른 버전, 다른 DB간 사용 가능 논리적으로 해석작업이 이루어지므로 특정 테이블, DB 범위 제한 가능
    - 변환 오버헤드가 발생할 수 있음



## DB Replication Vs DB Clustering

- 서버의 역할

  - DB Replication은 서버들이 Master, Slave 역할로 구분되어있다.
    - Master 역할을 하는 서버가 다운될 경우 Failover가 매우 어렵다.

  - DB Clustering은 서버들이 동일한 역할을 수행한다.
    - 한 서버가 다운될경우 Fail Over가 가능하다.

- 데이터 일관성

  - DB Replication은 데이터 복제가 실시간으로 이루어지지 않는다. 따라서 읽기에 대한 데이터가 일관성을 가지지 않을 수 있다.
    - Master 서버에서 Slave 서버로의 Replication은 실시간 동기가 아닌 비동기적인 작업이기때문
  - DB Clustering은 데이터 동기화가 여러 서버에 걸쳐 동기적으로 이루어지기때문에 데이터가 일관성을 가진다.
    - 하지만 이로인해 쓰기작업에 대해 지연이 발생한다.



## DB Replication의 이점

- 별도의 쓰기 지연없이 Read 트래픽에 대한 부하 분산이 가능하다.
- 대부분의 DB 요청이 읽기 작업이므로 Replication 만으로 성능 개선이 충분히 이루어진다.

## Reference

https://nesoy.github.io/articles/2018-02/Database-Replication

https://silight.tistory.com/21

https://mangkyu.tistory.com/97
