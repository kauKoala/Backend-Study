**by 소트웍스 앤솔러지**

## 1. 한 메서드에 오직 한 단계의 들여쓰기(indentation)만 한다.

한 메서드에 들여쓰기가 많다면, 해당 메서드는 여러가지 일을 할 가능성이 있음.

메서드가 **하나의 일**만 수행하도록 하여, **가독성**과 **유지보수성**을 높일 수 있음.

```java
// AS-IS
public void process(List<Integer> numbers) {
    for (int num : numbers) {
        if (num > 10) {
            System.out.println(num);
        }
    }
}

// TO-BE
public void process(List<Integer> numbers) {
    for (int num : numbers) {
        printIfGreaterThanTen(num);
    }
}

private void printIfGreaterThanTen(int num) {
    if (num > 10) {
        System.out.println(num);
    }
}
```

## 2. else 예약어를 쓰지 않는다.

`else` 를 사용하지 않으면 코드의 복잡성이 줄어듦.

```java
// AS-IS
public String getResponse(boolean approved) {
    if (approved) {
        return "Approved";
    } else {
        return "Rejected";
    }
}

// TO-BE
public String getResponse(boolean approved) {
    if (approved) {
        return "Approved";
    }
    return "Rejected";
}

```

## 3. 모든 원시값과 문자열을 포장(wrap)한다.

원시 데이터를 단순히 사용하지 말고, 클래스로 감싸서 사용하라.

객체로 포장하여 유효성 검사, 행동 추가 등을 할 수 있음

```java
// TO-BE
// 생성자에서 유효성 검사

public class Age {
    private final int value;

    public Age(int value) {
        if (value < 0 || value > 150) {
            throw new IllegalArgumentException("Invalid age: Age must be between 0 and 150.");
        }
        this.value = value;
    }

    public int getValue() {
        return this.value;
    }

    @Override
    public String toString() {
        return "Age: " + value;
    }
}
```

```java
// AS-IS
public class Person {
    private int age;

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age: Age must be between 0 and 150.");
        }
        this.age = age;
    }

    public int getAge() {
        return this.age;
    }
}
```

## 4. 한 줄에 점을 하나만 찍는다.

메서드 체이닝을 제한하여, 각 메서드 호출을 정확하게 나타냄.

cf. 스트림 등의 체이닝은 제외 (내 생각 - 개행이라도 똑바로 하자!)

```java
// AS-IS
public String getFormattedDate() {
    return new SimpleDateFormat("yyyy-MM-dd").format(new Date()).toUpperCase();
}

// TO-BE
public String getFormattedDate() {
    Date now = new Date();
    DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");
    String formatted = formatter.format(now);
    return formatted.toUpperCase();
}

```

## 5. 줄여쓰지 않는다.

명확한 이름을 사용하여 코드의 **가독성** 높이기

```markdown
BatchOperService -> BatchOperationService
```

## 6. 모든 엔티티를 작게 유지한다.

**50줄 이상** 되는 클래스 또는 **10개 파일** 이상의 패키지는 없어야 함

## 7. 2개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.

**인스턴스 변수**? 기본형 또는 자료구조형 객체 (일급 컬렉션이나 wrapper 객체 제외)

```java
// AS-IS
public class User {
    private String name;
    private String email;
    private int age;
		/* 
    이하 생략 
    */
}

// 좋은 예:
public class User {
    private Name name;
    private Email email;
    // 각각의 클래스에 책임 분배 
}

```

## 8. 일급 컬렉션을 사용한다.

**일급 컬렉션?** 다른 멤버 변수 없이 하나의 컬렉션만을 멤버 변수로 가지는 클래스

컬렉션에 대한 비즈니스 로직을 한 곳에 모으고, 불변성을 보장하여 부작용을 줄일 수 있음.

```java
// AS-IS
public class Group {
    private List<Member> members;
    /* 
    이하 생략 
    */
}

// TO-BE
public class Members {
    private final List<Member> members;

    public Members(List<Member> members) {
        this.members = members;
    }

    /* 
    컬렉션 관련 작업 생
    */
}
```

## 9. Getter/Setter/프로퍼티를 쓰지 않는다.

객체의 내부 상태를 숨기고, 메서드를 통해 객체와의 상호작용을 수행함으로써 객체의 캡슐화 유지

```java
// AS-IS
public class Book {
    private String title;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}

// TO-BE
public class Book {
    private String title;

    public Book(String title) {
        this.title = title;
    }

    public void displayTitle() {
        System.out.println("Title: " + title);
    }
}

```

### 참고자료

- https://jamie95.tistory.com/99
- [https://velog.io/@marisol/객체지향-프로그래밍-객체지향-생활체조원칙](https://velog.io/@marisol/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%83%9D%ED%99%9C%EC%B2%B4%EC%A1%B0%EC%9B%90%EC%B9%99)