# 0. JOIN이란?

**데이터베이스에서 두 개 이상의 테이블을 연결하여 하나의 결과 테이블로 만드는 방법**

## 특징

- 연결하려는 테이블들이 적어도 하나의 컬럼을 공유하고 있어야 함
- INNER JOIN, OUTER JOIN (LEFT, RIGHT, FULL) 등이 있음

**ON과 WHERE의 차이점**

- ON → JOIN → WHERE 순서로 실행
    - ON 조건으로 필터링을 거친 후, 결과 레코들간의 JOIN 실행
    - JOIN한 결과에서 WHERE 조건절로 필터링

# 1. INNER JOIN
<img width="445" alt="스크린샷 2024-01-13 오후 12 28 42" src="https://github.com/kauKoala/Backend-Study/assets/84485400/2b5a1398-ef98-4b8f-b6b4-09f5a172a40f">

```sql
SELECT *
FROM A a 
	INNER JOIN B b
	ON a.KEY = b.KEY
```

# 2. OUTER JOIN
<img width="635" alt="스크린샷 2024-01-13 오후 12 24 15" src="https://github.com/kauKoala/Backend-Study/assets/84485400/694a5a4c-f56d-43eb-a85e-ac2eefd9e2db">

- OUTER 생략 가능
- `MYSQL`에서는 FULL OUTER JOIN을 지원하지 않음
    - LEFT JOIN과 RIGHT JOIN을 합쳐서 쿼리를 짜야함

        ```sql
        # 공통 부분 포함
        
        SELECT *
        FROM A a
        	LEFT JOIN B b 
        	ON a.KEY = b.KEY
        
        UNION
        
        SELECT *
        FROM A a
        	RIGHT JOIN B b 
        	ON a.KEY = b.KEY;
        ```

        ```sql
        # 공통 부분 미포함
        
        SELECT *
        FROM A a
        	LEFT JOIN B b 
        	ON a.KEY = b.KEY
        WHERE b.KEY IS NULL
        
        UNION
        
        SELECT *
        FROM A a
        	RIGHT JOIN B b 
        	ON a.KEY = b.KEY
        WHERE a.KEY IS NULL;
        ```

- FULL OUTER JOIN은 많은 양의 NULL 값을 생성할 수 있으며, 큰 데이터 세트에서 성능 저하를 일으킬 수 있음

**INNER JOIN과 OUTER JOIN의 차이점**

- `INNER JOIN`은 두 테이블 간에 **일치하는 행**만 반환
- `OUTER JOIN`은 일치하는 행 뿐만 아니라, 한 쪽 또는 양 쪽 테이블에만 있는 행도 포함하여 반환

# 3. 그 외의 JOIN

## CROSS JOIN
<img width="505" alt="스크린샷 2024-01-13 오후 12 57 52" src="https://github.com/kauKoala/Backend-Study/assets/84485400/0aac7878-107b-4085-80ff-78aaa7a3ba5e">

- 한 쪽 테이블의 모든 행과 다른 쪽 테이블의 모든 행을 조인시키는 기능
- CROSS JOIN의 전체 행 개수는 두 테이블의 각 행의 개수를 곱한 수

```sql
SELECT *
FROM A a
    CROSS JOIN B b;
```

## SELF JOIN
<img width="501" alt="스크린샷 2024-01-13 오후 1 00 08" src="https://github.com/kauKoala/Backend-Study/assets/84485400/90d8c68e-cb11-48b6-875f-6d37790c70d0">

- 자기 자신과의 조인 = 1개의 테이블만 사용

```sql
SELECT *
FROM A a1
    INNER JOIN A a2
    ON a1.KEY = a2.KEY;
```

**참고자료**

- https://pearlluck.tistory.com/46
- https://dev-coco.tistory.com/158
- [https://hongong.hanbit.co.kr/sql-기본-문법-joininner-outer-cross-self-join/](https://hongong.hanbit.co.kr/sql-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95-joininner-outer-cross-self-join/)
