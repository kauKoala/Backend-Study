## 0. Compare and Swap의 한계 (CAS)

이전 글에서 CAS에 대해 다루었는데, CAS의 단점은 단일 값만 원자성을 보장해주는 알고리즘이다.

트랜잭션처럼 여러 개의 값을 수정하는 과정은 Multi Compare and Swap(MCAS) 기술이 필요한데, 하드웨어적으로 구현하기가 굉장히 힘듬. Intel에는 Double Compare and Swap 기술이 적용된 명령어가 있으나, AMD는 찾아봐도 안보인다.

이후 Hardware Transactional Memory가 언급됐으나, 구현하는데 굉장히 어려워서 잘 사용되지 않는 기술로 보임

- Intel에서는 사용할 수 있도록 했었으나, 꾸준히 나오는 버그 및 취약점으로 인해 현재는 비활성화된 상태
- AMD는 기술 언급은 했으나 실제 적용된 제품은 없어보임

## 1. STM이란?

데이터베이스의 트랜잭션 개념을 메모리에 적용하여 트랜잭션의 원자성을 보장하기 위한 기술임

2000년대 초반에 멀티 코어 시대로 넘어가면서 성능 향상을 위해 병렬 프로그래밍이 필수적이 됐음

작동 방식이 MySQL MVCC 기술과 유사하다.

[트랜잭션 시작]

트랜잭션을 반영하기 위해 로그를 저장하는 임시 공간을 생성 (= Undo Log)

[트랜잭션 실행]

트랜잭션 연산시 임시 공간에 있는 값을 변경하여 원본 값에 영향을 주지 않음

트랜잭션이 끝나야 실제 값에 반영하므로 격리성을 보장함

[트랜잭션 커밋]

수정하려는 데이터가 트랜잭션 실행 중간에 바뀐적이 없는지 확인하고, 바뀐게 없으면 원자적 연산을 수행함. 낙관적 방식으로 진행하기 때문에 충돌이 거의 없다고 생각하지만, 만약 중간에 바뀐값을 발견하면 완전히 처음부터 재시작함. 메모리 저장 속도를 생각하면 눈 깜짝할 사이에 업데이트가 일어나기 때문에 원자적으로 이루어짐

## 2. 간단한 예제

은행 계좌 예제

[객체 선언]

```jsx
class Account {
    Int balance;
    synchronized void withdraw(int n) {
        balance = balance - n;
    }
    void deposit(int n) {
        withdraw(-n);
    }
}
```

[이체 메서드]

```jsx
void transfer(Account from, Account to, Int amount) {
    from.withdraw(amount);
    to.deposit(amount);
}
```

- 해당 메서드에는 문제점이 존재함
    
     `from.withdraw(amount)`와 `to.deposit(amount)` 사이에 다른 스레드가 중간 결과를 관찰해서 수정할 수 있음
    

[개선한 이체 메서드]

```jsx
void transfer( Account from, Account to, Int amount ) {
		from.lock(); to.lock();
		from.withdraw( amount );
		to.deposit( amount );
		from.unlock(); to.unlock(); 
}
```

- 여기서도 문제점이 발생함
    
    만약 from → to로 돈을 보내는 스레드와 to → from으로 돈을 보내는 스레드가 동시에 작동하면 데드락이 발생
    

[데드락을 피하는 이체 메서드]

락을 거는 규칙을 따로 정해서 데드락을 피하도록 함 (ex. id 번호)

```jsx
if (from < to) {
    from.lock();
    to.lock();
} else {
    to.lock();
    from.lock();
}
```

- 하지만 해당 메서드조차 금융권에서는 서비스를 위해 더 복잡한 상황도 발생해서 문제가 발생할 수 있음
    - 국민은행 계좌 1에 돈이 부족하다면 계좌 2에서 돈을 가져오거나
    - 토스 - 후불계좌 + 원래 계좌로 분할 결제를 하거나

## 3. 락은 문제가 있다.

락의 경우, 사용자가 잘 설계해야하고, 복잡한 상황에서도 다시 재설계를 해야할 상황이 존재한다. 다음의 문제 케이스 등이 있다.

- 락을 너무 적게/많게 사용하는 경우
- 잘못된 락을 사용하는 경우
- 잘못된 순서로 락을 거는 경우
- 이외에 여러가지

거기다가 락 기반 프로그래밍은 모듈화하기가 힘든데, 특히 락을 설정, 해제 코드를 모두 노출해야하는 단점

(90년대 ~ 00년대에 스레드 기반 프로그래밍이 매우 어렵다는 내용들이 활발하게 공유되었음)

