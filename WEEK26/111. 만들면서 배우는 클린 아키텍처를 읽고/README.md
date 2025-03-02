책 세부 내용 설명 정리보다는 아래 내용을 적어둠

- 책에서 나온 계층형 아키텍처에서의 아쉬운 점
- 헥사고날 아키텍처에 대한 간단 설명
- 어떻게 코드로 헥사고날 아키텍처를 적용할 수 있는지
- 이걸 토대로 레이어드 아키텍처를 어떻게 발전할 수 있을지

## 레이어드 아키텍처의 아쉬운 점

- 레이어드 아키텍처는 데이터베이스가 중요하기 때문에 디비 > 영속성 > 서비스 > 웹 순서로 자연스럽게 구현이 가능함. 그래서 보통 도메인이 곧 엔티티가 되는 상황임
- 현대 프로젝트는 디비도 중요하지만, 비즈니스 로직도 중요하기 때문에 도메인 ≠ 엔티티인 상황이 많이 발생하고 있음. 하지만 레이어드 아키텍처는 역사상 비즈니스 로직을 고려하지 않은 설계에 가까움
    
    ex) 만약 사람들이 특정 글을 조회하면 추천해주는 ‘같이 많이 본 글’ 기능을 추가한다고 해보자. 그러면 요구사항 자체가 특정 글에 따라 추천 글이 다르고, 시간에 따라 동적으로 추천 글이 달라지는 기능이기 때문에 데이터베이스에 데이터를 저장하기에는 애매한 상황임
    
- 레이어드 아키텍처의 유일한 규칙은 의존성이 위에서 아래로만 이동한다는 점임. 이로인해 규칙이 하나뿐인 아키텍처라 사람들이 나중에 후회하는 지름길을 택할 가능성이 높아짐
→ 이로인해 컨트롤러에서 엔티티를 호출하고.. 메서드가 비대해지고.. 테스트는 어려워지고.. 결과적으로 유지보수하기 어려운 상황이 펼쳐짐

## 그래서 헥사고날은 어떻게 만들까? (패키지 구조)

단순하게 돈을 송금하는 기능만 만든다고 가정함

- 레이어드 아키텍처 (우리가 보통 사용하는 방식)

```java
system
ㄴ controller
  ㄴ AccountController.java
ㄴ service
  ㄴ AccountService.java
ㄴ repository
  ㄴ AccountRepository.java
ㄴ domain
  ㄴ Account.java
  ㄴ TransactionHistory.java (입출금 기록)
```

- 헥사고날 아키텍처

```java
system
ㄴ adaptor
  ㄴ in
    ㄴ web
      ㄴ SendMoneyController.java
  ㄴ out
    ㄴ persistence
      ㄴ AccountPersistenceAdaptor.java
      ㄴ SpringDataAccountRepository.java
ㄴ domain
  ㄴ Account.java
  ㄴ TransactionHistory.java (입출금 기록)
ㄴ application
  ㄴ SendMoneyService.java
  ㄴ port
    ㄴ in
      ㄴ SendMoneyUseCase.java
    ㄴ out
      ㄴ LoadAccountPort.java
      ㄴ UpdateAccountStatePort.java
```

