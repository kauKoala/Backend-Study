# 20. 직렬화와 역직렬화

## 직렬화? 역직렬화?
![Serialize](https://github.com/sj7699/Backend-Study/assets/26706925/0be74820-f5a1-4344-9033-b857d22d3618)

직렬화는 객체를 저장 가능한 상태로 바꾸는 과정, 구체적으로는 객체의 상태를 바이트 스트림으로 변환하는 과정

직렬화된 객체는 다시 읽어와서 원래의 객체 상태로 복원할 수 있으며 이를 역직렬화라고 함

## 자바에서 직렬화를 지원하는 법

### Serializable
자바에서는 java.io.Serializable 인터페이스를 구현하여 직렬화를 지원

Serializable은 메소드가 존재하지 않는 마커 인터페이스로 JVM에게 직렬화 가능한 클래스임을 알리는 역할

### ObjectOutputStream

자바에서 객체를 직렬화할때는 ObjectOutputStream을 사용합니다. 

객체가 직렬화 될때는 객체의 인스턴스 값만 직렬화 됩니다.

```java
public static void main(String[] args) {
    // 직렬화할 고객 객체
    Customer customer = new Customer(1, "홍길동", "123123", 40);

    // 외부 파일명
    String fileName = "Customer.ser";

    // 파일 스트림 객체 생성 (try with resource)
    try (
            FileOutputStream fos = new FileOutputStream(fileName);
            ObjectOutputStream out = new ObjectOutputStream(fos)
    ) {
        // 직렬화 가능 객체를 바이트 스트림으로 변환하고 파일에 저장
        out.writeObject(customer);

    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
출처 : Inpa 블로그 (https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)

### ObjectInputStream

자바에서 객체를 역직렬화할때는 ObjectIutputStream을 사용합니다. 

직렬화 대상이 된 객체의 클래스가 외부 클래스라면, Class Path에 존재해야 하며 import 된 상태여야 합니다.
```java
public static void main(String[] args) {
    // 외부 파일명
    String fileName = "Customer.ser";

    // 파일 스트림 객체 생성 (try with resource)
    try(
            FileInputStream fis = new FileInputStream(fileName);
            ObjectInputStream in = new ObjectInputStream(fis)
    ) {
        // 바이트 스트림을 다시 자바 객체로 변환 (이때 캐스팅이 필요)
        Customer deserializedCustomer = (Customer) in.readObject();
        System.out.println(deserializedCustomer);

    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```
출처 : Inpa 블로그 (https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)

### serialVersionUID
serialVersionUID는 직렬화 대상 객체 필드로 선언하여 사용합니다. 클래스에 기술하지 않는다면 JVM 내부적으로 클래스의 메타데이터 정보를 통해 해시값을 생성하고 부여합니다.

```java
  private static final long serialVersionUID = 1L;
```

직렬화된 객체의 클래스가 직렬화 이후 클래스의 구조가 변경된다면, 역직렬화 과정에서 문제가 발생합니다.

수동으로 버전을 관리 할 수 있는 serialVersionUID라는 고유 식별자가 도입된 이유입니다. 클래스의 구조가 변경될 때마다 이 식별자를 명시적으로 업데이트하여 호환성 문제를 방지합니다.

serialVersionUID가 일치하지 않는다면 역직렬화 과정에서 InvalidClassException을 발생시킵니다.

하지만 serialVersionUID를 통해 직렬화 객체의 버전을 관리하여도 문제가 발생할 수 있는데 

**필드 변수의 타입을 바꿨을 때** 입니다. 같은 serialVersionUID 상황에서 멤버 변수 및 메서드 추가, 제거, 이름변경 등은 데이터 누락문제가 있을 수 있어도 예외가 발생하지 않습니다. 하지만 기존 필드 변수의 타입을 변경했을경우 InvalidClassException 가 발생합니다.


### transient

변수에 transient 키워드를 사용하면 직렬화 대상에서 제외할 수 있습니다.

transient 키워드가 포함된 객체를 직렬화하고 이를 다시 역직렬화할 경우 

Primitive Type은 각 타입의 기본값 Reference Type은 null이 됩니다.

## 직렬화를 주로 사용하는 곳

### 세션 상태의 관리

세션 데이터를 DB에 저장하거나 세션 클러스터링으로 다른 서버와 세션 상태를 공유할 때 Servlet 메모리에 있는 세션 정보를 직렬화할 필요가 있었습니다.


### RMI

JVM에서 다른 JVM으로 메소드 호출을 원격으로 수행할 수 있게 해주는 자바에서 지원하는 기술

이때 파라미터에 사용되는 객체, 반환되는 객체를 직렬화 / 역직렬화로 전송합니다.

한 시스템에 많은 프레임워크가 사용되는 현재 환경에서 자바에 종속되어있어 사용되지 않는 것 같습니다.

## 자바 직렬화 하지마라?

```
자바 직렬화의 대안을 찾으라

Serializable을 구현할지는 신중히 결정하라

커스텀 직렬화 형태를 고려해보라

readObject 메서드는 방어적으로 작성하라
- 이펙티브 자바 -
```

```
자바 직렬화는 장점이 많은 기술입니다만 단점도 많습니다.
문제는 이 기술의 단점은 보완하기 힘든 형태로 되어 있기 때문에 사용 시 제약이 많습니다. 그래서 이 글을 적는 저는 직렬화를 사용할 때에는 아래와 같은 규칙을 지키려고 합니다.

1. 외부 저장소로 저장되는 데이터는 짧은 만료시간의 데이터를 제외하고 자바 직렬화를 사용을 지양합니다.

2. 역직렬화시 반드시 예외가 생긴다는 것을 생각하고 개발합니다.

3. 자주 변경되는 비즈니스적인 데이터를 자바 직렬화을 사용하지 않습니다.

4. 긴 만료 시간을 가지는 데이터는 JSON 등 다른 포맷을 사용하여 저장합니다.

- 우아한 형제들 기술블로그 (자바 직렬화 실무편) - 
```

이펙티브 자바, 우아한 형제들 기술 블로그의 자바 직렬화와 관련된 글에서도 모두 직렬화를 신중히 사용하거나 대체할것을 매우 강조하고 있습니다.

직렬화의 문제점들은 다음과 같습니다.


### 직렬화된 객체의 크기가 다른 포맷에 비해 큼

- 자바 직렬화시에 기본적으로 타입에 대한 정보 등 클래스의 메타 정보를 모두 가지고 있습니다.

- JSON은 필드명과 값으로만 이루어져있고 메타 정보가 없기에 매우 가볍습니다

- 같은 데이터로 크기를 비교했을때 2배 정도 차이가 난다고 합니다.

### 자바에서만 사용이 가능함

- Java의 직렬화는 Java언어에 종속적이기 때문에 향후 Node.js, Python 과 같은 프레임워크로 서버 혹은 데이터가 마이그레이션 된다면 지원하는 라이브러리가 존재하더라도 호환성에서 문제가 발생할 수 있습니다.

### 보안

- 직렬화 시 private 멤버 또한 직렬화 대상이기에 외부로 노출됩니다.


- 역직렬화에 사용되는 ObjectInputStream의 readObject() 메소드는 Class Path만 import할 수 있다면 모든 Obejct를 생성자 없이 만들 수 있습니다.

    - 생성자에서 사용했던 로직들 우회하는 점을 이용해 악의적인 공격을 할 수 있습니다.
        - ex) 직렬화된 객체를 외부로 전송할때 공격자가 전송과정에 개입하여 잘못된 객체(생성자를 통해 범위를 제한한 멤버 변수)로 바꿔치기

    -  java.io.ObjectInputFilter로 객체 타입, 범위, 배열 사이즈를 제한하여 무분별한 readObject()를 막을 수 있습니다.

- 역직렬화 폭탄

    - 역직렬화 시간이 오래 걸리는 짧은 스트림을 역직렬화해서 DOS 공격이 가능함
        - 매우 깊은 객체 그래프를 가지는 객체를 역직렬화

    - java.io.ObjectInputFilter로 객체 그래프의 깊이, 참조의 개수를 조건으로 역직렬화 폭탄을 막을 수 있습니다.
 

### 유지 보수와 업데이트 둘 다 어려움

- 직렬화를 지원한 클래스가 다른 클래스에 멤버 변수, 상속에 많이 사용된다면 해당 클래스(직렬화를 지원한 클래스)가 변경될 경우 다른 클래스의 직렬화에도 영향을 미칩니다.

-  직렬화 가능한 클래스가 업데이트되면, 이전 버전들의 직렬화 방식이 현재 버전에서 역직렬화가 가능한지 테스트해야합니다. (테스트 양이 매우 커짐)

## 결론

피할 수 있으면 피하자

바꿀 수 있으면 바꾸자

진짜 못피하겠으면 매우 조심하자

## 참고
https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0

https://techblog.woowahan.com/2551/

https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%9D%EC%B2%B4-%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94-%EB%B0%A9%EC%96%B4-%EA%B8%B0%EB%B2%95-%EC%B4%9D%EC%A0%95%EB%A6%AC-%EB%AA%A8%EC%9D%8C
