## 0. UNIX 철학

[Everything is a file]

모든 것은 파일로 이루어져 있음

소켓이든 프로그램이든 파이프라인이든 Unix는 모든게 파일로 이루어져 있다는 뜻

---

</br>
</br>
</br>

## 1. 소켓이란?

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/a61a4c18-9e70-41ee-b2d3-550569c6e89e)

- 소켓은 운영 체제가 제공하는 인터페이스로, 애플리케이션 계층의 프로그램이 네트워크 서비스를 사용할 수 있게 하는 역할을 함
- 네트워크 통신에서 사용하는 엔드포인트로 TCP/IP 4계층으로 이야기하면 Application 계층과 Transport 계층 사이에 존재함

</br>

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/c10f98a9-ae00-46ea-8e83-49d1d5cfa810)

소켓 번호는 응용 프로그램마다 고유한 번호를 가지고 있는데, 파일 디스크럽터 번호를 사용함

[파일 디스크럽터 (fd)]

열린 파일이나 I/O 자원을 식별하는 값

- 파일이 열리면 고유한 파일 디스크럽터 값을 할당하고, 해당 값을 통해 파일 읽기/쓰기/닫기가 가능함
- 파일 디스크럽터 번호 0, 1, 2는 값이 정해져있음
    - 0: 표준 입력 (stdin)
    - 1: 표준 출력 (stdout)
    - 2: 표준 에러 (stderr)

[소켓 번호]

소켓도 파일이므로 파일 디스크럽터 번호를 공유하여 사용함 (3번부터 사용)

- 응용 프로그램마다 고유한 번호를 가지고 있음
- 그러므로 서로 다른 응용 프로그램이라면 FD 번호가 겹칠 수 있음

### [실습 1]

- 파일에 할당된 번호는 `lsof`를 입력하면 소켓과 파일 디스크럽터를 볼 수 있음

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/862c6376-9a34-4df9-b13d-286cb37146c0)

0r, 1u, 2u는 모든 파일에 있다는 것을 확인할 수 있음

- `숫자r` : 읽기 모드
- `숫자w` : 쓰기 모드
- `숫자u` : 읽기, 쓰기 모드

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/572e1475-61fd-430c-8bcd-eea3dd48d707)

이때 소켓은 IPv4, IPv6, unix, sock 과 같은 것으로 표기되어 있는걸로 구분할 수 있음

### [실습 2]

- 실시간으로 fd가 무슨 일을 하는지 보려면 터미널에 `sudo fs_usage`를 입력하기 (종료는 Ctrl + C)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/78992e6f-e6a3-4910-b897-5b16e2c3774f)

---

</br>
</br>
</br>

## 2. 소켓 통신 과정

소켓의 종류는 사용하는 전송 계층의 프로토콜에 따라 달라짐

각 소켓 종류마다 소켓 연결해서 데이터를 통신하는 방법이 다름

- [TCP] Stream Socket
- [UDP] Datagram Socket

소켓 번호와 (IP 주소, 포트번호)를 연결하기 위해서 아래의 정보가 필요함

1. 소켓 번호
2. 자신의 IP 주소
3. 자신의 포트번호

네트워크 통신에서 소켓을 통해 데이터를 보내기 위해서는 아래의 정보가 필요함

1. 전송 계층 프로토콜 종류
2. 자신의 IP 주소
3. 자신의 포트번호
4. 상대방의 IP 주소
5. 상대방의 포트번호

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/a76bbab9-a152-46c4-8a83-d92988695446)

위 사진은 TCP 소켓 프로그래밍 동작 방식임

- socket: 소켓 생성
- bind: 서버는 통신할 포트번호, IP주소가 필요하므로 소켓과 바인딩 과정이 필요함
- listen: 연결을 대기하는 상황
- connect: 클라이언트가 서버에 연결 요청을 하며 3-way handshake 수행
- accept: 연결 요청을 수락하여 `새로운 소켓`을 만들어서 연결함
(기존 소켓은 다시 새로운 클라이언트 요청을 기다림)
- recv(), send(): `새로운 소켓`에서 데이터를 받거나 보냄

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/3d591878-bf82-4f16-9807-18f202ca7da0)

