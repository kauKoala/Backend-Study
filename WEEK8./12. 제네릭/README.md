# 12. 제네릭

발표자: 이상재
발표 날짜: 2024년 2월 3일

# 12. 제네릭

## 제네릭이란?

제네릭이란 클래스 내부에서 사용할 자료형을 외부에서 정의하는 방식이다.

이펙티브 자바에서는 제네릭을 **타입 파라미터** 라고 한다.

```java
ArrayList<String> list = new ArrayList<>();
```

자바에서는 <> 꺾쇠 표현법을 통해 제네릭을 표현한다.

## 제네릭 탄생 배경

제네릭 도입 전 자바 개발자들이 겪던 문제 중 빈번한 타입 캐스팅이 있었고

특히 컬렉션 객체에서 원소를 추출할 때 타입 캐스팅 문제는 큰 골칫거리였다.

```java
List list = new ArrayList();
list.add("hello"); // String 객체 추가
list.add(10); // Integer 객체 추가, 컴파일 시점에서는 오류를 잡을 수 없음

String str = (String) list.get(0); // 올바른 캐스팅
String num = (String) list.get(1); // ClassCastException 발생
```

잦은 캐스팅과 이로 인해 발생하는 런타임 오류는 개발자의 숙련도와 별개로 많은 문제를 일으켰다.

## 제네릭의 효과

- 타입 안정성
- 캐스팅 작업 최소화
- 코드 재사용성 증가

## 타입 파라미터 지정과 생략

타입 파라미터는 클래스를 인스턴스화할 당시에 지정할 수 있다.

또한 **Primitive Type은 타입 파라미터가 될 수 없다.**

```java
class FruitBox<T> {
    List<T> fruits = new ArrayList<>();

    public void add(T fruit) {
        fruits.add(fruit);
    }
}

FruitBox<Integer> intBox = new FruitBox<>(); 
FruitBox<String> stringBox = new FruitBox<>(); 
```

또한 타입 파라미터는 타입 추론이 가능한 곳에서 생략이 가능하다.

```java
FruitBox<Apple> intBox = new FruitBox<Apple>();

// 다음과 같이 new 생성자 부분의 제네릭의 타입 매개변수는 생략할 수 있다.
FruitBox<Apple> intBox = new FruitBox<>();
```

## 타입 파라미터 컨벤션