## 4. 하스켈에서의 STM

프로그래밍 언어로 STM이 구현된 것은 하스켈이 처음이고, 해당 논문을 기점으로 다양한 프로그래밍 언어에 STM이 구현되었음

```haskell
[하스켈 메서드]
transfer :: Account -> Account -> Int -> IO ()
-- from에서 to로 amount를 이체함
transfer from to amount
= atomically (do { deposit to amount
								 : withdraw from amount})

[자바 메서드]
void transfer(Account from, Account to, Int amount)
```

[atomically 키워드]

- 원자성: 내부 블록이 다른 스레드에게는 한 번에 보임
- 고립성: 다른 스레드의 영향을 받지 않음

[deposit, withdraw 메서드 구현]

```haskell
type Account = TVar Int
withdraw :: Account -> Int -> STM ()
withdraw acc amount
= do { bal <- readTVar acc
     ; writeTVar acc (bal - amount) }

deposit :: Account -> Int -> STM ()
deposit acc amount = withdraw acc (-amount)
```

- T~로 시작하는 트랜잭션 변수명은 외부에서 읽거나, 쓰는 것을 방지함
- STM()은 원자적 연산을 수행할 수 있도록 함

[blocking, choice]

온전한 병렬 프로그램을 만들기 위해서는 추가적으로 blocking과 choice가 필요함

- blocking

```haskell
limitedWithdraw :: Account -> Int -> STM ()
limitedWithdraw acc amount
= do { bal <- readTVar acc;
			 if amount > 0 && amount > bal
			 then retry
			 else writeTVar acc (bal - amount) }
```

하스켈에서는 `retry` 단일 함수를 사용해 현재 트랜잭션은 포기하고, 다시 시도함. 이때 바로 다시 시도하는건 오버헤드만 커질 가능성이 높으므로 하스켈 내부에서 다른 스레드가 쓸 때까지 기다리도록 블로킹함

```haskell
limitedWithdraw :: Account -> Int -> STM ()
limitedWithdraw acc amount
= do { bal <- readTVar acc
     ; check (amount <= 0 || amount <= bal)
     ; writeTVar acc (bal - amount) }
```

`retry`가 들어간 코드는 실제 구현시 복잡도를 높이게 할 수 있음. 따라서 `check(...)` 를 사용해 `check(...) == False`면 `retry`를 호출하도록 함

- choice

```haskell
limitedWithdraw2 :: Account -> Account -> Int -> STM ()
-- acc1에서 돈을 인출하는 상황
-- 만약, acc1에 돈이 없으면 acc2에서 돈을 인출함
-- 만약, acc1과 acc2 둘 다 돈이 없으면 재시도를 수행함
limitedWithdraw2 acc1 acc2 amt
= orElse (limitedWithdraw acc1 amt) (limitedWithdraw acc2 amt)
```

`retry`와 `check`는 현재 조건에 부합하면 실행하거나, 다시 재시도하는 로직만 존재함

여기서는 `orElse`라는 액션을 추가해서 액션1, 액션2, 재시도 상황까지 만들어냈음

<retry 재시도 횟수 제한>

exponential backoff도 설정할 수 있고, limit도 설정할 수 있음 ([exponential backoff란?](https://github.com/kauKoala/Backend-Study/tree/master/WEEK7/64.%20%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%20%EC%97%90%EB%9F%AC%20%ED%95%B8%EB%93%A4%EB%A7%81))

## 5. 결론

Lock-free 알고리즘

- 락 획득 설계에 대한 고민을 하지 않아도 된다.
- 라이브락, 데드락 문제가 발생하지 않는다.
- 높은 부하에도 성능이 좋다.

다음 시간엔 동시성 모델인 actor에 대해 다룰 예정

## 출처

---

- [넥슨 개발자 컨퍼런스 2014 시즌 2: 멀티스레드 프로그래밍이 왜 이리 힘든가요? (Lock-free에서 Transactional Memory까지)](https://www.slideshare.net/slideshow/ndc2014-2/35357131)
- [ZIO Framework STM](https://zio.dev/reference/stm/)
- [Beautiful Concurrency](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/beautiful.pdf)
- [Haskell Documentation Control.Retry](https://hackage.haskell.org/package/retry-0.9.3.1/docs/Control-Retry.html)
- [9가지 프로그래밍 언어로 배우는 개념: 5편 - 동시성 프로그래밍 (메모리 공유부터 읽는걸 추천)](https://tech.devsisters.com/posts/programming-languages-5-concurrent-programming/)
- 같이 공부하면 좋은 내용: JPA 낙관적 락, MySQL MVCC + Undo Log