![image](https://github.com/user-attachments/assets/2af01d16-8473-4f26-9ce0-f8d58fd2c68b)

여기에 사용된 용어 설명

- port
    - 외부 세계인 어댑터와 내부 세계인 코어를 분리해주는 경계
- adaptor
    - 외부 의존성을 담당함 (JPA, 레디스, …)
- in / out
    - incoming adaptor: 외부에서 내부로 향하는 흐름을 가진 어댑터 (MVC Controller)
    - outcoming adaptor: 내부에서 외부로 향하는 흐름을 가진 어댑터 (JDBC, JPA)
- usecase
    - 어떤 상황에서 쓰이고, 어떤 행동을 하는지에 대한 클래스 (ex. 돈을 보내는 것이면 SendMoneyUseCase임)

여기까지 알았을 때의 헥사고날의 장점

- ‘상속보다는 조합을 사용하자’ 말을 더 잘 이해할 수 있음
보통 우리는 extends를 사용하든, implements 사용하든 보통 1개의 클래스, 인터페이스만 상속/구현하는 편임
하지만 헥사고날은 하나의 Use Case에서 여러개의 Port를 구현하기 때문에 조합이라는 단어가 더 와닿았음
- 도메인으로부터 외부 의존성 완전히 격리함
여기서 도메인에 관심이 있기보다는 외부 의존성까지 구분지어서 나눈 점이 장점으로 다가옴
레이어드 아키텍처는 JDBC를 사용하든, JPA를 사용하든 모두 Repository라는 말을 사용함
하지만 헥사고날은 JDBC, JPA까지 구분하므로 앞으로 코드를 보게 될 사람이 이해하기 편한 구조라고 느껴졌음

## 헥사고날 코딩은 어떻게 할까?

![image](https://github.com/user-attachments/assets/a05ad872-18d2-4bff-9d58-a34f0eb1110a)

```java
system
ㄴ adaptor
  ㄴ in
    ㄴ web
      ㄴ SendMoneyController.java
  ㄴ out
    ㄴ persistence
      ㄴ AccountPersistenceAdaptor.java
      ㄴ SpringDataAccountRepository.java
ㄴ domain
  ㄴ Account.java
  ㄴ TransactionHistory.java (입출금 기록)
ㄴ application
  ㄴ SendMoneyService.java
  ㄴ port
    ㄴ in
      ㄴ SendMoneyUseCase.java
    ㄴ out
      ㄴ LoadAccountPort.java
      ㄴ UpdateAccountStatePort.java
```

```java
@WebAdaptor
@RestController
public class SendMoneyController {

  private final SendMoneyUseCase sendMoneyUseCase

  @PostMapping
  void sendMoney (@PathVariable ...) {
    ...
    sendMoneyUseCase.sendMoney(...);
  }
}

===

public interface SendMoneyUseCase {
		
  boolean sendMoney(...);
}

===

@UseCase
public class SendMoneyService implements SendMoneyUseCase {

  private final LoadAccountPort loadAccountPort;
  private final UpdateAccountStatePort updateAccountStatePort;
		
  @Override
  public boolean sendMoney(...) {
    ...
  }
}

===

public interface LoadAccountPort {

  Account loadAccount(...);
}

public interface UpdateAccountStatePort(...) {

  void updateTransactionHistory(...);
}

===

@PersistenceAdaptor
public class AccountPersistenceAdaptor implements LoadAccountPort, UpdateAccountStatePort {

  private final SpringDataAccountRepository accountRepository;
  ...
		
  @Override
  public Account loadAccount(...) {
    ...
  }
		
  @Override
  public void updateTranscationHistory(...) {
    ...
  }
}
	
	===
	
interface SpringDateAccountRepository extends JpaRepository<AccountJpaEntity, Long> {
  ...
}
```

- 여기서 신기한 점은 Service는 Port를 호출하는데, Adaptor에서 모두 구현해도 정상 작동함

![image](https://github.com/user-attachments/assets/6379ee1e-4ecc-4621-ac86-6b8f18cc332d)

![image](https://github.com/user-attachments/assets/af3a3552-ca67-4a82-b588-e931347708a1)

## 이걸 토대로 레이어드 아키텍처를 어떻게 개선할 수 있을까?

책을 읽어보며 헥사고날이 말하고 싶은 내용은 크게 두가지로 보임

1. 도메인만큼은 의존성 없게 만들자
2. 클래스, 패키지를 기존에 사용했던 방식보다 더 나눠서 관리하자

그래서 레이어드 아키텍처도 개선할 수 있지 않을까 생각함

1. 레이어드 아키텍처 이전에 멀티 모듈로 역할을 분리하자
2. Controller, Service, Repository를 용도에 따라 분리해본다
    - 간단하게 CRUD 4가지로 나눠서 클래스를 관리할 수도 있어보임
        
        ```java
        account
          ㄴ contorller
            ㄴ CreateController.java
            ㄴ ReadController.java
            ㄴ UpdateController.java
            ㄴ DeleteController.java
        ```
        
    - 이후 서비스가 커진다면 어느 목적으로 쓰이는지 패키지를 나누고, 클래스를 나눌 수 있어보임
        
        ```java
        account
        ㄴ controller
          ㄴ cardregistrationevent
            ㄴ ...
          ㄴ deposit
            ㄴ ...
          ㄴ withdraw
            ㄴ ...
        ```
        
3. 비즈니스 로직을 위한 도메인을 따로 분리하기
    - 도메인 안에 패키지 계층을 하나 두어서 분리할만하다고 생각함
        
        ```java
        domain
        ㄴ persistence
          ㄴ account
            ㄴ Board.java
            ㄴ Comment.java
        ㄴ viewed
          ㄴ recoomendboard
            ㄴ RecommendBoard.java
        ```
        
        아니면 viewed는 repository를 사용하지 않으므로 service 패키지에서 관리하는 것도 좋아보임
        
        이러면 service 패키지에서는 따로 비즈니스 도메인을 분리할만한 패키지가 생성되어야 할 듯
