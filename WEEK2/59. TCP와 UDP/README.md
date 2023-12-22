면접에서 자주 물어보는 내용 위주로 작성

### 1. TCP와 UDP란?

- TCP: 서버와 클라이언트 간에 데이터 순서가 보장되고, 신뢰적인 데이터 전송을 위한 프로토콜
- UDP: 서버와 클라이언트 간에 빠른 데이터 전송을 위한 프로토콜 (비신뢰적, 데이터 순서 보장 X)
- 공통점: OSI 7계층중 Transport Layer에 속해 일을 할 포트번호인 프로세스에게 전달함

### 2. TCP 헤더 구조

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/54a1caf1-37c9-40b9-a9f4-3ae6e21e928d)


- 포트 번호
    - Source port: 어느 포트 번호에서 시작했는지
    - Destination port: 도착해야할 포트 번호는 어디인지
    - 사용 이유: 컴퓨터는 멀티 프로세스이기 때문에 포트 번호를 사용하여 프로세스까지 데이터가 전달될 수 있도록 해줌
    - 특징: Destination port를 미리 알고 있는 상태로 데이터를 전송 (ex. http 통신시 www.naver.com:80)
- Sequence number
    - 데이터 순서를 구분짓는 번호로 세그먼트 크기만큼 늘어남
- Acknowledgement number
    - Sequence Number를 보내면 다시 데이터를 받을 때 받길 원하는 Sequence number 값도 같이 보냄
- Window Size
    - 데이터를 보낼 수 있는 양
    - 흐름 제어에서 동적으로 설정됨
- Checksum
    - 데이터 통신중에 발생할 수 있는 세그먼트 오류를 확인함

### 3. UDP 헤더 구조

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e342a760-b22d-4578-a97f-3793ed0b39e9)

- 신뢰적이지 않고, 순서보장을 하지 않아 단순한 구조로 되어 있음
- 주로 게임, 동영상 스트리밍에서 사용됨
- checksum
    - 세그먼트에 오류가 있는지 확인하고, 패킷을 버릴지말지 결정함

### 4. 3-way handshake와 4-way handshake

- 3-way handshake

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/2eeed147-edc5-4f8c-9331-80d6c5377a13)

- 4-way handshake

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/84f413fc-a828-4397-a889-6a63b94cd07d)

### 5. 흐름제어와 혼잡제어

TCP 특징인 데이터 순서가 보장되고, 신뢰적인 데이터를 보낼 수 있는 방식으로 사용

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/0feb455c-e747-4ef2-b745-fef18f6052e2)

흐름제어

- 방법
    - Stop and Wait
        - 데이터를 전송한 뒤, ACK 응답을 받으면서 데이터를 전송
    - Sliding Window
        - 처음에 윈도우에 포함하는 모든 패킷을 전송하고, 윈도우 크기를 줄이다가 ACK을 받으면 윈도우를 크기를 다시 늘림
        - 첫 윈도우 크기는 3-way handshake를 할 때 결정

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/76b46883-6bc9-401e-a584-5eeea632fa03)

혼잡 제어

- Slow Start
    - 패킷이 문제 없이 도착하면 window size를 2배씩 늘려준다
    - 만약 패킷 전송에 문제가 발생하면 다시 window size를 1로 시작한다

- AIMD (Additive Increase / Multiplicative Decrease)
    - 패킷이 문제 없이 도착하면 window size를 1씩 늘려준다 (Addictive Increase)
    - 만약 패킷 전송에 문제가 발생하면 window size를 반절로 줄인다 (Multiplicative Decrease)

- Fast Retransmit
    - 데이터의 순서가 잘못됐을 경우, 문제가 되는 순서의 데이터를 재전송함
    - 중복된 번호의 데이터를 3번 받으면 window size를 줄임

- Fast Recovery
    - 데이터 전송에 문제가 생기면 window size를 1로 줄이지 않고, 반절로 줄여서 다시 Addictive Increase를 할 수 있도록 함

- 흐름제어와 혼잡제어의 차이점
    - 흐름제어: 송신측과 수신측의 데이터 처리 속도 차이를 제어하기 위한 방법
    - 혼잡제어: 송신측과 네트워크 데이터 처리 속도 차이를 제어하기 위한 방법
