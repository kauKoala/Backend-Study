## **kwargs**

인수를 딕셔너리 형태로 만든다.

하지만 키워드 인자처럼 사용하면 이런 결과가 나온다.

```
def test1(**kwargs):
    print(kwargs)
    pass

data = {'A': 1, 'B': 2}
test1(kwargs=data)
```

결과

```
{'kwargs': {'A': 1, 'B': 2}}
```

### **왜 이런 걸까?**

변수 data **kwargs 매개변수에 언팩(**`A=1`, `B=2`** 형태)되는 것이 아니라 kwargs 라는 하나의 인자로 전달된다.

그래서 **kwargs 에서 언팩 되지 않고 이것을 딕셔너리 형태로 받는다. ( *.*kwargs  와 kwargs 는 다른 거다.)

따라서 {'A': 1, 'B': 2} 가 아닌 {'kwargs': {'A': 1, 'B': 2}} 가 출력된다.

## ***args와 조합하면?**

args도 마찬가지로 키워드 인자로 쓴 args와 test1 매개변수인 *args는 **다른 거다.**

따라서, args='a' 는 **kwargs로 전달된다.

```
def test1(*args, **kwargs):
    print(f'args = {args}')
    print(f'kwargs = {kwargs}')
    pass

data = {'A': 1, 'B': 2}
test1(args='a', kwargs=data)
```

결과

```
args = ()
kwargs = {'args': 'a', 'kwargs': {'A': 1, 'B': 2}}
```

## 함수 호출과 정의 단에서 동작 정리

함수 정의(= def test1(**kwargs) ): 인자들을 딕셔너리로 패킹

함수 호출(= test2(**kwargs) ): 딕셔너리를 키워드 인자들로 언패킹

```
def test1(**kwargs):          # 1
    print(f'test1: {kwargs}') # 2
    test2(**kwargs)           # 3

def test2(**kwargs):          # 4
    print(f'test2: {kwargs}') # 5

data = {'A': 1, 'B': 2}
test1(kwargs=data)
```

결과

```
test1: {'kwargs': {'A': 1, 'B': 2}}
test2: {'kwargs': {'A': 1, 'B': 2}}
```

\#1: kwargs = {'A': 1, 'B': 2} 인자를 딕셔너리 형태로 패킹

\#2: 딕셔너리로 패킹된 kwargs를 출력

\#3: 딕셔너리로 패킹된 kwargs를 키워드 인자들로 언패킹

\#4: #1과 같음

\#5: #2와 같음