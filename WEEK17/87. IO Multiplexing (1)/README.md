동기/비동기, 블로킹/논블로킹는 이미 다룬 내용이라 생략함

</br>

## 1. 우리가 아는 내용 (multiplexing)

### [1] multiplexing, demultiplexing

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/3bf6abab-4249-47f3-8623-bfda8eb10f0e)

사실 네트워크 하향식 접근 책에 이런 내용이 있었다.

TCP / UDP에서의 multiplexing, demultiplexing

- multiplexing: 여러 클라이언트로부터 동시에 들어오는 데이터를 처리함
- demultiplexing: 수신된 패킷을 IP와 포트번호에 바인딩된 소켓에 전달

### [2] HTTP 2.0 multiplexing

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e74e78ff-3e36-4ce7-a095-5acea4c0ad4e)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e5265ce1-e5a6-4631-930c-fca8744730c7)

하나의 소켓에서 여러 개의 스트림 번호를 관리하여 Stream ID을 관리하여 여러개의 클라이언트의 요청/응답을 관리 가능해짐. 이로인해 응답 순서에 상관없이 패킷을 주고 받을 수 있어짐

### [3] Socket - select() multiplexing

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/a8262ef4-d48c-46d1-b330-9aa945a1f88d)

Socket 발표에서 잠깐 select()에 대해 언급했는데, select()는 fd_set이라는 여러개의 파일 디스크립트를 관리했음. 이를 통해 하나의 스레드가 여러 개의 소켓을 관리함

### [4] multiplexing이란?

`“everything is a file”` 철학을 가진 unix 관점으로 보면 `“하나의 스레드가 여러 파일을 관리하는 기술”`이라고 볼 수 있음

- 스트림도 수신 버퍼에 저장하는 일종의 파일이므로
- 소켓도 결국 파일이므로

---

## 2. 용어 정리

### [1] IPC (운영체제)

프로세스끼리 데이터를 주고 받을 수 있는 기술

- 공유 메모리
- 파이프 라인: 1개만 사용하면 단방향 (데이터 전송이 연속적으로 바이트 단위로 보냄, 주로 동기적으로 작동)
- 소켓: IP번호와 포트번호를 할당해 원격으로 공유함
- 메시지 큐: 메시지 구조체 단위로 메모리 공간에 구현된 메시지 큐에. 전달 (비동기, 영속적)

```cpp
struct message {
	long msg_type;        // 메시지 타입
	char msg_text[100];   // 메시지 데이터
};
```

위 사진은 메시지 큐 구조체 예시

### [2] signal

IPC 종류 중에 하나로 다른 프로세스에게 이벤트를 전송하는 기술 (말 그대로 신호라고 생각하면 좋음)

MacOS 기준 터미널에 kill -l을 입력하면 다양한 signal 명령어를 제공해줌

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/5d5c2858-1eb9-4282-8f93-44b33e96623e)

- `kill -9 PID` : 프로세스 강제 종료 (KILL)
- `kill -2 PID` : 프로세스 정상 종료 (INT)
- `kill -USR2 PID` : 프로세스에게 커스텀 신호를 보냄 (USR1, USR2)

---

## 3. fd_set과 select

가장 원시적인 방법

### [1] fd_set

파일 디스크럽터(fd)들을 저장하는 배열 자료구조, 최대 1024개 기록 가능하다

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/09024953-49a7-4a72-b7ab-3cf1c573b7fe)

비트 방식으로 파일 변경 감지를 기록하기 때문에 1, 0으로 기록한다

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/710ee66a-c064-4f54-a103-ff2149723952)

위 사진은 fd_set을 선언한 변수 이름이 readfds임

1. FD_ZERO: 모든 파일 디스크럽터를 0으로 초기화
2. FD_SET: 2, 4, 8 FD가 데이터 변경 감지
3. 4번 파일에 데이터가 있으면 1로 바뀐다

### [2] select

fd_set을 통해 입출력을 관리하는 기술

select는 fd_set의 크기만큼 완전 탐색 방식으로 동작해서 성능이 느리다 O(N)

```cpp
int select (int nfds,
						fd_set *restrict readfds,
						fd_set *restrict writefds,
						fd_set *restrict exceptfds,
						struct timeval *restrict timeout);
```

- `nfds` : 관리하는 파일의 개수 (파일 디스크럽터 개수 + 1)
- `readfds` : 읽을 데이터가 있는지 검사하기 위한 파일 목록
- `writefds` : 쓰여진 데이터가 있는지 검사하기 위한 파일 목록
- `exceptfds` : 예외 사항이 발생한 파일이 있는지 검사하기 위한 파일 목록
- `timeout` : select 함수를 호출하면 데이터 변경을 timeout 시간 동안 기다린다 (blocking)
- `return : int` : 데이터가 변경된 파일의 개수를 반환한다

