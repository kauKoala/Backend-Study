# MVCC 

## MVCC란?

- 하나의 데이터를 여러 버전으로 관리하는 방법



## MVCC는 왜 필요한지

- Lock방식만으로 DBMS가 동시성을 제어하기에는 성능상으로 한계가 존재했음
  - Lock 방식은 Transaction이 특정 데이터에 write나 read하기 위해서 lock을 취득하는 방식
  - Lock의 컨셉은 한 시점의 데이터를 하나의 버전으로 고정하여 쓰기, 읽기 일관성을 보장하는 것
  - Shared Lock과 같이 데이터의 변경을 필요로 하지 않는 작업끼리는 Lock을 공유하지만 나머지 경우에는 동시에 작업이 이루어지지 못함
    - 데이터를 변경될때 사용되는 Exclusive Lock은 해당 데이터에 대해 여러 트랜잭션이 동시에 읽고 쓰지 못하게 된다.
  - 이로 인하여 여러 문제가 발생함
    - 여러 트랜잭션이 다음 작업(read, write)에 필요한 lock을 서로가 다르게 보유하고 있으면서 대기할 경우 DeadLock 발생
    - 오래 점유되는 Lock들이 필요한 여러 작업을 가진 트랜잭션은 계속 Lock을 기다림
    - 인기있는 데이터의 접근의 경우 Lock은 한정되어 있어 너무 많은 트랜잭션이 동시에 접근하여 많은 트랜잭션이 대기하게됨



![img](https://velog.velcdn.com/images/cataiden/post/d4044749-ba15-4f23-8435-af461e4cd821/image.png)



## MVCC의 솔루션

- **MVCC**는 Lock없이 트랜잭션 내에서 일관된 읽기를 제공하는데에 초점이 맞춰져 있다. 

- 이전 발표에서 나왔던 **낙관적 동시성 제어 방법**이 기반이 됨

- 여러 트랜잭션이 동시에 한 데이터에 접근하고 수정할 수 있으며 **변경되었을 경우 해당 데이터의 새로운 버전을 저장**함
  - 이를 통해 Lock없이 트랜잭션내에서 일관된 읽기 보장

- 충돌(두 트랜잭션의 똑같은 버전의 데이터 동시 수정)을 허용하므로 이를 DB Level 밖에서 해결해야함
  - ~~하지만 빨랐죠??~~

![img](https://velog.velcdn.com/images/cataiden/post/d4044749-ba15-4f23-8435-af461e4cd821/image.png)

### MVCC가 지원되는 Isolation Level 

- **Read Uncommited**

  Commit된 데이터가 아니어도 Read가 가능한 격리 수준인 Read Uncomitted에서는 MVCC가 사실상 필요없다.

  (**Oracle**은 해당 격리수준을 지원하지 않는다)

  

- **Read Commited** 

  **Oracle**의 기본 격리 수준

  **Oracle**은 트랜잭션이 데이터를 변경하려할때 Undo 영역(Undo 세그먼트)에 데이터를 저장함

  **MySQL**(InnoDB)도 마찬가지로 Undo 영역(Undo 로그)에 데이터를 저장하나.

  둘다 쿼리를 실행할때마다 Undo 영역의 커밋된 버전의 데이터를 읽어 Read Commited를 보장한다.

  

- **Repeatable Read**

  **MySQL**(InnoDB)의 기본 격리수준

  **Oracle**은 해당 격리수준이 명시적으로 존재 X

  **MySQL**은 **트랜잭션 시작 전에** Undo 로그를 읽어 **Repeatable Read**를 보장한다.

  

- **Serializable**

  **MySQL**에서는 **Serializable**은 MVCC가 아님 

  - **MySQL**의 모든 Lock은 Commit, Rollback후 반환
  - Serializable Level에서는 **읽기, 쓰기 작업 모두 해당 튜플에 Lock을 검**
  - 따라서 MVCC를 지원하지 못함

  **Oracle**은 위에서 설명했듯이 Undo 세그먼트에 트랜잭션이 데이터를 변경할때마다 해당 스냅샷이 저장되기에 

  **트랜잭션 시작 전에** Undo 영역을 읽어 **Serializable**을 보장한다.

  

## 출처

https://velog.io/@cataiden/Isolation-Level-Lock-and-MVCC-in-RDBMS

https://mangkyu.tistory.com/53

https://easy-code-yo.tistory.com/27

