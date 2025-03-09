# BigDecimal

cf. Decimal; 소수

## 부정확한 실수 연산

```java
double a = 0.1;
double b = 0.2;

System.out.println(a + b); // 0.30000000000000004
```

Java의 기본 실수형 (`float`, `double`)은 **부동소수점** 방식 채택

0.1을 이진수로 변환하면

0.0001100110011001100110011001100110011… 무한한 값이 나옴

즉, 십진수를 이진수로 변환할 때 **근사 오차**가 발생함

```java
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
BigDecimal sum = a.add(b);

System.out.println(sum); // 0.3
```

⇒ `BigDecimal`을 사용한 정확한 연산

## 시스템 별 BigDecimal의 필요성

```java
/* 오차 발생 case */
public class BankAccountError {
    public static void main(String[] args) {
        double balance = 100.0;

        for (int i = 0; i < 1000; i++) {
            balance -= 0.1;
        }

        System.out.println("잔액: " + balance); // 예상: 0, 실제: 1.4043488594239761E-12
    }
}
```

그래픽, 센서, 음향 등의 처리 등의 오차는 인간이 감지할 수 없는 수준이기 때문에 BigDecimal을 사용하지 않아도 됨

but, 금융/회계/은행 등의 시스템에서는 오차 발생이 치명적이므로, **반드시** `BigDecimal`을 사용해야 함

## API

### 1. 생성

```java
// 생성
BigDecimal num = new BigDecimal("10.23"); // String을 인자로 생성
```

### 2. 기본연산

```java
// 기본 연산
BigDecimal result = a.add(b);
result = a.subtract(b);
result = a.multiply(b);
result = a.divide(b, 2, RoundingMode.HALF_UP); // setScale() 반올림 처리
```

### 3. compareTo()

```java
/* equals() */
BigDecimal a = new BigDecimal("10.0");
BigDecimal b = new BigDecimal("10.00");

System.out.println(a.equals(b)); // false (값이 같아도 다름으로 판별됨)

/* compareTo() */
BigDecimal a = new BigDecimal("10.0");
BigDecimal b = new BigDecimal("10.00");

System.out.println(a.compareTo(b)); // 0 (같은 값으로 인식됨)
```

`equals()`는 소수점 자리까지 비교하는 반면에,

`compareTo()`는 수학적으로 같은 값이면 동일하다고 판단함

(테스트 코드에서 많이 사용할 듯)

## BigDecimal의 단점

### 1. 성능

- double, int 등과 달리 **객체**로 동작하여 연산 속도가 느림
- 숫자 연산 시 **객체 생성과 메모리 할당**이 발생 → 속도 저하
- double에 비해 10배 이상 느릴 수 있음
    - 참고

        ```java
        import java.math.BigDecimal;
        
        public class BigDecimalPerformance {
            public static void main(String[] args) {
                long start, end;
        
                // double 연산 속도 테스트
                double dResult = 0.0;
                start = System.nanoTime();
                for (int i = 0; i < 10_000_000; i++) {
                    dResult += 0.1;
                }
                end = System.nanoTime();
                System.out.println("double 연산 시간: " + (end - start) + " ns");
        
                // BigDecimal 연산 속도 테스트
                BigDecimal bdResult = BigDecimal.ZERO;
                BigDecimal bdValue = new BigDecimal("0.1");
                start = System.nanoTime();
                for (int i = 0; i < 10_000_000; i++) {
                    bdResult = bdResult.add(bdValue);
                }
                end = System.nanoTime();
                System.out.println("BigDecimal 연산 시간: " + (end - start) + " ns");
            }
        }
        
        // double 연산 시간: 20~50ms
        // BigDecimal 연산 시간: 500~1000ms (10배 이상 느림)
        
        ```

- 실시간 연산이 필요한 경우(e.g. 게임, 센서 데이터 처리)에서는 double을 쓰는 것이 유리

### 2. 메모리 사용량

- BigDecimal은 내부적으로 **문자열(String)** 기반으로 숫자를 저장함 → 메모리 사용량이 double보다 큼
    - 참고

        ```java
        import java.math.BigDecimal;
        import java.util.ArrayList;
        import java.util.List;
        
        public class MemoryUsageTest {
            public static void main(String[] args) {
                int size = 1_000_000;
        
                Runtime runtime = Runtime.getRuntime();
        
                // double 리스트 메모리 사용량 측정
                List<Double> doubleList = new ArrayList<>();
                long beforeMemory = runtime.totalMemory() - runtime.freeMemory();
                for (int i = 0; i < size; i++) {
                    doubleList.add(123456.789);
                }
                long afterMemory = runtime.totalMemory() - runtime.freeMemory();
                System.out.println("Double 리스트 메모리 사용량: " + (afterMemory - beforeMemory) / (1024 * 1024) + "MB");
        
                // BigDecimal 리스트 메모리 사용량 측정
                List<BigDecimal> bigDecimalList = new ArrayList<>();
                beforeMemory = runtime.totalMemory() - runtime.freeMemory();
                for (int i = 0; i < size; i++) {
                    bigDecimalList.add(new BigDecimal("123456.789"));
                }
                afterMemory = runtime.totalMemory() - runtime.freeMemory();
                System.out.println("BigDecimal 리스트 메모리 사용량: " + (afterMemory - beforeMemory) / (1024 * 1024) + "MB");
            }
        }
        
        // Double 리스트 메모리 사용량: 24MB
        // BigDecimal 리스트 메모리 사용량: 50MB
        
        ```

- 대량의 숫자를 처리할 때 메모리 부담이 커질 수 있음

### 3. 주의할 점

예외 처리가 필요함

```java
/* 예외 발생 case */

BigDecimal a = new BigDecimal("10");
BigDecimal b = new BigDecimal("3");

// setScale() 없이 나누기 → 예외 발생
BigDecimal result = a.divide(b); // ArithmeticException: Non-terminating decimal expansion

/* 해결책 */
BigDecimal a = new BigDecimal("10");
BigDecimal b = new BigDecimal("3");

// 소수점 2자리까지 반올림하여 나누기
BigDecimal result = a.divide(b, 2, RoundingMode.HALF_UP);

System.out.println(result); // 3.33

```

`BigDecimal`은 정확한 연산을 원칙으로 하기 때문에 무한 소수는 표현할 수 없음

→ `ArithmeticException` 발생

⇒ `setScale()`을 사용하여 소수점 반올림하여 해당 exception을 처리함

### 참고자료

https://dev.gmarket.com/75

https://jsonobject.tistory.com/466

chat gpt 4o