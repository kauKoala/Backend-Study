# 120. HTTP2

발표자: 이상재
발표 날짜: 2024년 1월 13일

# HTTP2

## HTTP2 등장 배경

기존의 HTTP/1.1에서 지적 당해 오던 여러가지 문제를 해결하고자 개선된 버전을 만들게 됨

### HOL 문제

HTTP 1.1에서 발생하는 요청-응답이 항상 순차적으로 처리되어야 하는 문제

Pipelining을 통해서 요청을 먼저 보내는 것으로 해결하려 했으나 응답은 순서대로 처리해야했음

### 헤더 중복

HTTP 1.1에서는 요청-응답에서 해당하는 헤더를 모두 가지고 있어야 했습니다.

### 느린 웹페이지 로딩 속도

HTTP 1.1에서 Pipelining과 Persistence Connection을 통해 어느 정도 웹 리소스에 대한 로딩 지연을 줄였지만 커져 가는 웹 페이지 (SPA, SSR)와 더불어 여전히 크게 해소되지 못함

## 기본 용어들

- 메시지 : HTTP1에서는 하나의 요청 메시지, 하나의 응답 메시지였다면, HTTP2에서는 헤더와 데이터 영역으로 나누고, 각 영역도 여러 개의 프레임(일정한 크기)으로 하나의 메시지가 전달된다.
- 프레임 : HTTP2에서 통신의 최소 단위. 각 프레임마다 하나의 프레임 헤더를 포함하며, 이 프레임 헤더는 프레임이 속하는 스트림을 식별한다.
- 스트림 : 구성된 연결 내에서 전달되는 바이트의 양방향 흐름. 하나 이상의 메시지가 전달될 수 있다.(단위로는 하나의 요청과 하나의 응답)

## HTTP1.1에서 개선된 점

### Binary Protocol

왜 기존 텍스트 기반 프로토콜은 HTTP2에서 버림받았는가

![binary framing](https://github.com/sj7699/Backend-Study/assets/26706925/5dcdad2b-cf78-4f9d-bab7-c54220a6156d)


- 효율적이어서
    - 기존에는 줄바꿈으로 구분되던 아스키 코드 기반 텍스트
        - 텍스트는 사람이 이해하기 쉬움
        - 하지만 기계에게는..?
    - Binary 데이터는 효율적이고 빠른 처리가 가능하다
        - Binary 데이터는 기본적으로 동일한 정보 표현에 있어 텍스트보다 사이즈가 작음
            - 아스키 코드나 유니 코드의 사이즈!
        - 문자열 처리 오버헤드
            - 인코딩 처리 + 문자에 대한 오류 제어
- 멀티플렉싱을 위해서
    - 멀티 플렉싱에서는 병렬적으로 여러 요청과 응답이 발생한다.
    - 사이즈가 작은 binary 데이터는 네트워크 대역폭을 효율적으로 사용
- 보안 문제
    - Response Splitting Attack
    

### Multiplexing

![multiplex](https://github.com/sj7699/Backend-Study/assets/26706925/a467a5c5-2bce-4b54-842c-a531b33695ee)


Multiplexing은 Latency를 줄이고자 나온 개념이다.

단일 연결을 적극적으로 활용하고자 하기 위해 만들어진 방법

기본적으로 클라이언트가 여러 요청을 보내려면 여러 TCP Connection이 필요했음

또한 한 TCP Connection을 재활용하더라도 응답 처리 순서는 항상 순차적으로 진행했음

- 이를 개선할 수 없을까?


![http1 1 1 2](https://github.com/sj7699/Backend-Study/assets/26706925/00f1db8a-7559-42a3-90ba-fc79dae1950c)

메시지와 스트림을 사용하여서 동시에 여러개의 메시지 스트림을 응답 순서에 상관없이 주고 받을 수 있게 되었다.



### Server Push

![server push](https://github.com/sj7699/Backend-Study/assets/26706925/8b960039-83e1-4690-ab79-0d61bdba4b93)


- 비대해진 웹사이트에서 많은 자원들을 loading 해야함
    - css, js, 이미지 등등,,,
- 이때 잦은 latency가 발생
    - 이걸 해결할 수 없을까?
- 서버에서 미리 필요한(요청이 예상되는) 자원을 클라이언트에게 보내줌
    - 이를 통해 latency를 줄이고자함
    
- 하지만 이제 Chrome 브라우저에서 보지 못함
    - [https://developer.chrome.com/blog/removing-push?hl=ko](https://developer.chrome.com/blog/removing-push?hl=ko)
        - 요약하면 사이트에서 Server Push를 활용하는 사례가 매우 적음
            - 오히려 성능 저하가 발생했음
        - HTTP 3.0에서 구현되지 않음
        - 대안 기술이 생김

### 헤더 압축

![header compression](https://github.com/sj7699/Backend-Study/assets/26706925/7caf3873-669b-4a82-af92-22b6b94ecd83)


HTTP2에서는 이전 frame에서 전송된 헤더들을 다시 전송하지 않음

- 가능해진 이유
    - 헤더를 더 작은 단위인 frame으로 쪼갰다.
    - binary 데이터이기 때문에 기존의 key-value기반 텍스트보다 구조에서 자유롭다.

중복된 헤더를 체크하기 위해 클라이언트와 서버 측에서 헤더를 캐싱함

캐싱하는 테이블은 크게 두가지로 나뉘어짐

- 정적 테이블
    - 일반적으로 HTTP에 자주 사용되는 HTTP 헤더를 미리 정의
    - 헤더 필드명과 값이 미리 정해져 있음
        - 통신 시 인덱스만 전송하여서 사이즈를 줄임
- 동적 테이블
    - 같은 TCP Connection 중에 발생하는 헤더들을 저장
    - 같은 헤더를 사용해야 할 경우 정적 테이블과 마찬가지로 인덱스만 전송

중복 헤더 제거 뿐만 아니라 허프만 코딩으로 압축

중복 헤더 제거 + 압축으로 인한 사이즈 축소 ⇒ 헤더에서 발생하는 네트워크, 처리 부하 개선

이 과정들을 HPACK 이라고도 함 

# 더 알아보기

### 스트림 우선순위

### 흐름 제어

### HTTP 3.0

### QUIC

# Reference

[https://velog.io/@pds0309/http2](https://velog.io/@pds0309/http2)

[https://web.dev/articles/performance-http2?hl=ko#why-not-http12](https://web.dev/articles/performance-http2?hl=ko#why-not-http12)

[https://www.youtube.com/watch?v=uhlvXrDpM-Y](https://www.youtube.com/watch?v=uhlvXrDpM-Y)
