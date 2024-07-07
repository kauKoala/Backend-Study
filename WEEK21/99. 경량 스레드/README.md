## 1. 경량 스레드가 인기 있는 이유

[Traveloka가 경량 스레드를 고려하게 된 이유]

하나의 서비스를 모놀리틱하게 만들게 된다면 서비스의 이용자가 많아질수록 서비스 안에 여러 기능들이 추가되고, 많은 사용자 처리를 위해 그만큼 수많은 스레드가 생성되고 사라짐

요즘은 컴퓨터 하드웨어 스펙이 좋아져서 많은 스레드를 만들 수 있게 되었지만, JVM 기본 스레드 생성 크기는 1MB이기 때문에 동시에 수천개 이상의 요청이 들어오게 된다면 기가바이트(GB)단위의 램을 사용해야 한다는 특징을 가지고 있음

MAU가 4천만명으로 항공권 / 호텔 예약 앱인 Traveloka는 모놀리틱 구조를 가져가는 경우 10,000개의 스레드가 생성되기 때문에 하드웨어 스펙도 높아야하고, 컨텍스트 스위치로 인해 CPU 사용률도 높았음. 이런 문제를 해결하기 위해 일단 비대해진 서비스를 MSA로 분리하게 되고, 앞으로 하드웨어 스케일 업 없이 효과적으로 처리할 수 있는 방법이 무엇인지 찾아보다가 경량 스레드를 고려함

이 부분은 다른 회사도 마찬가지라는 생각이 들음

---

## 2. 선점형 멀티태스킹 / 협력적 멀티태스킹

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/9d3932bd-4df7-45b7-abe7-980fe2a35193)

선점형 멀티태스킹은 인터럽트를 사용하여 실행중인 프로세스를 강제적으로 변경함. 일반적으로 타이머 인터럽트를 사용하는 라운드로빈 알고리즘을 이용하는데, 특정 프로세스의 작업이 처리량이 많다고 하더라도 각 작업이 공평한 CPU 시간을 확보할 수 있도록 보장함

협력적 멀티태스킹은 현재 CPU를 점유하는 프로세스에게 강제로 포기시키지 않는 대신 자발적으로 포기할 수 있도록 기다림. 그래서 모든 프로세스가 협력적으로 활동해야한다는 전제조건을 가지고 있음. 만약 하나라도 협력적으로 활동하지 않는다면 그 프로세스는 자기 혼자 모든 작업을 처리하고 다른 프로세스에게 넘겨줄 것임. 이때 에러가 발생하면 모든 프로세스는 영원히 대기 상태에 빠짐

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/8d852497-dace-488d-a5a0-eb40a334c8b3)

협력적 멀티태스킹의 장점은 선점형과 다르게 일반적으로 컨텍스트 스위칭이 빈도 횟수가 낮은 편임

### [1] 선점형 멀티태스킹의 예시 (Java - ExecutorService)

```java
@Test
void preemptive_concurrency() {
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    CountDownLatch latch = new CountDownLatch(2);

    Runnable task1 = () -> {
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println("Task 1 - step " + i);
            }
        } finally {
            latch.countDown(); // 작업 완료 시 카운트 감소
        }
    };

    Runnable task2 = () -> {
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println("Task 2 - step " + i);
            }
        } finally {
            latch.countDown(); // 작업 완료 시 카운트 감소
        }
    };

    executorService.submit(task1);
    executorService.submit(task2);

    try {
        latch.await(); // 모든 작업이 완료될 때까지 대기
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }

    executorService.shutdown();
}
```

JVM에서는 따로 스레드 스케줄러가 존재하지 않기 때문에 운영체제 스케줄러를 사용하게 되는데, 운영체제 스케줄러는 라운드 로빈 방식을 사용하기 때문에 task1, task2의 순서대로 실행되지 않음

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/ba57ca31-924e-4db9-8423-f38b47afed24)

