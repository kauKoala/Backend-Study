# 16.mutable 과 immutable

### Immutable을 보장하는 방법

1. 클래스의 하위 클래스화 방지
2. setter 메서드 사용 X
3. **외부 참조를 취하는 생성자 관리**
4. **불변 객체에 대한 참조를 반환하는 Getter를 처리하세요.**
5. **모든 필드를 private final 사용**

### 문제 제기

코드 리뷰 과정에서 immutable 을 아는지란 질문을 받았다.

처음엔 final 키워드를 사용해 불변으로 만드는 것이라 생각했었다.

하지만, 컬렉션을 사용할 때 immutable을 제대로 보장하지 않는다는 걸 알게 됐다.

**한 줄로 요약하면 뒷문이 열려있었다고 보면 된다.**

### 왜 final을 쓰나?

1. gc 오버헤드를 감소시킬 수 있다.

오버헤드가 주제가 아니라 더 다루진 않는다.

링크: https://docs.oracle.com/javase/tutorial/essential/concurrency/immutable.html

2. 코드를 빠르게 읽는다.

이전 상태를 유지하니 기억할 게 줄어든다.

다르게 말하면 **외부에서 수정이 불가능해 객체간 결합도를 줄일 수 있다.**

### 문제가 된 부분

나는 여러 개의 Response를 반환하면 추상화해 ResponseList 객체를 반환한다.

여기서 문제가 발생했다.

```java
class ResponseList {

    private final List<Response> entries;
}
```

### final 선언해도 삽입, 삭제는 가능하다.

Collection에 final을 선언하면 참조 값만 고정되고, 원소 삽입, 삭제는 가능하다.

아래 코드를 보면 entries에 원소 삽입이 가능하다.

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;

public class Main {

    public static void main(String[] args) {
// Response, ResponseList 초기화
        List<Response> entries = new ArrayList<>();
        for (int i = 1; i <= 3; i++) {
            entries.add(new Response(i));
        }
        ResponseList responseList = new ResponseList(entries);
// 참조값 반환
        List<Response> modifiableList = responseList.getEntries();
// 참조값만 있으면 외부에서 삽입, 삭제 가능
        modifiableList.add(new Response(4));
    }
}

class Response {

    private Integer a;

    public Response(Integer a) {
        this.a = a;
    }
}

class ResponseList {

    private final List<Response> entries;

    public ResponseList(List<Response> entries) {
        this.entries = entries;
    }

    public List<Response> getEntries() {
        return entries;
    }
}
```

### Collections.unmodifiableList

해당 키워드를 사용하면 컬렉션 원소 삽입, 삭제가 불가능하다.

ResponseList 객체 생성자 코드를 수정해 런타임 상에서 문제를 예방할 수 있다.

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;

public class Main {

    public static void main(String[] args) {
// Response, ResponseList 초기화
        List<Response> entries = new ArrayList<>();
        for (int i = 1; i <= 3; i++) {
            entries.add(new Response(i));
        }
        ResponseList responseList = new ResponseList(entries);
// 참조값 반환
        List<Response> unModifiableList = responseList.getEntries();
// UnsupportedOperationException 발생
        unModifiableList.add(new Response(4));
    }
}

class Response {

    private Integer a;

    public Response(Integer a) {
        this.a = a;
    }
}

class ResponseList {

    private final List<Response> entries;

    public ResponseList(List<Response> entries) {
// 참조값을 얻어도 삽입, 삭제 불가this.entries = Collections.unmodifiableList(entries);
    }

    public List<Response> getEntries() {
        return entries;
    }
}
```

가장 좋은 방법은 entries 참조 값을 반환하지 않는 거지만

Jackson을 통해 JSON 변환이 불가능하니 범용적인 해결책은 아니라 생각한다.

### 해치웠나?

Response 객체는 아직 mutable 하다.

**즉, 외부에서 entries 원소를 변경할 수 있다.**

다음 코드는 entries의 첫 원소를 1 -> -1로 바꿀 수 있다.

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;

public class Main {

    public static void main(String[] args) {
// Response, ResponseList 초기화
        List<Response> entries = new ArrayList<>();
        for (int i = 1; i <= 3; i++) {
            entries.add(new Response(i));
        }
        ResponseList responseList = new ResponseList(entries);
// 참조값 반환
        List<Response> unModifiableList = responseList.getEntries();

// 첫 원소 가져옴
        Response response = unModifiableList.get(0);
// 원소의 객체 수정 가능
        response.setA(-1);

// -1, 2, 3 출력for (int i = 0; i < 3; i++) {
            System.out.println(unModifiableList.get(i).getA());
        }
    }
}

class Response {

// 외부에서 변경 가능private Integer a;

    public Response(Integer a) {
        this.a = a;
    }

    public Integer getA() {
        return a;
    }

    public void setA(Integer a) {
        this.a = a;
    }
}

class ResponseList {

    private final List<Response> entries;

    public ResponseList(List<Response> entries) {
// 참조값을 얻어도 삽입, 삭제 불가this.entries = Collections.unmodifiableList(entries);
    }

    public List<Response> getEntries() {
        return entries;
    }
}
```

이 경우 멤버 변수 a에 final을 선언해 간단히 해결할 수 있다.

### 3줄 요약

1. 컬렉션에 final을 선언해도 외부에서 삽입, 삭제 가능하다.
2. Collections.unmodifiableList 을 적용해도 내부 원소가 mutable 하면 외부에서 수정 가능하다.
3. 관련 객체도 immutable을 보장해야 한다.

### 그 외 java의 불변 객체들

1. String 클래스
1. Wrapper 클래스
1. UUID, Optional
1. LocalDate, LocalTime, LocalDateTime

### Reference

https://medium.com/@cs.vivekgupta/everything-about-immutable-classes-in-java-9f5fe8e6ca54