## 언제 사용할까

스레드 up이 어느 스레드에서 발생하냐에 따라 달라진다.

뮤텍스: **down과 up이 동일한 스레드에서 발생**해야 한다. 예를 들어, 노드 내의 데이터를 전부 삭제한다면 삭제 과정에서 다른 스레드가 개입하면 안 된다.

세마포어: **다른 스레드에 의해 up이 발생할 수 있다.** 생산자-소비자 예시를 주로 드는데, 소비자 스레드에서 버퍼가 없으면 down 상태가 된다. up이 될 때는, 생산자 스레드에서 시그널을 보낼 때다.

여기서 최대 값이 1인 바이너리 세마포어와 뮤텍스의 차이가 생기는데, 뮤텍스의 경우 동일한 스레드에서 up, down이 발생하지만 세마포어는 다른 스레드에 의해 up이 발생할 수 있단 차이가 있다.

## 어떻게 구현할까

### 생산자-소비자 구조

생산자는 버퍼가 가득 차 있지 않을 때 (세마포어 값 ≠ 0) 버퍼를 채우며, 버퍼가 가득 차면 (세마포어 값 == 0) 대기한다.

소비자는 버퍼가 비어있지 않을 때 (세마포어 값이 ≠ 0) 버퍼를 비우며, 버퍼가 완전히 비어있으면 (세마포어 값 == 0) 대기한다.

이 과정을 통해 한 쪽이 0이 될 떄 까지 생산과 소비를 반복한다.

Q. 그렇다면 카프카도 세마포어 기반인지?

## Thread Pool

DB 커넥션 풀과 같은 개념

필요할 때마다 Sleep 상태의 Worker Thread를 꺠운다. (V 연산)

Master Thread 가 관리하는 버퍼(ex 파일 디스크립터 버퍼)를 소비하며 작업을 처리한다.

여기서 Master Thread는 생산자, Worker Thread는 소비자가 된다.

## 읽기-쓰기 구조

데이터베이스에서 주로 사용

읽기 작업은 값 변경하지 않으므로, 상호 배제를 할 필요가 없지만, 쓰기 작업은 상호 배제가 필요하다.

이 경우 읽기, 쓰기 스레드가 **동시에 접근할 경우 누구에게 우선권을 줄 지**에 대해서 갈린다. 또한, 요청이 수시로 발생하는 상황에선 우선순위가 낮은 쪽이 기아 상태가 될 수 있다.

이 경우 읽기 스레드에 Threshold같은 임계값을 걸어 일정 횟수를 접근하면 쓰기 스레드에 우선권을 주는 식으로 조정할 수 있다.

## Reference

[When should we use mutex and when should we use semaphore](https://stackoverflow.com/questions/4039899/when-should-we-use-mutex-and-when-should-we-use-semaphore)

[Introduction to Priority Inversion](https://barrgroup.com/embedded-systems/how-to/rtos-priority-inversion)

[SP - 5.3 Semaphore Synchronization (2)](https://velog.io/@junttang/SP-5.3-Semaphore-Synchronization-2)