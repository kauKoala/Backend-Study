# 1. 싱글톤이란?

하나의 인스턴스만 생성하는 **디자인 패턴** (필요 시에 기존 인스턴스 재활용)

- 어떤 클래스의 인스턴스가 정확히 하나만 존재하도록 보장하는 것을 목표로 함
- 생성자가 여러 번 호출돼도, 실제로 생성되는 객체는 하나
- 생성자를 private으로 선언하여 클래스 외부에서 생성하지 못하도록 만듦

  (해당 인스턴스를 return하는 메서드를 구현하는 방식)


# 2. 구현 방식

### **Eager Initialization (즉시 초기화)**

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {} 

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

- thread-safe
- 인스턴스가 실제로 사용되지 않아도 메모리를 차지함

### **Lazy Initialization (지연 초기화)**

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

- **Eager Initialization**의 불필요한 메모리 차지 문제 해결
- 멀티스레드 환경에서 동시성 문제 발생 가능

### Thread Safe Lazy **Initialization (스레드 안전 지연 초기화)**

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

- **`synchronized`** 키워드를 사용하여 thread-safe하게 구현 가능
- but, 해당 키워드는 큰 성능 저하를 발생시키므로 권장되지 않음

### **Double-checked Locking (이중 체크 잠금 방식)**

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

- 인스턴스 확인 후 **`synchronized`** 를 검사함으로써 성능 향상
- `volatile`(Java 5 이상)을 사용해야 올바르게 동작함
    - **`volatile`**로 선언된 변수는 모든 스레드에게 항상 가장 최신의 값을 보장함

### **Bill Pugh Singleton Implementation (빌 퓨 방식)**

```java
public class Singleton {
    private Singleton() {}

    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

- **`static inner class`** 사용
    - 외부클래스가 로드될 때 로드되지 않고, getInstance()가 호출될 때 로드됨 ⇒ Lazy
- 가장 많이 추천되는 싱글톤 패턴 구현 방식

# 3. 장단점

### 장점

- 객체를 재사용으로 인한 시스템 자원 절약 (메모리 낭비 방지)
- 공통 리소스나 서비스에 대한 접근 용이

### 단점

- 전역 객체에 대한 의존성이 높아짐. 이에 따른 결합도 증가
- 멀티스레드 환경에서의 동기화 처리의 복잡성