## 1. `e.printStackTrace();`란?

Java에서 예외가 발생했을 때, **스택 트레이스(Stack Trace)**를 **표준 오류 출력**(`System.err`)에 출력하는 메서드. 주로 디버깅용에 사용됨.

cf. **스택 트레이스(Stack Trace)**란?

프로그램 실행 중 예외(Exception)가 발생했을 때,

예외가 어디서 발생했고, 그 이전에 어떤 메서드들이 호출되었는지 **역순**으로 보여주는 정보 목록

```java
public class StackTraceExample {
    public static void main(String[] args) {
        a();
    }

    public static void a() {
        b();
    }

    public static void b() {
        int result = 10 / 0; // 예외 발생
    }
}
```

```java
java.lang.ArithmeticException: / by zero
	at StackTraceExample.b(StackTraceExample.java:12)
	at StackTraceExample.a(StackTraceExample.java:8)
	at StackTraceExample.main(StackTraceExample.java:4)
```

cf. **표준 오류 출력**(Standard Error Output, `System.err`)

Java에는 일반 출력을 담당하는 `System.out`과 에러 메시지 출력을 담당하는 `System.err`라는 두 가지 출력 채널이 있음

`System.err` : 에러 메시지를 출력하는 전용 **콘솔 스트림**

## 2. `e.printStackTrace();` 쓰지마?

`e.printStackTrace();`가 운영 환경에 부적절한 이유는?

1. 로그 파일로 남지 않고 콘솔에만 출력되어 추적이 어려움

   (기본적으로 `System.err`에만 출력되며, 로그 파일로 남기려면 별도 설정 필요)

2. 로그 레벨(`INFO`, `ERROR`, `DEBUG`) 구분 없이 출력됨
3. 메시지에 민감 정보가 포함되면 콘솔 노출 위험 있음

## 3. 로깅 프레임워크와의 비교

```java
public class Example {
    public static void main(String[] args) {
        try {
            int result = 10 / 0;
        } catch (Exception e) {
            // A. printStackTrace()
            e.printStackTrace();

            // B. 로깅 프레임워크
            Logger logger = LoggerFactory.getLogger(Example.class);
            logger.error("예외 발생!", e);
        }
    }
}

```

```java
// A. printStackTrace()
java.lang.ArithmeticException: / by zero
	at Example.main(Example.java:5)
```

→ 단순 예외 메시지와 스택 경로만 출력

```java
// B. 로깅 프레임워크
[2025-05-18 20:30:12] ERROR com.example.Example - 예외 발생!
java.lang.ArithmeticException: / by zero
	at Example.main(Example.java:5)
```

→ 시간, 로그 레벨, 클래스명 포함

→ 로깅 정책(Logback 등)에 따라 파일 분리, 색상, JSON 형식 등도 가능

| 항목 | `e.printStackTrace()` | `logger.error("msg", e)` |
| --- | --- | --- |
| **로그 레벨** | ❌ 없음 | ✅ ERROR, WARN 등 설정 가능 |
| **출력 위치 제어** | ❌ System.err 고정 | ✅ 파일, 콘솔, 네트워크 등 설정 가능 |
| **로그 포맷 설정** | ❌ 불가 | ✅ 로그백/log4j 설정 가능 |
| **로그 필터링/검색** | ❌ 불편함 | ✅ 로그 수집 시스템에서 용이 |
| **로깅 정책 적용** | ❌ 불가 | ✅ 적용 가능 (rolling, masking 등) |
| **JSON, XML 출력** | ❌ 불가 | ✅ 가능 |
| **ELK, Cloud 연동** | ❌ 어렵다 | ✅ 가능 |
| **운영 적합성** | ❌ 부적합 | ✅ 매우 적합 |
(by. Chat GPT) 

번외) 왜 `e.printStackTrace();`가 간단하다는 거지?

과거에는 로깅 설정 파일에 콘솔 출력을 설정하지 않으면, 콘솔에 출력되지 않았음.

반면에, 현재는 콘솔 출력도 default이기 때문에 그냥 로깅 시스템을 사용하자!