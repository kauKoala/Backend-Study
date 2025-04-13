# select + @Transactional

단순히 데이터를 조회하는 경우는 트랜잭션을 걸 필요가 없음

읽기 작업은 데이터의 변경이 없기 때문에 롤백이나 커밋이 의미 없고, 성능상 오히려 불리할 수 있음

그렇다면 @Transactional의 사용이 이득인 경우는?

## 1. Lazy Loading 사용 시

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;
}

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional(readOnly = true)
    public List<Order> getUserOrders(Long userId) {
        User user = userRepository.findById(userId).orElseThrow();
        return user.getOrders(); // 여기서 Lazy 로딩 발생
    }
}
```

- `orders`는 `LAZY`로 설정되어 있기 때문에, DB에서 실제 데이터를 가져오는 시점은 `getOrders()` 호출 시점임

  ⇒ 이때 트랜잭션이 열려 있어야 Lazy 로딩이 작동함

- `@Transactional(readOnly = true)`를 안 붙이면 트랜잭션이 닫혀서 `LazyInitializationException` 발생

## 2. 여러 SELECT 쿼리의 일관성 보장 필요

```java
@Transactional
public void checkStockConsistency(String itemCode) {
    int stock1 = productRepository.getStock(itemCode);
    
    Thread.sleep(500); // 외부 API 호출 or 시간 지연

    int stock2 = productRepository.getStock(itemCode);

    if (stock1 != stock2) {
        throw new IllegalStateException("stock1 != stock2");
    }
}
```

- 중간에 외부 호출 등이 들어가면 첫 번째와 두 번째 SELECT 시점에 데이터가 바뀔 수 있음
- 트랜잭션을 걸면 첫 번째 조회 시점의 스냅샷을 유지하므로 `stock1 == stock2`가 보장됨

## 3. 읽기 전용 최적화

```java
@Transactional(readOnly = true)
public void testReadOnly() {
    User user = userRepository.findById(1L).get();
    user.setName("new name"); // DB에 반영되지 않음
}
```

- 자동 flush를 하지 않기 때문에, 개발자의 실수 방지 (DB 반영 방지)
- dirty checking, flush 등을 하지 않기 때문에, 불필요한 트래픽과 CPU 연산을 줄일 수 있음 ⇒ 성능 향상

  cf.

  **dirty checking**: 트랜잭션이 끝나기 직전, 엔티티가 바뀌었는지를 자동으로 감지

  **flush**: Hibernate의 1차 캐시에 저장된 변경 내용을 실제 DB에 반영하는 시점


## 그럼 Mybatis는?

- MyBatis는 Hibernate와 다르게 Dirty Checking이 없음
- LAZY 로딩, 1차 캐시 등의 구조도 없음
- 일부 DB는 SELECT 시 트랜잭션이 열려 있으면, 오히려 락 경합이나 성능 저하가 생기기도 함

  ⇒ Mybatis에서 `@Transactional`은 실질적 효과를 제공하지 못함


but,

2번의 경우와 같이 **여러 SELECT를 Snapshot isolation으로 묶고 싶은 경우**에는 `@Transactional`을 붙이는 것이 맞음