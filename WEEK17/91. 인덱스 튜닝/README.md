# 1. 테이블 액세스 최소화

cf. 테이블 랜덤 액세스 절차

1. 인덱스 스캔 후 ROWID 획득
2. 버퍼 캐시에서 해당 ROWID가 가리키는 테이블 블록 탐색
3. 2에서 찾지 못하면 ROWID를 통해 디스크에서 블록 탐색

### 클러스터링 팩터 (CF)

: 같은 값을 가진 데이터가 물리적으로 모여있는 정도 (계수)

테이블 블록에 대한 포인터를 바로 해제하지 않음. (Buffer Pinning)

그렇기 때문에 이전에 스캔한 ROWID와 같은 테이블 블록을 가리키는 ROWID라면 포인터를 재활용함

그러므로, 1) CF가 좋으면 2) 블록 I/O가 적게 발생하고 3) 탐색 속도가 빨라짐

### 인덱스 컬럼 추가

```sql
-- deptno + job로 구성한 emp_x01 인덱스

select /*+ index(emp emp_x01) */ *
from emp
where deptno = 30
		and sal >= 2000
```

기존 인덱스에 sal을 추가하는 것 만으로도 액세스를 줄일 수 있음

### 인덱스 구조 테이블 (IOT, Index-Organized Table)

(↔ 힙 테이블)

: 데이터가 인덱스 키 순서대로 정렬되어 저장됨

- 정렬되어 저장되기 때문에 BETWEEN, 부등호 등을 사용할 때 효율적인 **범위 검색**이 가능함
- 읽기 작업과 범위 검색에 적합함
- 변경 작업에 비효율적일 수 있음

```sql
-- ORGANIZATION INDEX 키워드로 IOT 생성

CREATE TABLE employees (
    employee_id NUMBER PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    hire_date DATE,
    salary NUMBER
) ORGANIZATION INDEX;
```

# 2. 부분 범위 처리 활용

: 클라이언트가 데이터 요청을 할 때마다 일정량의 데이터만을 전송하는 방식

### Fetch Size 조절

```sql
-- Paging 예시

SELECT employee_id, first_name, last_name
FROM employees
ORDER BY employee_id
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```

### ORDER BY와 인덱스 활용

: 인덱스 컬럼과 동일한 컬럼을 ORDER BY에 사용함으로써, SORT 연산을 생략하고 일부 데이터만 Fetch할 수 있음

# 3. 인덱스 스캔 효율화

### BETWEEN을 IN-List로 전환

선행 컬럼을 BETWEEN 해야 한다면, IN으로 변경하여 옵티마이저가 INLIST ITERATOR (UNION ALL)을 사용하게 하여 성능을 높일 수 있음

```sql
SELECT 부서번호, 연봉
FROM 사원
WHERE 부서번호 IN('1', '2')
AND 연봉 = '4000'

-- 옵티마이저가 최적화한 쿼리 (UNION ALL)
SELECT 부서번호, 연봉
FROM 사원
WHERE 부서번호 = '1'
AND 연봉 = '4000'
UNION ALL
SELECT 부서번호, 연봉
FROM 사원
WHERE 부서번호 = '2'
AND 연봉 = '4000'
```

- 선택되는 레코드끼리 멀리 떨어져 있을 때 효율이 좋음

### LIKE 대신 BETWEEN

크게 차이나지 않지만, BETWEEN보다 LIKE가 스캔량이 더 많음

### 선행 컬럼:: 등치 O, 범위 X

효율적인 인덱스 스캔을 위해 선행 컬럼이 **등치(=) 조건**이어야 함

범위 조건이 선행 컬럼에 적용되면 인덱스 스캔의 효율성이 저하됨

# 4. 인덱스 설계

### 인덱스가 많을 때의 단점

- DML 성능 저하
- 데이터베이스 사이즈 증가
- 데이터베이스 관리 및 운영 비용 상승

### 가장 중요한 두 가지 선택 기준

- 조건절에 항상 혹은 자주 사용하는 컬럼을 선정할 것
- ‘=’ 조건으로 자주 조회하는 컬럼을 앞쪽에 둘 것

### 소트 연산 생략을 위한 인덱스 설계 공식

1. ‘=’ 연산자로 사용한 조건절 컬럼을 가장 앞에 선정
2. ORDER BY 절에 기술한 컬럼 추가
3. ‘=’ 연산자가 아닌 조건절 컬럼은 데이터 분포를 고려하여 추가 여부 결정