스레드에게 주어진 시간만큼 작업을 하기 때문에 시작부터 Task 1이 step 0, step 1을 바로 출력하는걸 볼 수 있음

### [2] 협력적 멀티태스킹의 예시 (python - yield)

```python
def task1():
    for i in range(5):
        print("Task 1 - step", i)
        yield  # 상태 저장 및 양보

def task2():
    for i in range(5):
        print("Task 2 - step", i)
        yield  # 상태 저장 및 양보

def cooperative_scheduler(tasks):
    while tasks:
        task = tasks.pop(0)
        try:
            next(task)
            tasks.append(task)
        except StopIteration:
            pass

tasks = [task1(), task2()]
cooperative_scheduler(tasks)
```

yield를 사용하여 자신을 호출한 함수에게 제어권을 넘겨줌

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/1f169bb0-b865-4bff-af25-084555cd69ee)

스케줄러 함수만 잘 만들면 이상적인 실행 순서인 0 → 1 → 2 → 3 → 4를 만들 수 있음

---

## 3. 스레드 모델

### [1] 스레드의 종류

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/4c04fc1c-d91e-47e9-af2d-7a80f8e0785e)

스레드 모델은 크게 Kernel Level Thread와 User Level Thread로 나눌 수 있음

- 커널 수준 스레드: 운영체제에서 관리하는 스레드로 스레드 생성, 스케줄링, 상태 전환을 커널에서 관리함
- 유저 수준 스레드: 애플리케이션 단에서 관리하는 스레드로 운영체제는 유저 수준 스레드가 보이지 않여서 운영체제가 지원하지 않음

여기서 유저 수준 스레드는 주로 경량 스레드 (파이버)라고 부르며 고루틴, 코루틴, 액터, 버츄얼 스레드 등이 있음. 루비나 크리스탈 언어에서는 파이버라는 용어를 그대로 사용함

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/18699c61-213d-4118-90e0-2caab858f226)

이전에 Actor Model을 발표할 때, 2020년에 루비 3.0 버전이 나오면서 루비에도 액터가 나왔다고 했는데, 같이 나온 것이 Fiber임

### [2] 커널 수준 스레드 (1:1 모델)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/d714fda4-0f0c-4303-9732-40487b6bcb61)

유저 수준 스레드와 커널 수준 스레드를 1대1로 매핑하여 유저 수준 스레드가 커널 수준 스레드처럼 운영체제가 관리할 수 있도록 함. 현재 자바의 Thread가 커널 수준 스레드로 1대1 모델을 사용하고 있음

ex) JVM, Rust, Python, Ruby, …

### [3] 유저 수준 스레드 (N:1 모델, Green Thread)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/3e60107d-26d6-4a37-b6ab-162c32ea5e16)

사용자 수준 스레드 여러 개를 단일 커널 수준 스레드에 매핑하여 사용함. 이러면 사용자 수준 스레드는 운영체제 스케줄러에 동작하지 않고, 사용자가 직접 스케줄러와 스레드 관리를 할 수 있음

처음 자바 1.1 버전에서 JVM에서 스레드를 관리하는 Green Thread를 사용했으나, 당시 멀티 코어 시대로 넘어간지가 몇 년 됐는데, 멀티 코어를 제대로 활용하지 못해서 커널 수준 스레드로 변경하게 되었음

❗주의할 점) 그린 스레드. 하이브리드 스레드 용어를 혼용해서 사용하는 곳이 많은데, 요즘의 그린 스레드는 하이브리드 스레드로 이해하는게 좋음

ex) 초기 JVM, 초반 Ruby, …

### [4] 하이브리드 스레드 (N:M 모델)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/b7c2935d-6745-44ae-8993-58daffe18306)

하이브리드 스레드는 커널 수준 스레드와 사용자 수준 스레드의 장점을 결합해서 성능을 최적화 시킨 방식임

