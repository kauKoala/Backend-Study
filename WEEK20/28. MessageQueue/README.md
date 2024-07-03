# MessageQueue
## 특징

- 분산 시스템 환경에서 비동기적 통신을 가능하게 함
- 메시지(요청)을 일정 시간 동안 저장하고 처리
- pub/sub 구조로 여러 사용자가 동시에 메시지를 보내고, 처리할 수 있음

## 대표적인 기술들

- rabbitMQ
    - 중앙 집중형으로 수직 확장은 쉽지만 수평 확장은 어렵다
    - 전통 큐 방식으로 안정성과 scale up 이 쉬운 장점이 있지만, scale out이 어렵다는 평가가 있다.
- Kafka
    - 이제 대용량 서비스로 떠나버린 그..
    - 높은 처리량을 보장한다.
- Amazon SQS
    - NACK를 보내 메시지 처리를 재시도 할 수 있다.
    - 여기에 관해서도 문제가 있지만 지엽적이라 여기서 다루진 않음
## RabbitMQ? Kafka?

찾아본 결과 rabbitMQ의 대부분 기능을 kafka로 처리할 수 있다.

그래서 무지성 Kafka를 써도 무관하다곤 하지만, 서비스 목적을 고려한다면 소규모에 Scale Up이 간단한 RabbitMQ가 더 유리한 상황도 있다.

요약하면 HashMap(=RabbitMQ) 왜 씀? Redis 쓰면 되는데(Kafka) 라고 보면 된다.

## 어느 이점이 있나?

- 확장성 증가: 구성 요소가 분리되고, 세부 사항을 알 필요 없이 독립적으로 작동할 수 있게 함.
  즉, 전체 요소에 영향을 주지 않고 독립적으로 작동할 수 있게 해준다.
- 내결함성: 메시지 큐가 죽지 않는 한, 요청된 메시지는 큐에 남아있기 때문에 일시적인 중단을 겪더라도 손실의 위험이 적어진다.
- 로드밸런싱: worker는 가능한 만큼 메시지를 소비하기 때문에 균등하게 부하를 줄 수 있다.
  즉, 하나의 인스턴스에 트래픽이 집중되는 현상을 예방한다.
- 이기종 시스템 간 통합: 서로 다른 기술로 구축된 시스템이라도 큐와 통신할 수 있는 인터페이스가 있다면 정보를 쉽게 교환할 수 있다.
- 우선순위 제공: 이벤트 순서를 지키거나 중요한 메시지를 우선 처리할 수 있게 한다.

## Quorum 큐

기존 방식의 큐 문제를 해결

큐를 미러링하고, 합의 알고리즘을 거쳐 consume할 메시지가 유효한지 확인
메시지가 모든 큐에 복제되는 단점이 있지만, 데이터 안정성과 신뢰도 ⬆ 

ex) 주문 및 선거 시스템같이 데이터 **무결성이 중요한 경우**에 사용한다.

## Stream

RabbitMQ에서 도입된 기술, 요약하면 메시지 상태를 보존한다.

기존 큐는 한번 소비(consume) 되면 다시 읽을 수 없었던 문제를 해결 메시지를 디스크에 저장해 큐가 종료되는 상황이라도 복구 가능

### ex) 여러 구독자에게 동일한 메시지를 전달할 때

1. 기존 방식은 전용 큐를 만들어야 한다.
2. 사용자가 많고, 메시지 종류가 다양할수록 관리 큐의 관리 비용은 증가

![Untitled](https://github.com/kauKoala/Backend-Study/assets/7845568/5471163d-c4e0-4a2d-89cc-88f0ac8b436c)

![Untitled (1)](https://github.com/kauKoala/Backend-Study/assets/7845568/84767777-cce3-4dd6-b0aa-182f208bc1e8)

## Reference

https://www.rabbitmq.com/docs/quorum-queues

https://www.rabbitmq.com/docs/streams

https://www.linkedin.com/pulse/why-do-we-need-message-queue-ambar-gupta/

https://channel.io/ko/blog/tech-backend-aws-sqs-introduction