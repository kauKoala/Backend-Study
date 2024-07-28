# Oracle vs MySQL







## Join 지원

### Oracle

Oralce은 **Nested Loop Join** 과 더불어 **Hash Join**, **Sort Merge Join**을 지원한다.



### MySQL

사실 MySQL은 **Nested Loop Join** 만 지원했으나...

Join하는 두 테이블의 크기가 매우 큰 차이를 보일때 혹은 인덱스를 사용할 수 없을때 등등 특정 상황에서 **Nested Loop Join**은 한계가 있어

**Hash Join**은 MySQL 8.0.18버전 부터 지원하게 되었다.

**Sort Merge Join**은 아직 지원하지 않는다.





## Auto Increment

### Oracle

Oracle은 특정 칼럼에 대해 **Auto Increment** 속성이 존재하지 않는다. 

기존 Oracle은 **Oracle**에서 특정 칼럼의 자동 증가 속성을 사용하고 싶다면 **Sequence**를 생성해야 했다.

하지만 **Identity**를 사용하여 칼럼을 생성할때 **Sequence** 없이 칼럼 값 자동 증가가 가능해졌다!



### MySQL

MySQL은 자체적으로 **Auto Increment** 속성을 통해 칼럼 값 자동 증가를 지원한다.



## 잔잔바리 DB 함수들



### Null Check

- Oracle
  - NVL
- MySQL
  - IFNULL

### 현재 날짜 / 시간

- Oracle
  - SYSDATE
- MySQL
  - DATE

### 날짜 포맷 변환

- Oracle
  - TO_CHAR
- MySQL
  - DATE_FORMAT

### 정규표현식

- Oracle
  - REGEXP_LIKE
- MySQL
  - REGEXP
  - RLIKE

### 부분 문자열

- Oracle
  - SUBSTR()
- MySQL
  - SUBSTRING()



### 페이징

- Oracle
  - ROWNUM

- MySQL
  - LIMIT





## 가격

### 온디맨드 구축

- MySQL
  - 버전에 따라 다르다. 하지만 Community Version이라는 무료 버전을 지원한다.
  - Standard 버전이라는 상용 유료 버전은 200 ~ 1000달러 사이다. 
  - ~~MariaDB 쓰자.~~

- Oracle
  - CPU Core에 따라 가격을 매기며 유지보수 가격 또한 포함되어있다.
  - XE라는 무료버전이 존재하나 한계가 많다.
    -  스토리지 제한 : 11GB  메인 메모리 사용 제한 : 1GB ·CPU 사용 제한 : 1개

### AWS RDS for MySQL

![image](https://github.com/kauKoala/Backend-Study/assets/26706925/dd830f9d-9c58-46cf-bd00-ea1e7d503cf4)



### AWS RDS for Oracle

![image](https://github.com/kauKoala/Backend-Study/assets/26706925/b3977837-c462-4bf7-a0d0-5b08e565f279)

![image](https://github.com/kauKoala/Backend-Study/assets/26706925/8ca265c7-c6ee-4e96-b486-5ff42c626f37)

## Reference

https://stackoverflow.com/questions/11296361/how-to-create-id-with-auto-increment-on-oracle

https://aws.amazon.com/ko/rds/oracle/pricing/