- 커널 수준 스레드의 장점: 멀티스레드 지원, 스레드 스케줄링 지원으로 효율적인 자원 관리
- 사용자 수준 스레드의 장점: 빠른 스레드 생성, 낮은 컨텍스트 스위칭

만약 커널 수준 스레드를 사용하게 된다면 1:1 매핑이므로 블로킹이 일어나면 운영체제 스레드까지 블로킹이 된다는 단점이 있음. 하지만 하이브리드 스레드는 사용자 수준 스레드에서 블로킹이 일어난다면 운영체제 스레드는 다른 사용자 수준 스레드를 할당하여 작업을 이어나가기 때문에 효율적인 자원관리가 가능함

ex) goroutine, coroutine, actor, fiber, …

---

## 4. 경량 스레드와 동시성

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/f42aa9fd-6830-4290-821c-da8b61e82305)

하나의 프로세스 안에 여러 개의 스레드가 존재하는 것처럼 하나의 스레드 안에 여러 개의 파이버가 실행될 수 있음

그래서 멀티스레드에서 스레드가 완료되거나, 중지 됐으면 운영체제에게 알리지만, 파이버는 스레드에게 알림

그래서 파이버는..

1) 파이버 자체는 협력적 멀티태스킹, 운영체제 스레드는 선점적 멀티태스킹을 사용해서 관리

2) 하이브리드 스레드 모델을 채택

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/ad129462-fdbc-44eb-99de-c1e478f102c3)

- Kotlin의 Coroutine: **명시적**으로 suspend / resume 관련 명령어를 통해 경량 스레드의 제어권을 조작해서 다른 스레드로 이리저리 옮겨다니며 실행할 수 있음
- Go의 Goroutine: Go 스케줄러가 모든 함수 호출에 대해 필요한 경우 제어권을 양보해서 명시적으로 제어권을 양보할 필요가 없음
- Java의 Virtual Thread: JVM에서 가상 스레드 스케줄러를 통해 필요한 경우 제어권을 양보해서 고루틴과 비슷하게 제어권을 명시적으로 양보할 필요가 없음

[동작 방식 예시]

1) Virtual Thread와 Carrier Thread를 1대1 매핑해서 사용하다가 Virtual Thread에서 Blocking 동작이 발생

2) 이때 전통적인 자바 방식은 Carrier Thread도 Blocking이 일어나는게 정상임

2-1) 하지만 Virtual Thread Scheduling을 통해 Carrier Thread는 Blocking된 Virtual Thread의 연결을 끊고, 새로운 Virtual Thread와 연결하여 Blocking이 되지 않은채로 효율적인 작업을 수행함

3) Blocking 동작이 끝난 Virtual Thread는 Carrier Thread와 연결해서 다시 작업을 수행할 수 있도록 함

❗경량 스레드가 나오게 된 이유는 우리가 CPU 자원을 최대한 효율적으로 사용하려는 것처럼 이제 스레드도 효율적으로 사용하려는 움직임이라고 볼 수 있음

❗경량 스레드의 개념 자체는 어렵지 않은데, Coroutine, Goroutine, Virtual Thread 같은 기술 구현체를 공부하는게 굉장히 어려운 것으로 예상함

## 5. 출처

- [traveloka 기술 블로그 - 협력적 vs 선제적: 동시 접속자 수를 극대화하기 위한 노력](https://medium.com/traveloka-engineering/cooperative-vs-preemptive-a-quest-to-maximize-concurrency-power-3b10c5a920fe)
- [findstar 블로그 - Virtual Thread란 무엇일까? (1)](https://findstar.pe.kr/2023/04/17/java-virtual-threads-1/)
- [2019 NHN Cloud 컨퍼런스 영상 - Java에서 Fiber를 이용하여 동시성 프로그래밍 쉽게하기](https://youtu.be/7H_ROv5rNIg?si=Z8ydy4KaMDc7Xxdb)
