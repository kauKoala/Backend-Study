# 0. 개요

오버라이딩과 오버로딩은 **다형성**을 실현하는 방법

cf. **다형성**: 하나의 객체가 여러 타입의 인스턴스로 취급될 수 있는 특성으로, 객체지향의 특징 중 하나

# 1. 오버로딩(Overloading)

**같은 이름의 메소드를 클래스 내에 여러 개 정의하는 것**

```java
class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

- 메소드 이름이 같아야 함
- 매개변수의 목록이 달라야 함 (매개변수의 개수, 타입, 순서 등)
- 반환 타입과 접근 지정자는 오버로딩에 영향을 주지 않음
    - 메소드 호출시 반환 타입까지 작성하지 않는 것을 떠올리면 납득이 쉬움

    ```java
    // 컴파일 에러 발생하는 예시 코드
    
    class Calculator {
        public int calculate(int a, int b) {
            return a + b;
        }
    
        public double calculate(int a, int b) {
            return a + b + 0.1;
        }
    }
    ```


cf. C언어에서는 함수명이 고유하게 존재해야 하기 때문에 오버로딩이 없음

`System.out.println()` 대표적인 오버로딩 메소드

# 2. 오버라이딩(Overriding)

**부모 클래스로부터 상속받은 메소드를 자식 클래스에서 재정의하는 것**

- 메소드 이름, 매개변수 목록이 부모 클래스와 일치해야 함
- 자식 클래스의 접근 제어자는 부모 클래스보다 더 좁게(제한적으로) 설정할 수 없음
    - 더 제한적으로 설정한다면, 부모 클래스 타입의 참조 변수로 자식 클래스 인스턴스를 참조했을 때 해당 메소드를 사용할 수 없게 됨.
    - 코드가 아래와 같다고 가정한다면, Parent 클래스 컨텍스트에서는 Child 인스턴스의 private 메소드에 접근할 수 없음

    ```java
    class Parent {
        public void display() {
            System.out.println("Parent");
        }
    }
    
    class Child extends Parent {
        @Override
        private void display() {  // 컴파일 에러 발생
            System.out.println("Child");
        }
    }
    ```

    ```java
    Parent obj = new Child();
    obj.display();  // 컴파일 에러
    ```

- 상속(extends) 혹은 인터페이스(implements)를 통해 오버라이딩 가능
- `@Override` 를 붙이는 것이 권장됨

**@Override를 붙이는 이유?**

해당 어노테이션이 없어도 작동하지만, 컴파일러에게 오버라이드를 알리기 위해 명시함

**장점**

- 부모 클래스가 변경되었을 때, 자식 클래스의 컴파일 오류를 바로 잡아낼 수 있음
- 코드 가독성 향상

**오버라이딩의 리턴 타입은 달라도 되는가?**

- Java 5부터 도입된 '**공변 반환 타입'(covariant return type)** 기능을 사용하면, 오버라이딩된 메소드가 부모 클래스의 메소드보다 더 구체적인 타입을 반환하는 것이 허용됨
- 반환 타입이 객체 타입일 때만 가능하며, 기본 타입에는 적용되지 않음

```java
class Animal {
    Animal get() {
        return this;
    }
}

class Dog extends Animal {
    @Override
    Dog get() {
        return this;
    }
}
```

**참고자료**

- https://hyoje420.tistory.com/14
- https://tcpschool.com/java/java_polymorphism_concept
- https://private.tistory.com/25