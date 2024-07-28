# Group By(1)

## Group By란?

![img](https://blog.kakaocdn.net/dn/1kPUl/btqTEMascrt/P6HbT25ol4cWR4LfKDRK4k/img.png)

Group By는 테이블에서 집계함수를 사용할때 사용하는 구문으로 일정한 기준으로 데이터들을 그룹하여 보여준다.

MIN, MAX, SUM과 같이 여러 데이터들을 취합하여 하나의 데이터로 보여주는 집계함수에서 필수적인 구문이라고 볼 수 있다.

예를 들어 방문자와 방문날짜가 기록되어 있는 테이블에서 일일 방문자가 궁금하다면

```sql
SELECT
 date,
 COUNT(*)
FROM visit
GROUP BY date;
```

GROUP BY를 사용하여 날짜를 기준으로 그룹화하고 집계함수를 사용하여 총합을 구할 수 있게 된다.

## Having

Group By의 문제는 SQL 실행 순서상 데이터들을 그룹화한 후 (Group By 실행 후) 나온 데이터들에 대해서 조건을 걸 수 없다는 점에 있다.

![img.png](https://blog.kakaocdn.net/dn/dOq9i4/btsDQ0mOa2U/Ok1Xmn3BXiQ6ELcFzr7HLk/img.png)

위 이미지에서 알 수 있듯이 GROUP BY

따라서 그룹화한후 데이터들에 대해서 조건을 통해 필터링할 수 있는 HAVING이 생기게 되었다.

위에서 들었던 일일방문자 예시에서 일일방문자가 3명이상인 날에 대한 데이터를 얻고 싶다면

```sql
SELECT
 date
FROM visit
GROUP BY date
HAVING COUNT(*) > 3
ORDER BY date;
```

이런식으로 쿼리를 만들 수 있다.

## MySQL의 Group By



### MySQL의 Group By 성능 최적화

- Full Table Scan

  - 기본적으로 Full Table Scan을 통해 테이블의 모든 행을 가져온 후 Group By의 조건에 따라 그룹화된 행끼리 연속된 임시 테이블을 만든다. 그 후 집계함수가 있는 경우 임시테이블에 대해 집계작업이 이루어지게되는데..
    - Full Table Scan이 일어나므로 최적화여지가 있어보인다!

- 인덱스를 사용하는 Group By

  - 임시테이블을 만드는 목적을 주목해보면 위에서 언급했듯이 **Group By의 조건(선택된 칼럼들)에 따라 그룹화된 행끼리 연속**된 테이블을 만드는 것이 목표이다.
  - 인덱스는 특정 칼럼들에 대해 **동일한 키 값을 가지는 데이터가 연속된 위치에 저장되도록 보장**한다. 또한 **전체 테이블에 대해서도 이미 특정 칼럼들에 대해 정렬되어 있는 상태로 저장**된다.
    - 이미 Group By에 사용되는 임시테이블의 역할을 하고 있다고 볼 수 있다!
    - 단 Group By에서 사용되는 모든 칼럼들에 대해 단일 인덱스가 지정되어 있어야한다. 또한 해시 인덱스 같은 경우에는 키를 순서대로 저장하지 않아 사용될 수 없다.
  - 인덱스를 사용하여 Group By를 하는 방법은 크게 두가지로 나뉜다.

- Loose Index Scan

  - 인덱스는 키가 정렬되어 있다는 특성을 이용해 Full Table Scan을 하지 않고 그룹을 찾는 방식이다.

    - 따라서 B-Tree 인덱스 같이 정렬된 인덱스에서만 가능하다.

  - 하지만 Loose Index Scan을 Group By에 적용하기 위해서는 여러 조건들이 필요하다.

    - 단일 테이블 쿼리

      - Group By를 포함하는 쿼리가 단일 테이블에 대해서만 이루어져야 한다.

    - Group By에 사용되는 칼럼들의 순서는 인덱스에 사용되는 칼럼들의 순서와 같아야 하며 첫 칼럼을 포함해야한다. (왼쪽 접두사여야 한다.)

      - 다중 칼럼 인덱스는 정렬 기준을 잡는 순서때문에 생성시 칼럼들의 순서를 가진다.

        ```sql
        CREATE INDEX idx_example ON table_name (column1, column2, column3);
        ```

      - ex)  테이블에 (c1 , c2, c3) 인덱스가 있는 경우, GROUP BY c1, c2는 가능하지만 GROUP BY c2,c3는 불가능하다.

    - 인덱스에 없는 칼럼들은 상수 조건에 사용되어야한다.

      - ```sql
        SELECT c2 FROM t1 WHERE c1 < const GROUP BY c1, c2;
        ```

        ```sql
        SELECT c1, c2 FROM t1 WHERE c3 = const GROUP BY c1, c2;
        ```

    - 집계함수는 MIN, MAX만 허용되고 해당 함수들이 적용하는 칼럼은 인덱스와 Group By에 있어야 한다.

    - 부분 인덱싱은 허용되지 않는다. 

  - 만약 Loose Index Scan이 적용되었다면 Explain에서 Using Index for Group-By라는 문구를 볼 수 있다.

    ![MySQL的loose index scan - wingsless - 博客园](https://images2015.cnblogs.com/blog/352291/201512/352291-20151210223757558-1265654313.png)

- Tight Index Scan

  - 위에서 봤듯이 Loose Index Scan은 조건이 상당히 까다로운 것을 알 수 있다.

    - 이를 보완하기 위해 Loose Index Scan과 마찬가지로 임시테이블을 만들지 않는 Tight Index Scan의 조건은 좀 더 쉽다!

  - Tight Index Scan의 조건은 다음과 같다.

    - 단일 테이블 쿼리, 부분 인덱스 조건은 Loose Index Scan과 같다.

    - 칼럼들의 왼쪽 접두사를 상수 조건으로 완성해도 된다.

      ```sql
      SELECT c1, c2, c3 FROM t1 WHERE c2 = 'a' GROUP BY c1, c3;
      ```

      ```sql
      SELECT c1, c2, c3 FROM t1 WHERE c1 = 'a' GROUP BY c2, c3;
      ```

### MySQL의 ONLY_FULL_GROUP_BY

ONLY_FULL_GROUP_BY 옵션은 MySQL에 존재하는 옵션으로 **GROUP BY**절에 포함되지 않는 열은 모두 집계함수를 사용해야만 SQL이 실행되는 옵션이다. 정확히는 **SELECT**, **HAVING** , **ORDER BY**에 사용되면 안된다.

예를 들어 아래와 같이 SELECT에 사용된 'id'열이 GROUP BY에 포함되지 않았다면 집계함수가 사용되어야하므로

해당  SQL이 실행되지 않는다.

```sql
SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));

-- 예제 테이블
CREATE TABLE sales (
    id INT,
    product VARCHAR(50),
    amount DECIMAL(10, 2)
);

INSERT INTO sales (id, product, amount) VALUES
(1, 'Product A', 10.00),
(2, 'Product B', 20.00),
(3, 'Product A', 30.00);

-- GROUP BY 절에 포함되지 않은 열 (id)
SELECT id, product, SUM(amount)
FROM sales
GROUP BY product;

```



만약 ONLY_FULL_GROUP_BY 모드에서 해당 SQL을 실행시킬라면 집계함수로 감싸야하는데 해당 열의 아무 데이터나 뽑아오는 ANY_VALUE()를 사용할수도 있다.

```sql
SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));

-- 예제 테이블
CREATE TABLE sales (
    id INT,
    product VARCHAR(50),
    amount DECIMAL(10, 2)
);

INSERT INTO sales (id, product, amount) VALUES
(1, 'Product A', 10.00),
(2, 'Product B', 20.00),
(3, 'Product A', 30.00);

-- GROUP BY 절에 포함되지 않은 열 (id)
SELECT ANY_VALUE(id), product, SUM(amount)
FROM sales
GROUP BY product;
```



언뜻 보면 왜 사용되는지 이해가 가지 않는 모드지만 GROUP BY를 사용하여 그룹화한 결과에 집계함수가 적용되지 않는 열이 존재할 경우

해당 열은 의미 없는 값이 들어가기 때문에 이를 방지하기 위해 만들어진 모드이다.



### GROUP BY(2)의 내용

- Oracle GROUP BY 방식
  - Sort
  - Hash
  - 임시테이블
- Oracle의 ROLL UP (GROUPING)
- DISTINCT 와 GROUP BY



## Reference

- https://dev.mysql.com/doc/refman/8.4/en/group-by-modifiers.html
- https://dev.mysql.com/doc/refman/8.4/en/group-by-handling.html
- https://dev.mysql.com/doc/refman/8.4/en/group-by-optimization.html



 
