# Producer-Consumer

## 1. Producer-Consumer 패턴이란?

- 정의: 데이터를 생산하는 쪽(Producer)과, 그 데이터를 소비하는 쪽(Consumer)을 분리하고, 둘 사이를 **버퍼(Queue)**로 연결하는 디자인 패턴
- 목적: 생산과 소비의 속도 차이를 자연스럽게 조정하고, 두 작업을 **비동기적**으로 독립시켜 전체 시스템의 효율을 높이는 것

## 2. 왜 필요한가

Producer가 데이터를 생성하는 속도와 Consumer가 데이터를 처리하는 속도가 다를 때 문제 발생

- 생산이 소비보다 빠르면? 데이터가 넘쳐서 메모리 부족, 시스템 장애
- 소비가 생산보다 빠르면? 리소스 낭비 (Consumer 놀고 있는 상태)

직접적으로 서로를 호출하면 두 컴포넌트가 강하게 결합되어 유연성이 떨어지므로,

중간에 **Queue**를 두고 decoupling(느슨한 연결)을 하는 방식으로 처리

## 3. 구조

```markdown
+-----------+      +------------+       +------------+
|  Producer | ---> |    Queue   | --->  |  Consumer  |
+-----------+      +------------+       +------------+
```

- Producer: 데이터를 생성해서 Queue에 집어넣음
- Queue (Buffer): 데이터를 임시로 저장하는 공간. 생산과 소비를 이어주는 다리 역할.
- Consumer: Queue에서 데이터를 꺼내어 처리



## 4. 예시

엑셀의 데이터를 Stream으로 읽어와, DB에 삽입하는 프로그램

cf. `ExcelSheetHandler.readExcel()`; 엑셀 파일을 읽는 **Producer**

`insertUploadFile()`; 객체를 insert하는 **Consumer**

```java
int batchSize = 100;
int threadCount = 4;

BlockingQueue<List<ExcelVO>> queue = new LinkedBlockingQueue<>(batchSize * 2);

// Consumer 스레드를 관리할 ExecutorService 생성
ExecutorService executor = Executors.newFixedThreadPool(threadCount);

// 4개의 Consumer 스레드를 미리 시작
for (int i = 0; i < threadCount; i++) {
    executor.submit(() -> {
        while (true) {
            try {
                // queue에서 데이터를 하나 꺼낸다 (비어있으면 대기)
                List<ExcelVO> batch = queue.take();

                // 꺼낸 데이터가 빈 리스트이면 종료 신호로 판단하고 break
                if (batch.isEmpty()) {
                    break;
                }

                // 가져온 batch를 DB에 insert
                itemStockMasterService.insertUploadFile(batch);

            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    });
}

try (InputStream in = uploadFile.getInputStream()) {
    // 업로드된 엑셀 파일을 읽어서, batchSize 단위로 queue에 데이터를 넣는다
    ExcelSheetHandler.readExcel(in, queue, batchSize, messageSource);

    // 모든 Consumer에게 종료 신호를 보냄
    // (빈 리스트를 threadCount(=4)만큼 넣어준다)
    for (int i = 0; i < threadCount; i++) {
        queue.put(Collections.emptyList());
    }

    // ExecutorService에 더 이상 작업 제출을 하지 않겠다고 알림
    executor.shutdown();

    // 모든 Consumer 스레드가 종료될 때까지 최대 20분 동안 대기
    executor.awaitTermination(20, TimeUnit.MINUTES);

} catch (Exception e) {
    LOGGER.error(e.getMessage(), e);
    throw new BizException(e.getMessage());
}
```

- Consumer 스레드를 먼저 띄워서 미리 준비시킴. (Producer가 데이터를 넣자마자 바로 처리하게 하기 위함)
    - Consumer를 먼저 `executor.submit()`으로 등록해두면
    - 이들은 `queue.take()`를 호출한 채 **대기 상태**에 들어가 있음 (데이터가 올 때까지)
    - 이후에 Producer가 데이터를 `queue.put()` 하면, 대기 중이던 Consumer가 즉시 깨어나서 ****데이터를 처리할 수 있음
- **BlockingQueue**는 락(lock) 기반으로 내부를 보호하므로 (=thread-safe), 여러 Consumer가 동시에 take()해도 중복 없이 서로 다른 데이터를 안전하게 가져갈 수 있음.

  (그렇지 않으면, 별도의 락이나 세마포어를 수동으로 관리해야 함)




나중에 공부해볼 것)

Concurrent Collection (`SynchronousQueue`)

blocking