TCP는 accept가 끝나면, 새로운 소켓을 만들어서 일대일 통신을 진행하고, recv / send 작업을 수행함

이때 새로운 소켓번호에 채우는 정보는 (상대방의 IP주소, 포트번호)를 사용함

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/49b5bcf2-6ea1-423d-9475-3bf3253427e2)

</br>

여기서 UDP는 비연결지향 프로토콜이기 때문에 listen - connect - accept 과정이 필요 없음

UDP는 새로운 소켓을 생성하지않고 하나의 소켓에서 다 처리함

---

</br>
</br>
</br>

## 3. 이외 다른 내용

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/0caf90eb-7853-4fc6-9344-88c7d46424ce)

</br>

### [1] 채팅방 만들기

여러 명의 사람이 들어오는 채팅방은 multiplexing 서버 방식을 사용함

multiplexing: 하나의 스레드로 다수의 통신 요청을 처리함

fd_set을 통해 파일 디스크립터들을 하나의 자료구조로 관리할 수 있음

따라서 하나의 채팅방에 N+1개의 소켓이 필요함

select 함수는 소켓 상태가 변경되면 이벤트가 발생함

- [처음보는 소켓이면?] accept를 통해 fd_set에 소켓 번호 추가
- [기존 소켓이면?] 채팅을 보낸 것이므로 모든 소켓에게 채팅 메시지 발송

참고로.. redis가 성능이 좋은 이유도 단일 스레드에서 클라이언트 요청을 처리하는데, multiplexing을 사용함

(완전한 싱글 스레드는 아니고 내부 처리는 비동기로 처리해서 여러 개의 스레드를 사용하긴 함)

### [2] 포트번호 개수만큼 소켓을 만들 수 있다?

지금까지 내용을 살펴보면 TCP로 인해 포트번호만큼 소켓을 만들게 되면 모든 포트번호를 활용하지 못하는 문제가 발생함. 그래서 소켓은 컴퓨터가 만들 수 있을 때까지 계속 만들어 낼 수 있음

[궁금해서 찾아본 점]

소켓도 이론상 무제한으로 만들어낼 수 있고, 이전에 Actor Model도 이론상 무제한으로 만들어낼 수 있는데, 스레드는 어떤지 살펴봄

이론적으로는 스레드도 메모리가 버틸 수 있는만큼 스레드를 생성할 수 있지만, 스레드에 할당되는 스택 메모리가 최소 1MB이기 때문에 많이 만들 수는 없음 

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/fa3a7300-5a14-45ce-9baa-601bda4646b3)

</br>

터미널에 `sysctl kern.num_taskthreads`를 입력하면 각 태스크당 최대 몇 개 만들 수 있는지 확인이 가능함

코드로 확인해보니 프로젝트에 따라 최대 스레드 개수는 달라지는데, Thread 객체는 메모리 + JVM에서 사용하는 스레드에 따라 달라지는 것으로 보임

[종설 프로젝트]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/1af51059-56eb-4a6e-ab31-0ef948946d49)

</br>

[우테코 프로젝트]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/2512a513-81b6-4065-a3f5-4729e4ea4b07)

</br>

---

</br>
</br>
</br>

## 4. 참고하면 좋은 사이트

[이거 하나로 공부 가능]

http://jkkang.net/unix/netprg/chap2/net2_intro.html

[논블로킹]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/ca67ec37-a722-4dd5-b866-63be7fbf7653)

</br>

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/28a1e916-6c65-44f0-8d94-7e29ed110604)

</br>

`sudo fs_usage`를 입력하면 나오는 [ 35]는 EAGAIN (= EWOULDBLOCK)가 패킷을 온전하게 받기 전까지는 에러 메시지를 보냄

---
