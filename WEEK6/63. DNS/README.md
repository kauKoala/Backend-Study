### 1. DNS란?

Domain Name System으로 도메인을 IP 주소로 변환해주는 시스템

### 2. 반복적 DNS 질의

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/341fb2e9-9434-42ce-be3f-f0a61395e62d)

www.[google.com](http://google.com)을 입력할 경우 DNS에 질의하는 과정

중간에 캐시가 되어 있으면 캐시된 IP를 클라이언트에게 반환함

- 클라이언트와 가장 가까운 DNS 서버에 질의를 한다. (편의상 가장 가까운 DNS 서버를 A라고 명칭함)
- A에 정보가 없으면 A는 루트 도메인에 [www.google.com](http://www.google.com)이 있는지 조회 메시지를 보낸다.
루트 도메인은 정보가 없지만, com 도메인의 아래에 있으므로 com 도메인의 DNS 서버 IP 주소를 반환한다
- A는 com 도메인의 DNS 서버에 www.[google.com](http://google.com)의 IP 주소가 있는지 조회 메시지를 보낸다.
com 도메인은 정보가 없지만, google.com 도메인의 DNS 서버 IP 주소를 반환한다.
- A는 이것을 반복하여 결국 www 도메인의 DNS 서버에 www.google.com의 IP 주소가 있는지 조회 메시지를 보낸다.
www 도메인은 www.google.com의 IP 주소가 있기 때문에 해당 IP 주소를 반환하게 된다.
- A는 클라이언트에게 www.google.com의 IP 주소를 반환한다.

### 3. 재귀적 DNS 질의

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/eaf13b2f-a73d-4d71-9ac2-4ecdcb6d9c77)

www.[google.com](http://google.com)을 입력할 경우 DNS에 질의하는 과정

중간에 캐시가 되어 있으면 캐시된 IP를 클라이언트에게 반환함

일반적으로 캐시 기능을 사용에 용이한 재귀적 DNS 질의를 사용함

- 클라이언트와 가장 가까운 DNS 서버에 질의를 한다.
- 가장 가까운 DNS 서버에 정보가 없으면 가장 가까운 DNS 서버는 루트 도메인에 www.google.com이 있는지 조회 메시지를 보낸다.
- 루트 도메인은 com 도메인의 DNS 서버에 조회 메시지를 보내고,
com 도메인은 google.com의 DNS 서버에 조회 메시지를 보내고,
google.com 도메인은 [www.google.com](http://www.google.com) DNS 서버에 조회 메시지를 보낸다.
- 이후 IP 주소를 반환하면 [google.com](http://google.com) → com → 루트 도메인 → 가장 가까운 DNS 서버 순서대로 반환받게 된다.
- 이후 가장 가까운 DNS 서버는 클라이언트에게 IP 주소를 반환해준다.

### 4. 특징

- 기본적으로 UDP 통신을 한다 (TCP의 경우 서로 연결을 해야하므로 비연결지향 프로토콜을 활용)
- TCP 통신을 하는 상황
    - 응답 메시지가 512 바이트를 넘어갈 때
    - 2개 이상의 DNS 서버를 사용할 경우, Master에서 Slave으로 Zone 정보를 보낼 때 (= 동기화 과정)
    - 그 외 여러가지가 있는데 찾아보시길..
