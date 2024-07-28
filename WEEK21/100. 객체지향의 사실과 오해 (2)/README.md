## 추상화

(e.g. 지하철노선도)

: 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로써 복잡도를 극복하는 방법

복잡성을 다루기 위해 추상화는 두 차원에서 이뤄진다.

- 첫 번째 차원은 구체적인 사물들 간의 **공통점을 취하고 차이점은 버리는** 일반화를 통해 단순하게 만드는 것이다.
- 두 번째 차원은 중요한 부분을 강조하기 위해 불필요한 세부사항을 제거함으로써 단순하게 만드는 것이다.

## 개념, 타입

**객체**란 특정한 **개념**을 적용할 수 있는 구체적인 사물을 의미한다. 개념이 객체에 적용됐을 때 객체를 개념의 **인스턴스**라고 한다.

**개념**에 다양한 객체들이 속함 (e.g. 트럼프라는 개념에 다양한 하트, 다이아몬드 등 다양한 객체들이 속함)

**개념 = 타입**; 개념처럼 타입도 여러가지 객체들을 포괄할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c85016d6-9aa9-4077-a2c2-27194c83947d/9b76e91e-66cb-4046-87ff-24f3c6e6bfaa/Untitled.png)

클래스와 타입은 동일한 것이 아니다. 타입은 객체를 분류하기 위해 사용하는 개념이다. 반면 클래스는 단지 타입을 구현할 수 있는 여러 구현 매커니즘 중 하나일 뿐이다. 실제로 자바스크립트와 같은 프로토타입 기반의 언어에는 클래스가 존재하지 않는다.

### 역할과 타입의 비교

역할 : 타입 = 일반화 : 특수화

; 특정 타입은 A 역할과 동시에 B 역할을 가질 수도 있다.

## 책임과 메시지

객체지향 세계는 자율적인 객체들의 공동체라는 점을 명심하라. 객체가 자율적이기 위해서는 객체에게 할당되는 책임의 수준 역시 자율적이어야 한다.

자율적인 책임의 특징은 객체가 ‘**어떻게(how)**’ 해야 하는가가 아니라 ‘**무엇(what)**’을 해야 하는가를 설명한다는 것이다.

송신자는 수신자가 메시지를 이해한다면 누구라도 상관하지 않는다. 송신자는 수신자에 대한 어떤 가정도 하지 않기 때문에 수신자를 다른 타입의 객체로 대체하더라도 송신자는 알지 못한다.

(e.g. 판사 - 증언하라())

**What/Who 사이클**

객체 사이의 협력 관계를 설계하기 위해서는 먼저 어떤 행위(what)를 수행할 것인지를 결정한 후에 누가(who) 그 행위를 수행할 것인지를 결정해야 한다. (책임-주도 설계의 핵심)

**묻지 말고 시켜라 (Tell, Don't Ask)**

메시지가 **어떻게** 해야 하는지를 지시하지 말고 **무엇**을 해야 하는지를 요청하는 것

= 객체의 내부 상태를 묻지 말고, 객체에게 원하는 작업을 시키는 것

```java
class Person {
    private int age;

    public Person(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person(20);

        // 나이를 묻고, 외부에서 논리를 처리
        if (person.getAge() >= 18) {
            System.out.println("This person is an adult.");
        } else {
            System.out.println("This person is not an adult.");
        }
    }
}
```

```java
class Person {
    private int age;

    public Person(int age) {
        this.age = age;
    }

    public boolean isAdult() {
        return age >= 18;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person(20);

        // Person 객체에게 직접 성인 여부를 물어봄
        if (person.isAdult()) {
            System.out.println("This person is an adult.");
        } else {
            System.out.println("This person is not an adult.");
        }
    }
}
```