![Untitled](https://github.com/sj7699/Backend-Study/assets/26706925/ba79e0ea-651c-48b7-b997-ac68f2dc5c48)


## 제네릭 인터페이스

인터페이스에도 제네릭을 적용할 수 있지만 해당 인터페이스를 구현한 클래스에서도 똑같이 제네릭 타입 파라미터를 선언해야한다.

 오버라이딩한 메소드에도 인터페이스와 동일하게 파라미터에 타입 파라미터를 활용해야한다.

```java
interface ISample<T> {
    public void addElement(T t, int index);
    public T getElement(int index);
}

class Sample<T> implements ISample<T> {
    private T[] array;

    public Sample() {
        array = (T[]) new Object[10];
    }

    @Override
    public void addElement(T element, int index) {
        array[index] = element;
    }

    @Override
    public T getElement(int index) {
        return array[index];
    }
}
```

## 제네릭 메서드

제네릭 메서드 같은 경우 클래스 타입 파라미터를 사용할 수도 있지만

함수 선언 시 반환 타입 앞에 **메서드 만의 독립적인 제네릭**을 선언할 수 도 있다.

메서드에서 별도의 제네릭을 사용할 경우 호출 할때 **파라미터의 자료형을 통해 추론**할 수 있다.

```java
class FruitBox<T> {
	
    // 클래스의 타입 파라미터를 받아와 사용하는 일반 메서드
    public T addBox(T x, T y) {
        // ...
    }
    
    // 독립적으로 타입 할당 운영되는 제네릭 메서드
    public static <T> T addBoxStatic(T x, T y) {
        // ...
    }
}

public class Test{
	public static void main(String args[]){
		FruitBox.addBoxStatic(1,2);
	}
}
```

## 제네릭 타입 한정

타입 매개 변수의 문제는 **범위의 제약이 없어** 개발자가 원하는 기능에 대한 **타입 제한을 만들지 못한다**는 점이다.

따라서 타입 파라미터의 범위를 제한하는 키워드를 제네릭에서는 지원한다.

### 제네릭 와일드 카드

**?** 를 통해 표현하며 제한 없이 어떤 클래스, 인터페이스 타입이든 올 수 있는 걸 뜻한다.

### extends

상한 경계를 뜻하는 키워드로 해당 타입 파라미터의 상한을 제한하는 키워드이며

대표적으로 Java의 Number 타입을 활용한 방법이 있다.

```java
public class NumberProcessor<T extends Number> {
    // 인스턴스 메소드: 제네릭 타입 T의 객체를 받아, 해당 숫자의 제곱을 반환
    public double square(T number) {
        return number.doubleValue() * number.doubleValue();
    }
}

public class Main {
    public static void main(String[] args) {
        NumberProcessor<Integer> intProcessor = new NumberProcessor<>();
        NumberProcessor<Double> doubleProcessor = new NumberProcessor<>();

        System.out.println("Integer 4의 제곱: " + intProcessor.square(4));
        System.out.println("Double 5.5의 제곱: " + doubleProcessor.square(5.5));
    }
}
```

### super

하한 경계를 뜻하는 키워드로 해당 타입 파라미터의 하한을 제한하는 키워드이며

타입 파라미터를 특정 타입의 부모 타입으로 제한하여 부모에 해당하는 기능은 구현되었음을 보장하기 위해 사용된다. 

```java
class Animal {
    void whoAmI() {
        System.out.println("I am an Animal");
    }
}

class Dog extends Animal {
    @Override
    void whoAmI() {
        System.out.println("I am a Dog");
    }
}

public class GenericSuperExample {
    public static void addAnimal(List<? super Animal> animals) {
        animals.add(new Animal());
        animals.add(new Dog()); // Dog도 Animal의 인스턴스이므로 추가 가능
    }

    public static void main(String[] args) {
        List<Animal> myAnimals = new ArrayList<>();
        addAnimal(myAnimals);

        for (Object obj : myAnimals) {
            ((Animal) obj).whoAmI();
        }
    }
}
```

### 언제 <? extends T> <? super T>를 사용해야 하나

**Producer Extends, Consumer Super**

**외부에서 데이터를 생산한다면(Producer), extends를, 외부에서 데이터를 소모한다면(Consumer), super를 사용하라**

```java
public void copyList(List<? extends Reader> in, List<? super Reader> out){
    for (Reader integer : in) {
        out.add(integer);
    }
}
```

## Type Erasure

이전 자바 버전에선 제네릭 문법이 없었기에 타입 파라미터 없이 자바 코드를 컴파일해왔다. 

제네릭 도입 이후에서도 이전 버전에 대한 호환성을 위해 제네릭에 대한 코드는 컴파일 이후 사라지게 된다. 

즉, 컴파일 이후에는 **제네릭 타입 파라미터에 대한 타입이 고정**된다.

### static 멤버 변수의 문제

static 멤버 변수를 제네릭 타입으로 선언할 경우 위에서 설명한 Type Erasure와 static의 특성 때문에 **컴파일 전에 타입이 특정 타입으로 고정** 되어야 하는 **static 멤버 변수는 제네릭을 사용할 수 없다.**

마찬가지로 **static 메서드의 반환 타입으로 제네릭을 사용할 수 없다**.

### 더 알아보기

### 힙 오염

[https://inpa.tistory.com/entry/JAVA-☕-제네릭-힙-오염-Heap-Pollution-이란?category=976278](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%9E%99-%EC%98%A4%EC%97%BC-Heap-Pollution-%EC%9D%B4%EB%9E%80?category=976278)

## Reference

[https://www.youtube.com/watch?v=Vv0PGUxOzq0](https://www.youtube.com/watch?v=Vv0PGUxOzq0)

[https://inpa.tistory.com/entry/JAVA-☕-제네릭Generics-개념-문법-정복하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%ADGenerics-%EA%B0%9C%EB%85%90-%EB%AC%B8%EB%B2%95-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0)

[https://inpa.tistory.com/entry/JAVA-☕-제네릭-타입-소거-컴파일-과정-알아보기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%83%80%EC%9E%85-%EC%86%8C%EA%B1%B0-%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EA%B3%BC%EC%A0%95-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
