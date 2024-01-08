# 7. HTTP

## HTTP란?
- 월드 와이드 웹에서 데이터를 주고받기 위한 프로토콜
- 주로 HTML 문서, 이미지, 비디오 등의 리소스를 전송하는 데 사용
- 응용 계층에서 사용되는 프로토콜
- TCP/IP 기반

## HTTP 주요특징
- 비연결성
    - 클라이언트가 서버에 요청을 보내고 서버가 응답을 보낸 후 연결을 끊음
    - HTTP/1.1에서 Persistence Connection이 도입되어 연결을 유지할 수 있게됨
- 무상태성
    - 서버는 이전에 이루어진 요청에 대한 정보를 기억하지 않음
    - 상태 정보가 필요한 경우, 쿠키나 세션과 같은 기술을 사용하여 상태를 관리
- 요청과 응답으로 구성
    - 클라이언트는 URL을 통해 특정 리소스를 요청하는데 상태 코드를 사용하여 요청의 종류를 정함
    - 서버는 요청을 처리하고 상태코드와 함께 요청된 리소스 혹은 에러메시지 클라이언트에게 전송
    - HTTP 상태코드와 메소드는 따로 주제로 있기에 다루지 않음
## HTTP 1.0 과 1.1 버전의 차이

HTTP 1.0이 도입됐을때와 달리 웹 규모가 커지면서 HTTP가 여러 문제들을 가지게 되었다.  
이에 대한 개선방법들이 이후 버전에 도입되었다.

### Persistence Connection

#### 기존의 문제점
- HTTP 1.0에서는 요청이 발생할떄마다 TCP의 3way handshake가 매번 발생했음
- 잦은 3way handshake는 네트워크 혼잡, 높은 latency를 유발하였다.

#### Persistence Connection vs Parallel Connection
클라이언트는 연속적으로 서버에 요청을 보낼 확률이 매우 높다.  
ex) 한 웹 페이지를 브라우저에서 로드할때 서버에 연속적으로 request를 보내는 것
이를 **site locality**라고 한다.

Parallel Connection = 병렬적으로 동시에 여러 TCP Connection을 맺는것 (동시에 TCP Connection을 시작함으로써 지연시간 줄이기)
Persistence Connection = 하나의 TCP Connection을 통해 여러개의 HTTP 요청과 응답을 보내는것

Persistence Connection의 유리한점
- Parallel Connection은 결국 많은 TCP Connection 비용을 유발한다.
    - 대역폭 소모 
    - 너무 많은 Conncection 유발

Persistence Connection의 문제
- 너무 많은 idle connection으로 인해 서버 부담 증가
- 따라서 현재 많은 Web Application들이 적은 수의 Parallel Connection을 Persistence하게 유지하고 있음

#### Keep-Alive 옵션 도입
- HTTP/1.0+ 에서는 하나의 TCP Connection에 여러 HTTP 요청과 응답을 보낼 수 있는 Persistence Connection을 Keep-Alive 옵션을 통해 해결했다.

- KEEP-ALIVE 옵션을 사용하기 위해서는 요청 Connection HTTP 헤더에 Keep-Alive 속성을 보내면 서버에서 마찬가지로 Keep-Alive를 Connection 응답헤더에 보내야 Connection을 재사용했다.

- 만약 서버측 응답에서 Keep-Alive가 없었다면 서버측에서 지원하지않는 것으로 판단하고 Connection을 재사용하지 않았다.

- Keep-Alive헤더는 timeout (최대 idle 유지 시간), max(최대 request개수)로 구성되었다.

- 또한 Keep-Alive에 해당하는 헤더값을 모든 요청에 보내야지만 Connection을 재사용했으며 정확한 Content-Length를 사용해야 특정 요청의 종료를 판단할 수 있었다.
#### KEEP ALIVE의 문제
- 멍청한 프록시 문제
    1. 웹 클라이언트는 프록시에 Connection : Keep-Alive 헤더와 함께 메시지를 전송한다.
    2. 클라이언트는 커넥션을 유지하자는 요청에 대한 응답을 확인하기 위해 기다린다.
    3. 멍청한 프록시는 요청받은 HTTP의 Connection 헤더를 이해하지 못한다.
    4. 프록시는 Keep-Alive를 모르기 때문에 다음 서버에 메시지를 그대로 전달한다.
    5. Connection 헤더는 홉별 헤더였다. (다음 hop에서 처리되야하는 헤더)
    6. 서버는 프록시로부터 Connection헤더를 받고 프록시와 Connection 유지   
    7. 프록시는 Keep-Alive를 몰라 서버와의 연결종료 대기 & 클라이언트는 응답 대기 
    8. Timeout까지 지루한 대기..

#### HTTP/1.1 개선안
- Proxy Connection 헤더
    - 비표준이고 또다른 문제를 발생
- HTTP/1.1 에서는 모든 Connection을 Persistence Connection으로 한다.
- 만약 클라이언트가 Persistence Connection을 끝내고 싶다면 Connection을 명시적으로 close해야한다.
    - HTTP Connection 헤더값을 close로 둔다.


### 파이프라이닝
위에서 설명했듯이 클라이언트는 연속적으로 여러 요청을 보내는 경향이있음 (웹페이지로드시)

하지만 HTTP/1.0에서는 클라이언트가 요청을 보내고 서버에서 응답을 받은후 다시 요청을 보내면서 latency(지연시간)이 발생하게됨

그래서 HTTP/1.1에서는 파이프라이닝을 도입하여 여러 요청을 한 TCP/IP 패킷으로 보내면서 latency를 줄였음

하지만 HOL 문제가 발생하게됨

