# scale up vs scale out

## scale up

장비 성능을 높임

초기엔 이 방법을 통해 기술 비용을 절감할 수 있다.

장애 발생 시 치명적

### OLTP

단순한 crud 작업을 처리하고 속도가 중요한 경우 scale up이 대체로 유리.

속도와 안정성이 중요하기 때문에 관리 포인트를 줄이는 것이 좋다.

→ scale up이 불가능할 정도로 트래픽이 많다면 읽기 전용 데이터 분리, 캐시 서버 증설 등의 방법이 있다.

### 32bit → 64bit 지원

표현 가능한 데이터 범위가 커진다는 점에서 scale up이라 할 수 있다.

### 주요 기술

- SSD(스토리지 고속화)
- 대역폭 향상 및 포트 확장
- 데이터 압축 및 캐싱

## scale out

장비를 추가해 작업을 분산

데이터 수정 과정에서 정합성이 필요함

장애 발생 시 피해 범위가 scale up에 비해 적음

### 분석용 데이터베이스

데이터를 여러 서버로 분산해 서버당 처리량을 줄인다.

### 대체 경로 확보

트래픽이 몰리는 상황에서 임시 페이지로 경유시킬 때 사용

### **RDMA(Remote Direct Memory Access)**

![Untitled (1)](https://github.com/brorica/board/assets/7845568/279a2fe4-e27a-4aa4-901c-b55e8be7fe63)

데이터를 버퍼에 복사하고, 계층별 헤더를 추가해야 함

이 과정을 생략하고 상대방 메모리로 직접 메모

즉, cpu를 사용하지 않고 데이터를 전달한다.

네트워크 대역폭 효율 증가

단방향 통신이기 때문에 전송 완료 같은 이벤트를 확인하기가 어려움

RDMA가 지원되는 NIC가 있어야 한다.

### 주요 기술

- 로드 밸런싱
- MSA
- 데이터 복제, 이중화

## scale in

scale out 으로 늘린 인스턴스를 다시 줄이는 작업

## Reference

https://www.fibermall.com/ko/blog/what-is-rdma.htm

https://m.blog.naver.com/islove8587/220548900044