## 1. GC(Garbage Collection)

---

### **Garbage란?**

유효하지 않은 메모리

= 프로그램이 더 이상 사용하지 않는 메모리/리소스

### **Garbage Collection**

- Garbage를 자동으로 찾아내어 해제하는 기능
    - Garbage Collector가 그 기능을 수행
- 메모리 누수를 방지하고, 프로그램의 안정성과 효율성을 높임

## 2. 메모리 종류

---

JVM Heap의 두 가지 세부 영역에 따라 GC가 다름

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c85016d6-9aa9-4077-a2c2-27194c83947d/9c2daf93-bff8-4b89-b27e-a47b3bdd3ce3/Untitled.png)

### Young 영역 (Young Generation)

- 새롭게 생성된 객체가 할당되는 영역
- 대부분의 객체는 금방 **접근 불가능한 상태 (Unreachable)**가 되기 때문에, 많은 객체가 Young 영역에서만 있다가 사라짐
- **Minor GC:** Young 영역에 대한 GC

### Old 영역 (Old Generation)

- Young 영역에서 **살아남은 객체 (Reachable 상태 유지)**가 복사되는 영역
- Young 영역보다 크게 할당
    - Young 영역의 수명이 짧은 객체는 큰 공간을 필요로 하지 않음
    - 큰 객체들은 Old 영역에 바로 할당됨
- **Major GC**: Old 영역에 대한 GC

## 3. GC 동작 순서

---

### [GC 공통]

메모리 구조(Young, Old)에 상관없이 실행되는 공통적인 동작

### Stop The World

- JVM이 애플리케이션 실행을 멈추는 작업
- GC를 실행하기 위해, GC 담당 스레드를 제외한 타 스레드의 작업 중단
- `GC 튜닝` 은 일반적으로 Stop The World 시간을 줄여 성능 개선하는 일을 뜻함

### Mark and Sweep

**Mark**: 사용되지 않는 메모리를 식별하는 작업

**Sweep**: 사용되지 않는 메모리로 식별(Mark)된 메모리를 해제하는 작업

- 즉, Stop The World → Mark → Sweep 순서로 진행

메모리 구조(Young, Old)에 따라 다른 세부적인 동작

### [GC 개별]

### Minor GC

**Young 영역 구조**

- 1개의 Eden 영역과 2개의 Survivor 영역
    - Eden: 새로 생성된 객체가 할당되는 영역
    - Survivor: 최소 1번의 GC를 거치며 살아남은 객체가 존재하는 영역

**동작 방식**

1. Eden 영역에 새로 생성된 객체 할당
2. **Eden 영역이 꽉차면 Minor GC 실행**
    1. 사용되지 않은 객체: 메모리 해제
    2. 살아남은 객체: 1개의 Survivor 영역으로 이동
3. 1~2의 과정이 반복되다가 Survivor 영역이 가득 차면, 살아남은 객체는 다른 Survivor 영역으로 이동
4. 위의 과정을 반복하여 살아남은 객체는 Old 영역으로 이동 (Promotion)

### Major GC

- 계속되는 Promotion으로 인해 Old 영역의 메모리가 부족해지면 발생
- Young 영역과 비교하여 크기가 크기 때문에, Minor GC보다 시간이 오래 걸림 (10배 이상의 시간)
- Full GC: Minor GC와 Major GC가 동시에 처리되는 것

## 4. 장단점

---

### 장점

- 개발자가 메모리 할당 및 해제를 수동으로 관리할 필요가 없어, 메모리 누수와 같은 오류를 줄일 수 있음
    - C언어에서는 free()라는 함수를 통해 직접 메모리를 해제해야 함
- 개발자가 메모리 관리에 신경 쓰지 않아도 되므로, 개발 생산성 증가

### 단점

- GC가 언제 실행될지 예측하기 어렵기 때문에, 실시간 시스템이나 응답 시간이 중요한 애플리케이션에서 문제가 될 수 있음
- GC가 실행될 때 시스템의 성능이 일시적으로 저하될 수 있으며, 특히 Major GC의 경우 더욱 뚜렷

**참고자료**

https://mangkyu.tistory.com/118

https://coding-factory.tistory.com/829