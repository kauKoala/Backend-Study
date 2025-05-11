## 트랜잭션 전파(Propagation)란?

- 정의: 현재 트랜잭션이 존재할 때, 호출된 메서드가 이 트랜잭션을 그대로 이어받을지, 새로 만들지 등에 대해 결정하는 정책
- 종류: `REQUIRED` `REQUIRES_NEW` `NESTED` `SUPPORTS` `NOT_SUPPORTED` `MANDATORY` `NEVER`

## REQUIRED

- default 속성
- 기존 트랜잭션이 있으면 참여, 없으면 새로 생성
- 트랜잭션을 공유하기 때문에, 한 곳에서 예외가 발생하면 전체가 롤백됨

  ⇒ 모든 작업이 성공해야만 commit되어야 할 때 사용 (All or Nothing)

- 대부분의 서비스에서 사용되는 속성

## REQUIRES_NEW

- 서로 독립된 트랜잭션 → 하나 실패해도 다른 쪽에 영향 없음

  ⇒ 로그 기록, 알림 발송, 통계 수집 등 실패해도 전체 흐름에는 영향을 주면 안되는 부가 작업에 사용


```java
@Service
public class OrderService {
    @Transactional // = REQUIRED
    public void createOrder() {
        orderRepository.save(order); // 주문

        notifyService.sendNotification(); // 예외 발생 시 전체 롤백
    }
}

@Service
public class NotifyService {
    @Transactional // = REQUIRED
    public void sendNotification() {
		    /* 전송 로직 생략 */ 
		    
        // 예외 발생
        throw new RuntimeException("알림 전송 실패");
    }
}
```

## NESTED

- 같은 트랜잭션 내부에 savepoint를 만들어 동작함
    - 트랜잭션을 새로 만들지 않고 현재 트랜잭션 안에 savepoint를 하나 생성
    - 예외 발생 시 savepoint 이후 작업만 rollback
- JPA에서는 (제대로) 동작하지 않음 → JDBC 전용
- **상위 트랜잭션이 롤백되면 같이 롤백됨** (≠ `REQUIRES_NEW` )

```java
@Service
public class UserService {

    @Transactional
    public void registerUser(User user, Address address) {
        userRepository.save(user);

        try {
            addressService.saveAddress(address); // 트랜잭션 내 savepoint 생성
        } catch (Exception e) {
            // 주소 저장 실패 무시
        }

        validateAndFinalize(user); // 실패라고 가정 
    }

    public void validateAndFinalize(User user) {
        // 검증 실패 시 예외
        if (user.getName() == null || user.getName().isBlank()) {
            throw new IllegalArgumentException("사용자 이름이 비어 있음");
        }
    }
}

@Service
public class AddressService {

    @Transactional(propagation = Propagation.NESTED)
    public void saveAddress(Address address) {
        addressRepository.save(address); // 성공이라고 가정
    }
}
```

비교)

`NESTED`는 트랜잭션을 새로 시작하지 않고, 기존 트랜잭션 안에 savepoint만 생성함 → **비교적 가볍고 빠름**

`REQUIRES_NEW`는 완전히 별도 트랜잭션임 → **커넥션 새로 잡고, 리소스 더 씀**

세 가지 외의 속성 알아보기 ⇒ https://mangkyu.tistory.com/269