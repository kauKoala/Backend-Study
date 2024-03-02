# 67. JPA Auditing

발표자: 이상재
발표 날짜: 2024년 2월 17일

인턴 이O재가 혼난 이유를 찾으시오

![%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%86%AB%E1%84%89%E1%85%B1%E1%86%B8%E1%84%80%E1%85%AA%E1%84%8C%E1%85%A6-5](https://github.com/sj7699/Backend-Study/assets/26706925/498d1208-4bd4-4813-8f41-badb782062aa)


## JPA Auditing 이란?

Auditing = 감시하다라는 의미 

Entity를 감시하고 있으면서 해당 Entity가 DB에 저장되거나 업데이트 될때 개입하여 메타데이터(생성일시, 수정일시, 생성자, 수정자)를 추가한다.

우리가 흔히 알고 있는 위의 Auditing은 Spring Data JPA의 Auditing 기능이다.

JPA 인터페이스 자체는 위 Auditing을 지원하지 않는다.

## 생성자 수정자는 왜 필요한가?

- 로깅, 모니터링
  - 비즈니스 로직에 사용되는 작성자, 수정자와 별개로 테이블마다 존재

## @EnableJpaAuditing

- @EnableJpaAuditing의 위치
  - SpringBootApplication의 메인
    - Spring Context를 실행할때마다 필수적으로 Auditing을 하기때문에 테스트 데이터를 생성할때(임의의 엔티티 생성) 문제가 생길 수 있음
  - 위 이유때문에 따로 JPA 설정을 위한 Configuration을 만들고 @EnableJpaAuditing을 하는 것을 선호함

## @EntityListeners

JPA는 로깅과 감사하는 api를 자체적으로 제공하지는 않고 엔티티 생명주기 콜백을 제공함

- @PrePersist
- @PostPersist
- @PreUpdate
- @PostUpdate
- @PreRemove
- @PostRemove
- @PostLoad

위 생명주기 콜백은 다양한 비즈니스(라고 말하고 복잡한,,,) 요구사항을 구현하다보면 사용하게 되는 기능들이다.

@EntityListeners를 활용하여  위 생명주기 콜백을 활용한 기능과 엔티티를 분리한다. (공통되는 중복 코드 제거)

```java
@EntityListeners(MyEntityListener.class)
public class MyEntity {

}

public class MyEntityListener {
    @PrePersist
    public void prePersist(MyEntity myEntity) {

    }
}
```

## Spring Data의 JPA Auditing

다시 주제로 돌아와서 Spring Data의 JPA Auditing을 활용할때의 코드를 같이 보자

```java
@EntityListeners(value = {AuditingEntityListener.class})
@MappedSuperclass
@Getter
public abstract class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdTime;

    @LastModifiedDate
    private LocalDateTime modifiedTime;
}
```

위 코드는 흔히 사용하는 공통 엔티티를 구현한 코드이다. EntityListeners에서 AuditingEntityListener.class를 인자로 받는걸 볼 수 있다. 

이제 Spring Data에서 어떻게 Auditing을 구현한지 감이 잡힌다.

Spring Data에서 Auditing을 지원하는 두가지 방법

## 어노테이션 기반 Auditing

상속받을 BaseEntity에 선언하는 방법

클래스 멤버로 임베디드 하기

```java
class Customer {

  private AuditMetadata auditingMetadata;

  // … further properties omitted
}

class AuditMetadata {

  @CreatedBy
  private User user;

  @CreatedDate
  private Instant createdDate;

}
```

## 인터페이스 기반 Auditing

만약 createdBy와 updatedBy를 사용하고 싶다면 AuditorAware<>를 구현해야함 

AuditorAware의 제네릭 타입은 getCurrentAuditor()의 반환값

즉, 현재 엔티티에 수정을 가하거나 생성을 하려는 주체에 대한 정보의 타입에 따라 달라짐

스프링 시큐리티의 예시,,

```java
class SpringSecurityAuditorAware implements AuditorAware<User> {

  @Override
  public Optional<User> getCurrentAuditor() {

    return Optional.ofNullable(SecurityContextHolder.getContext())
            .map(SecurityContext::getAuthentication)
            .filter(Authentication::isAuthenticated)
            .map(Authentication::getPrincipal)
            .map(User.class::cast);
  }
}
```

세션을 이용한 예시,,

```java
import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.AuditorAware;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpSession;
import java.util.Optional;

@RequiredArgsConstructor
@Component
public class LoginUserAuditorAware implements AuditorAware<Long> {

    private final HttpSession httpSession;

    @Override
    public Optional<Long> getCurrentAuditor() {
        SessionUser user = (SessionUser) httpSession.getAttribute("user");
        if(user == null)
            return null;

        return Optional.ofNullable(user.getId());
    }
}
```

참조

[https://velog.io/@minji104/CreatedBy-UpdatedBy-기능-구현](https://velog.io/@minji104/CreatedBy-UpdatedBy-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84)

[https://docs.spring.io/spring-data/jpa/reference/auditing.html](https://docs.spring.io/spring-data/jpa/reference/auditing.html)

[https://velog.io/@yun8565/JPA-Auditing-정리](https://velog.io/@yun8565/JPA-Auditing-%EC%A0%95%EB%A6%AC)

[https://www.baeldung.com/database-auditing-jpa](https://www.baeldung.com/database-auditing-jpa)