[select의 단점]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/db491e19-0d25-45e5-a21c-3b961df06c44)

select를 호출하면 다음과 같이 동작한다.

1. select를 호출한다. 이때 `readfds`만 사용한다고 가정한다.
2. `readfds` 에 연결된 소켓을 배열 크기만큼 완전탐색으로 확인한다.
3. 만약 읽을 데이터가 있으면 해당 소켓의 파일 디스크럽터 값을 1, 없으면 0으로 설정한다.
4. 반환값으로 읽기 가능한 파일 디스크럽트의 수를 반환해준다.

여기에서의 문제점은 3번이다. select를 호출할 때마다 이전 값이 모두 날아가기 때문에 따로 저장해두어야한다.

### [3] 정리

장점

- 가장 단순한 형태라 쉽게 이해하고 사용할 수 있음
- 지원하는 OS가 많아서 이식성이 좋음

단점

- 완전 탐색이라 성능이 좋지 않음
- 이전 상태 값을 매번 복사해야함

---

## 3. poll

`select`는 최대 1024개의 FD만 검사할 수 있었으나, `poll`은 무한대의 FD를 검사할 수 있다.

```cpp
int poll (struct pollfd *fds,
					nfds_t nfds,
					int timeout);
```

- `pollfd` : 동적으로 확장 가능한 파일 디스크럽트를 검사하는 배열
- `nfds_t` : 파일 디스크럽터의 개수
- `timeout` : 지정된 timeout 시간동안 파일 }디스크럽터의 상태가 바뀔 때까지 기다림 (blocking)

```cpp
struct pollfd {
  int fd; /* file descriptor */
  short events; /* requested events */
  short revents; /* returned events */
};
```

- `fd` : 감시할 파일 디스크럽터
- `events` : 감시할 이벤트 (읽기 - POLLIN, 쓰기 - POLLOUT,  긴급 - POLLPRI, …)
- `revents` : events에서 요청한 이벤트에서 실제로 발생한 이벤트

[동작 방식]

여전히 완전탐색으로 O(N)대신 select랑 살짝 다르다.

- `select`: 배열 크기만큼 완전 탐색을 수행
- `poll` : 파일 디스크럽터 개수만큼 완전 탐색을 수행

[poll이 파일 디스크럽터 개수만큼 완전 탐색을 수행하는 이유]

select의 fd_set 구조체는 고정된 크기를 가짐 (기본값은 1024임, [코드 링크](https://codebrowser.dev/glibc/glibc/misc/sys/select.h.html))

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/857ae72e-f6c4-4c12-b4e4-0c4cdbbd9faf)

하지만 poll은 아래 코드와 같이 배열로 선언이 가능함

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/217a1cb4-d08e-4097-86ba-0431c9fc674f)

이 과정에서 malloc, realloc을 통해 시작부터 메모리 할당을 동적으로 변경하거나, realloc을 통해 가변적으로 변할 수 있음

---

## 4. 더 찾아보면 좋은 내용

select, poll에서 개선된 방식도 있음

- pselect (select의 개선 버전)
- ppoll (poll의 개선 버전)

아예 select, poll보다 더 발전된 방식도 있음

- epoll (linux)
- kqueue (freebsd)
- IOCP(windows)

더 공부하고 싶은 분은 아래 링크를 통해 공부하는걸 추천함

- [네이버 클라우드 기술 블로그 - IO Multiplexing 기본 개념부터 심화까지 1부](https://blog.naver.com/n_cloudplatform/222189669084)
- [네이버 클라우드 기술 블로그 - IO Multiplexing 기본 개념부터 심화까지 2부](https://blog.naver.com/n_cloudplatform/222189669084)

---

## 5. 출처

사실 epoll까지 다루고 싶었는데, 이벤트에 대해 아는게 많이 없어서 다음 시간에 발표할 예정임

성능이 유독 좋다? 찾아보면 보통 multiplexing과 event loop가 적용되어 있음

- [DZone - Getting Started with HTTP/2](https://dzone.com/articles/understanding-http2)
- [네이버 클라우드 기술 블로그 - IO Multiplexing 기본 개념부터 심화까지 1부](https://blog.naver.com/n_cloudplatform/222189669084)
- [JOINC - select를 이용한 입출력 다중화](https://www.joinc.co.kr/w/Site/system_programing/File/select)
- gpt - 4o

---