#### HOL(Head of Line Blocking)
- 결국 서버는 한 TCP/IP패킷안에 있는 요청들을 순차적으로 처리해야한다.
- 이 과정에서 순서가 앞인 요청들이 처리속도가 오래걸리게된다면 응답을 보낼 수 없음
- 이 문제로 인하여 요청-응답과정의 전체 latency를 줄이는데에는 한계가 있음

### Cache-Control 헤더

같은 HTTP 요청을 매번 서버에게 보내지 않고 이에 대한 HTTP 응답을 재활용하여 서버의 부하, 네트워크에 대한 부하를 감소시키는 방법을 HTTP 캐싱이라함

HTTP 캐시는 크게 두종류로 사설캐시, 공유 캐시가 있으며 자세히는 브라우저 캐시, 프록시 캐시, 게이트웨이 캐시(리버스 프록시 캐시), CDN 등이 있는데 일단 이에 대한 내용은 다음 발표때,,,

HTTP 캐시에 저장되는 단위는 캐시 엔트리라 하고 캐시 엔트리는 캐시 키와 하나 이상의 HTTP 응답으로 구성되어있다.
- 캐시 키는 HTTP 메소드와 HTTP 요청으로 구성

HTTP/1.1 에서는 Cache-Control 헤더를 통해 HTTP 요청과 응답에서 HTTP 캐싱에 대한 옵션을 제어할 수 있다.

#### HTTP 요청에 해당하는 Cache-Control 디렉티브
- max-age
- max-stale
- min-fresh
- no-cache
- no-store


#### HTTP 응답에 해당하는 Cache-Control 디렉티브
- must-revalidate
- no-cache	
- no-store
- public	
- private	
- proxy-revalidate
- max-age
- s-max-age


**자세한 내용은 HTTP 캐싱 발표때...**

### Host 헤더

서버의 성능이 발달하면서 하나의 웹 서버에서 여러 웹 사이트나 도메인을 호스팅하는게 가성비에 맞게되었다.

이를 가상 호스팅이라고 하며 가상 호스팅에는 4가지 방식이 존재한다.

#### URL 경로를 통한 가상 호스팅
- http://kau.ac.kr/kau/index
- http://edu.ssafy.com/ssafy/index

위 사이트 http://kau.ac.kr/ 과 http://edu.ssafy.com/은 같은 서버에서 호스트(같은 IP, 같은 port)이므로 호스트 구분을 url (kau, ssafy)로 구분함


#### 포트번호를 통한 가상 호스팅
위와 같이 사이트 url을 바꾸는 대신에 다른 포트로 가상 호스트 구별
- 웹 서버 표준 포트가 80이므로 비권장

#### IP주소를 통한 가상 호스팅
가상 IP를 할당하여 가상 호스트를 구별
웹 서버가 가상 IP에 대한 리소스의 위치를 구분하여 응답

- 장비 IP개수가 제한적임 (대규모 호스팅시 문제 발생)
    - 리눅스 커널은 물리적인 서버 하나당 255개 IP가 가능


이와같은 문제에도 불구하고 현재도 사용


#### Host헤더를 통한 가상 호스팅
Host 헤더를 통해 가상 호스트 구별

Host 헤더 규칙  
- Host를 사용할 때 규칙Host 헤더에 포트가 쓰여있지 않다면 기본 포트를 사용
- URL에 IP 주소가 있으면 Host 헤더는 같은 주소를 포함해야 함
- URL에 호스트 명이 기술되어 있으면, Host 헤더는 같은 호스트 명을 포함해야 함
- URL에 호스트 명이 기술되어 있으면, Host 헤더는 URL의 호스트 명이 가리키는 IP주소를 포함해서는 안 됨.(여러개의 가상 사이트를 하나의 IP로 묶은 서버에서 문제가 생길 수 있다고 합니다.)
- 클라이언트가 특정 프록시 서버를 사용한다면, Host 헤더에 프록시 서버가 아닌 오리진 서버의 호스트 명과 포트를 기술해야 함.
- 웹 클라이언트는 모든 요청 메시지에 Host 헤더를 기술해야 함웹 프락시는 요청을 전달하기 전에 요청 메시지에 Host 헤더를 추가해야 함 
- HTTP/1.1 웹 서버는 Host 헤더 필드가 없는 HTTP/1.1 요청 메시지를 받으면 400 상태 코드로 응답해야 함


### Range Request

크기가 큰 파일이나 데이터를 주고받을때 HTTP 응답을 나눠서 오류, 일시정지, Pagination을 지원하는데 유용함

HTTP 요청에서 사용할때는 
```
Range: bytes=100-200
```
위와 같이 HTTP 헤더를 설정 (100바이트부터 200바이트까지 요청)

처음 서버 HEAD 요청에 대한 HTTP 응답으로
```
Accept-Ranges: bytes
```
위와 같은 HTTP 헤더가 설정되었다면 범위요청에서 바이트가 최소단위

```
Accept-Ranges: none
```
이라면 Range Request미지원


HTTP 요청에서 사용할때는 
```
Range: bytes=0-1023
```
위와 같이 HTTP 헤더를 설정 (100바이트부터 200바이트까지 요청)

HTTP 응답에서 사용할때는
```
Content-Range: bytes 0-1023/146515
Content-Length: 1024
```
위와 같이 HTTP 헤더에 Content-Range로 보내는범위/리소스 전체크기를 알려주고 Content-Length를 통해 요청한 범위의 크기를 알려줌

## 더알아보기

#### HTTP/2.0 

#### HTTP 특징을 이용한 공격

#### QUIC
