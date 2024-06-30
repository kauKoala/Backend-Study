# Sort Area

MAX, MIN

# 소트 튜닝의 문제점

디스크 소트가 발생하는 순간 SQL 수행 성능은 나빠질 수밖에 없음.

많은 서버 리소스를 사용하고 디스크 I/O가 발생하는 것도 문제지만, **부분범위** 처리를 불가능하게 함으로써 OLTP 환경에서 애플리케이션 성능을 저하시키는 주요인이 됨

소트 연산은 메모리 집약적이고 CPU 집약적임. 처리할 데이터량이 많아 디스크 I/O까지 발생하면 성능이 매우 느려짐

대표적인 order by, group by 뿐만 아니라 Union, Minus, Distinct 연산자도 중복 레코드를 제거하기 위한 소트 연산을 발생시키므로 꼭 필요한 경우에만 사용하고, 성능이 느리다면 소트 연산을 피할 방법이 있는지 찾아봐야 함!

# 소트가 발생하지 않도록 SQL 작성하기

### Union vs. Union All

인스턴스 중복 가능성이 없는 쿼리에는 union이 아닌 union all을 사용해야함.

```sql
SELECT
	..
WHERE 결제일자 = '20180316'
UNION ALL
SELECT
	..
WHERE 주문일자 = '20180316'
AND 결제일자 <> '20180316' 
```

→ 중복을 피하는 예시

### Exists

Distinct를 사용하면 조건에 해당하는 데이터를 모두 읽어서 중복을 제거해야 함. 부분범위 처리는 당연히 불가능하고, 모든 데이터를 읽는 과정에 **많은 I/O**가 발생함.

**EXISTS 서브쿼리**는 데이터 존재 여부만 확인하면 되기 때문에 조건절을 만족하는 데이터를 모두 읽지 않음!

```sql
-- 이 경우 B 데이터를 모두 읽고, 중복을 제거하는 작업이 필요
SELECT DISTINCT A.*
FROM A, B
WHERE A.x = :x
AND B.y = A.y
AND B.z between :dt1 and :dt2

-- 이 경우, B 데이터가 한건이라도 존재하는지만 확인하므로 효율적임
SELECT A.*
FROM A
WHERE A.x = :x
AND EXISTS(
	SELECT 'x' FROM B
	WHERE B.y = A.y
	AND B.z between :dt1 and :dt2
)
```

cf. MINUS도 NOT EXISTS로 튜닝할 수 있음.

# 인덱스를 이용한 소트 연산 생략

: 키 컬럼 순으로 정렬된 상태를 유지하는 인덱스의 특성을 이용하여 소트 연산 생략하기

### Sort Order By 생략

```sql
CREATE INDEX idx_employees_last_name ON employees (last_name);
```

```sql
SELECT * FROM employees ORDER BY last_name;
```

인덱스를 order by로 사용하면 데이터를 정렬 없이 반환할 수 있음

(인덱스가 이미 정렬된 데이터를 가지고 있기 때문)

### Top N 쿼리

: 특정 조건에 맞는 상위 N개의 행을 가져오는 쿼리

```sql
SELECT *
FROM (
    SELECT *
    FROM employees
    ORDER BY salary DESC
)
WHERE ROWNUM <= 10;
```

### 최대/최소값 구하기

```sql
SELECT MIN(salary) FROM employees;
SELECT MAX(salary) FROM employees;
```

```sql
CREATE INDEX idx_employees_salary ON employees (salary);
```

### 이력 조회

: 특정 시점 또는 조건에 맞는 최신 레코드를 조회할 때 인덱스 활용하기

```sql
SELECT * 
FROM employee_history 
WHERE employee_id = 123 
ORDER BY change_date DESC FETCH FIRST 1 ROW ONLY;
```

```sql
CREATE INDEX idx_employee_history ON employee_history (employee_id, change_date DESC);
```