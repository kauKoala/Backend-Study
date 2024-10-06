책 범위: 1장, 2장 일부 

## 1. 책 선택한 이유

- 코틀린 배워야해서
- 여러가지 좋은 책들이 있지만 코틀린 인 액션이 제일 언급과 호평이 많았음
- 자바에 대해 알고 있는 사람이 읽으면 좋다고 해서 구매함. 완전한 기초책은 피하고 싶었음

## 2. 1장 : 코틀린이란 무엇이며, 왜 필요한가?

코틀린의 주목적은 자바가 사용되고 있는 곳에 더 간결하고, 생산적이고, 안전한 대체 언어를 제공하는 것

코틀린이 숙련될수록 코틀린만의 짧은 코드로 의미를 부여할 수 있어 생산성을 올릴 수 있다

```kotlin
// 코틀린의 data class = 자바의 record
// val은 getter를 생성하지만, setter는 생성하지 않음 (var은 모두 생성)
// Int?는 널이 존재할 가능성을 의미함
data class Person(val name: String, val age: Int? = null)

// ?:는 엘비스 연산자로 it.age 값이 널이면 0을 반환함 = 자바의 Optional의 orElse(0)
// $는 문자열 템플릿 = 파이썬에서 print(f'나이가 많은 사람: {oldest}')와 같음. 자바는 기본 제공 X
fun main() {
    val persons = listOf(Person("영희"), Person("철수", age=20))
    val oldest = persons.maxBy{it.age ?:0}
    println("나이가 많은 사람: $oldest")
}
```

이외의 내용들

- 기본적으로 자바, 코틀린 모두 타입 명시 언어지만, 둘 다 타입 추론(var) 사용이 가능함
- 자바의 Stream API와 같은 함수형 프로그래밍을 둘 다 지원함
- 자바와 다르게 줄 끝에 세미콜론(`;`)을 붙이지 않아도 댐
- 자바랑 코틀린의 가장 큰 차이는 null point exception이 발생할 수 있는지 컴파일 시점에서 확인 가능

제일 신기한 점

- 코틀린은 표준 라이브러리의 import문이 자동 적용되어 있어서 바로바로 `println(...)`, `arrayListOf(...)`를  사용 가능함

## 2. 코틀린 기초

### [1] 코틀린 함수의 기본 구조

```kotlin
fun max(a: Int, b: Int): Int {
	return if (a > b) a else b
}

fun 함수명 (파라미터: 타입): 반환 타입 {
	return 반환값
}
```

### [2] 변수

```kotlin
var : 타입 추론
val : 불변 (= final)

[1]
val name: "정다빈"

[2]
val name: String = "정다빈"

[3]
val name: String
name = "정다빈"

[타입을 추론할 수 없어서 사용 불가 = 자바도 마찬가지]
var name
```

### [3] 문자열 템플릿

```kotlin
// 파이썬과 유사함
val name: "정다빈"

[1]
println("Hello, $name")

[2]
println("${name}님 안녕하세요")
```

### [4] 클래스

```kotlin
// 자바
public class Person {

	private final String name;
	
	public Person(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
}

// 코틀린
fun Person(val name: String)
```

### [5] 프로퍼티

```kotlin
// 자바
public class Person {

	private final String name;
	private final Boolean isMarried;
	
	... getter ...
}

// 코틀린
class Person (
	val name: String,
	val isMarried: Boolean
)
```

### [6] getter

```kotlin
// 자바 record와 비슷함

[1] 자바의 일반 getter 호출
person.getName();

[2] 자바의 record getter 호출
person.name;

[3] 코틀린
person.name
```

### [7] enum

```kotlin
// 자바
enum Color {
	RED(255, 0, 0),
	BLUE(0, 0, 255);
	
	private final int r;
	private final int g;
	private final int b;
	
	Color(int r, int g, int b) {
		this.r = r;
		this.g = g;
		this.b = b;
	}
	
	public int rgb() {
		return (r * 256 + g) * 256 + b;
	}
}

// 코틀린
enum class Color (
	val r: Int,
	val g: Int,
	val b: Int
) {
	RED(255, 0, 0),
	BLUE(0, 0, 255);
	
	fun rgb() = (r * 256 + g) * 256 + b
]
```

## [8] when (= switch)

```kotlin
// 자바
public String getWarmth(Color color) {
    switch (color) {
        case RED:
        case ORANGE:
        case YELLOW:
            return "warm";
        case GREEN:
            return "neutral";
        case BLUE:
        case INDIGO:
            return "cold";
        default:
            return "unknown";
    }
}

// 코틀린
fun getWarmth(color: Color) = 
	when(color) {
		Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
		Color.GREEN -> "neutral"
		Color.BLUE, Color.INDIGO -> "cold"
	}
```

### 여기까지 읽어 보고 느낀 점..

- 생산성은 확실히 좋은 큰 이유가 읽는 코드가 적어서 피로도가 적어보임
- 근데 사소하게 자바랑 달라서 적응하는데 시간이 좀 걸릴 것으로 보임
- 우테코 프리코스로 직접 코딩도 하고, 다른 사람들 코드 보면서 공부하는게 좋아보임
