## call by value

자바와 JS에서 사용하는 방식

객체 전달의 경우, 객체의 참조값을 보낸다는 점에서 call by reference라 생각할 수 있다.

하지만 참조값을 “**복사**” 해 전달하는 점에서 call by value다.

자바의 경우 객체를 읽을 땐 참조가 같지만, 수정하면 객체를 복사해서 생성한다.

참조값을 조작해도 원본 객체에 영향이 가지 않는다.

```java
public class Main {

    public static void main(String[] args) {
        Integer a = 3;
        System.out.println(System.identityHashCode(a));
        func(a);
        System.out.println(System.identityHashCode(a));
    }

    private static void func(Integer a) {
        System.out.println(System.identityHashCode(a));
        a = 4;
        System.out.println(System.identityHashCode(a));
    }
}
```

Output

```java
2003749087
2003749087
1324119927
2003749087
```

JS의 경우 참조값을 통해 원본 객체를 수정할 수 있다.

하지만, 다른 참조값으로 덮어쓰면 원본 객체는 변동이 없다.

코드를 보면 10을 20으로 바꾸면 **원본 객체도 변경**되지만

아예 다른 객체로 변경하면 **원본 객체는 변경이 없다.**

```jsx
let obj = { value: 10 };

function changeValue(obj) {
    obj.value += 10;
}

changeValue(obj); // 함수 내에서 매개변수(객체의 참조값)를 통해 객체의 속성을 변경
console.log(obj.value); // 원래 객체의 속성값이 변경됨, 출력: 20

let obj2 = { value: 10 };

function changeObject(obj2) {
    obj2 = { value: 20 };
}

changeObject(obj2); // 함수 내에서 매개변수(객체의 참조값) 자체를 변경
console.log(obj2.value); // 그러나 원래 객체는 변하지 않음, 출력: 10
```

call by reference

**큰 구조의 데이터를 인수 하나로 전달할 수 있어 성능에 좋다**

대부분의 언어가 call by value 방식을 사용하는 이유가 원본 참조값이 덮어씌워지는 불상사를 막기 위해서가 아닐까 라고 생각한다

## call by name

함수형 언어에서 사용하는 방식

인자가 즉시 계산되는 call by value와 달리, 함수 내에서 해당 인자가 사용될 때 계산을 수행하는 방식

지연 평가(Lazy evaluation)에 유리하다.

함수형 프로그래밍에서 많이 사용되지만, 디버깅 과정에서 값을 빠르게 확인하기 어렵다.

간단한 예시를 들면

```jsx
function example(x) {
    return x + x;
}

result = example(3 + 2);
```

example(3 + 2) 호출시, 3 + 2 는 바로 계산되지 않고

return x + x 에서 계산이된다.

### 언제 쓸까?

응답이 오래 걸리는 인자를 보내지만, 분기를 타면 사용할 수도 있고 안 할 수도 있는 상황에서 쓰임

코드 단으로 해결할 수 있지 않을까? 라고 생각을 했지만, 2개 이상의 인자가 계산 시간이 오래 걸린다면 call by name 사용이 적합하다.

## Reference

[Is Java "pass-by-reference" or "pass-by-value"?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)

[Java is Pass-by-Value, Dammit!](https://www.javadude.com/articles/passbyvalue.htm)

https://medium.com/@lazysoul/%EC%9D%B4%EB%A6%84%EC%97%90-%EC%9D%98%ED%95%9C-%ED%98%B8%EC%B6%9C-%EA%B0%92%EC%97%90-%EC%9D%98%ED%95%9C-%ED%98%B8%EC%B6%9C-call-by-name-call-by-value-5598aa89fdaa