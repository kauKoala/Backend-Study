## 1. HTTP와 HTTPS

패킷의 암호화 여부로 HTTPS는 암호화되어 패킷을 알아볼 수 없음

HTTP는 80번 포트로 통신이 진행되고, HTTPS는 443 포트번호로 모든 통신을 진행함

- HTTP 패킷

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/ed251875-53c0-47d9-8cf1-9963ae7040bb)

- HTTPS 패킷

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/7624db47-b886-41d3-867e-6282a9e331db)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/97a30c99-8f81-443c-b04b-ac7f6bfea3e7)

- SSL 닫힘 통지는 서로 `close_notify`를 보내어 종료함. 이후 TCP 4-way handshake가 이루어짐

---

## 2. SSL, TLS 관련 이야기

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/3178cfde-ee41-4f98-b0d6-9ce9858b6240)

[로마시대 암호학]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/041ad6f4-5b7f-437e-bd7f-a8f84851b3f1)

[세계 2차 대전 암호학 - 에니그마]

풀네임

- SSL (Secure Sockets Layer)
- TLS (Transport Layer Security)

SSL 역사

- 1994년: Netscape 회사에서 웹 브라우저 보안 프로토콜인 SSL 1.0을 제작
(너무 이론 중심 구현이라 출시 X)
- 1995년: SSL 2.0을 만들었으나 마찬가지로 보안 문제가 발견되어 빠르게 3.0으로 넘어감
- 1996년: SSL 3.0에서 많은 사람들이 사용했으나 2015년에 심각한 보안 문제로 패킷을 해독할 수 있는 방법이 발견되어 TLS로 넘어감

TLS 역사

- 1999년: IETF 기관에서 SSL 3.0의 업그레이드 버전인 TLS 1.0을 제작함
보안과 안정성면에서 SSL 3.0보다 좋음
- 2000년 이후: TLS 1.1 (2006), TLS 1.2 (2008), TLS 1.3 (2018)로 계속해서 업데이트를 진행중

통계 자료 (2018년 상반기 기준 - [링크](https://webtribunal.net/blog/ssl-stats))

- 상위 100,000개의 사이트 79%가 HTTPS 통신을 사용
- 상위 100,000개의 사이트중에 6.8%가 SSL 2.0 / 3.0을 지원하는중
    - 반대로 말하면 93.2%가 TLS handshake만 지원하고 있음

---

## 3.  대칭키, 비대칭키

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e2d8a76d-fa16-4b2b-83ff-0dad7884feff)

대칭키

- 암호화와 복호화를 하나의 키로 사용
- 예시
    - AES 알고리즘 (가장 많이 사용함)
    - DES 알고리즘 (AES 이전에 사용하던 표준 암호 알고리즘)

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/00504878-df31-4ec9-a123-bc89204fd2d1)

비대칭키

- 암호화와 복호화를 서로 다른 키를 사용함
- 종류
    - 공개키
    - 비공개키
- 상황
    - 공개키로 암호화를 하면 → 비공개키로 복호화 가능 → 데이터 보안에 유리함
    - 비공개키로 암호화를 하면 → 공개키로 복호화 가능 → 공개키는 보여줘도 되므로 인증에 유리함 (TLS)
- 예시
    - RSA

SSL handshake에서는 대칭키와 비대칭키 둘 다 사용하지만, 서명 인증 과정은 비대칭키가 중점적으로 사용되고, 이후 통신에서는 대칭키를 사용해 통신함

---

## 4. SSL/TLS handshake 과정

---

SSL/TLS handshake는 3-way handshake 직후에 진행함

### 4-1) 구체적인 SSL/TLS Handshake 동작 과정

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e3d6057d-54ec-4e6b-88c6-a3aefae4119a)

- 아래 중요 내용은 모두 6번(이외에 궁금해서 찾아본 내용)에서 설명해뒀습니다.
- 중요 1) 그림에 나온 CA는 이미 생성된 대칭키와 비대칭키를 가지고 있습니다.
(그림의 3번 과정이 실제로는 없지만 설명을 쉽게하기 위해 추가된 것으로 보임)
- 중요 2) 여기서는 CA와 클라이언트가 직접적인 통신을 하는 것 같지만, 실제로는 클라이언트 내부에서 작동합니다.

[인증서 등록]

1. `서버` 서버에서 비대칭키를 생성 (Server_Public - 공개키, Server_Private - 비밀키)
2. `서버 → 인증서 기관` 사이트 정보와 Server_Public를 통해 SSL 인증서 생성 요청
3. `인증서 기관 → 서버` SSL 인증서를 CA_Private로 암호화 후 서버에게 전달

[클라이언트에서 서버로 요청을 보내기 시작할 때]

1. `클라이언트 → 서버` 클라이언트가 서버에게 인증서를 요구
2. `서버 → 클라이언트` 암호화된 자료를 전달
3. `클라이언트 → 인증서 기관` SSL 인증서 서명과 Server_Public을 얻기 위해 CA_Public 요청
4. `클라이언트` CA_Public을 받아 인증서를 복호화하고 서명을 검증해서 유효한 서명인지 확인함
5. `클라이언트` 클라이언트의 대칭키를 생성 후, 대칭키를 Server_Public으로 암호화함
6. `클라이언트 → 서버` 암호화된 대칭키를 서버로 전달
7. `서버` 암호화된 대칭키를 Server_Private으로 복호화를 진행해서 대칭키를 얻음
8. `클라이언트 <-> 서버` 이후 대칭키로 암호화해서 진행함

* 클라이언트 - 서버 사이에 CA라는 제3자를 두어서 CA_Public이 없으면 인증서 서명 검증을 할 수 없도록 함

[서명 검증 과정]

1. `복호화된 해시값`CA_Public으로 인증서를 복호화하면 서명에 사용된 해시 함수와 특정 해시값이 나옴
2. `새로운 해시값` 서명에 사용된 해시 함수를 인증서 내용에 해싱하면 새로운 해시값이 나옴
3. `복호화된 해시값` = `새로운 해시값`을 확인해서 같은 값이면 유효한 서명임

### 4-2) 실제로 동작하는 SSL/TLS Handshake 동작 과정

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e6b718b3-f852-4ca1-8e1f-f956dcee56fc)

여기서 다양한 정보들을 전송하는데, 중요한 전달 과정에 대해서만 설명함

- `빨간색` - `클라이언트 → 서버`
- `파란색` - `서버 → 클라이언트`

1. `Client Hello` : 사용하는 SSL/TLS 버전, 사용 가능한 암호화 알고리즘 목록
2. `Server Hello` : Client Hello에서 보낸 암호화 알고리즘 선택
    
    ![image](https://github.com/kauKoala/Backend-Study/assets/79046106/0f18b935-59ad-49d8-a0de-39a7df7be7c0)
    
    - `TLS`: TLS 통신 진행함
    - `ECDHE`: ECDHE 암호화 알고리즘을 통해 키 교환을 진행함 (비대칭키)
    - `RSA`: 서버의 인증서가 RSA로 암호화 되어 있음 (비대칭키)
    - `AES_256_GCM`: AES 알고리즘을 진행, key size는 256비트이고, GCM 모드로 수행함 (대칭키)
    - `SHA384`: 메시지 인증 코드로 패킷의 손상/변형 여부를 검증함 (해시 함수)
3. `Certificate`: 서버가 클라이언트에게 인증서 정보를 전송함 (RSA로 암호화되어 있음)
4. `Server Hello Done`: 서버가 보낼 패킷이 끝났음을 알림
5. `Client Key Exchange`: 대칭키를 생성하여 Server_Public으로 암호화해서 전송
6. `Change Cipher Spec`: 협상된 알고리즘과 키로 메시지를 암호화할 것을 알림
7. `Finish`: 클라이언트에서 보낼 패킷 끝 (패킷을 암호화해서 전송)
8. `Change Cipher Spec`: 서버도 협상된 알고리즘과 키로 메시지를 암호화할 것을 알림
9. `Finish`: 서버에서 보낼 패킷 끝 (패킷을 암호화해서 전송)

---

## 5. 대칭키 알고리즘 동작 과정 (레인달 알고리즘)

---

대칭키 알고리즘은 AES 알고리즘이 많이 사용되고 있음

[AES 알고리즘이란?]

Advanced Encryption Standard

미국 표준 기술 연구소에서 2001년에 발표한 알고리즘으로 AES 알고리즘은 공모전을 열어 채택한 알고리즘이라 진짜 알고리즘은 Rijndael 알고리즘(레인달 알고리즘)을 사용함

### 암호화 동작 과정

출처: [http://ho-story.tistory.com/27](http://ho-story.tistory.com/27) (아래 내용은 간략하게 설명한거라 링크를 보는게 더 좋음)

- AES-256은 블록사이즈가 128비트로 14 round로 진행 (key size가 256비트)
- 레인달 알고리즘은 128비트만 존재함
- 암호화는 4x4 행렬을 사용한 연산이 진행
- 동작 과정은 크게 4가지 원리로 이루어져있음

[초기 세팅]

- 데이터를 16바이트 단위로 나눠서 4x4 행렬로 세로 방향으로 채워줌 (= 상태행렬)
- 비밀키도 4x4 행렬로 상태행렬 만드는 방식과 똑같이 만들어냄 (= 라운드키)

[1] Add Round Key

새로운 상태행렬을 생성해냄

new 상태행렬 = old 상태행렬 `XOR` 라운드키

[2] Sub Byte 변환

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/d9b80402-a2b3-469a-a235-4bcdf5832549)

위 표를 보면서 변환을 진행함

93 → x = 9, y = 3 → DC로 변환

[Shift Rows]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/1d520c14-a6c2-4cf7-a02b-c4c483aec3c1)

- 0행은 움직이지 않음
- 1행은 왼쪽으로 한 칸 이동
- 2행은 왼쪽으로 두 칸 이동
- 3행은 왼쪽으로 세 칸 이동

[Mix Column]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/af04bd5c-bcea-49b4-91f3-071dfad06474)

위의 행렬과 행렬 곱셈을 진행함

[Before Main Round]

1. AddRoundKey

[Main Round - 1 round ~ 13 round ]

1. Sub Bytes
2. ShiftRows
3. MixColumns
4. AddRoundKey

[Final Round - 14 round]

1. Sub Bytes
2. ShiftRows
3. AddRoundKey

### 복호화 동작 과정

모든 것을 역순으로 진행하면 된다.

- S box → Reverse S box
- Shift Rows는 왼쪽 shift 하는게 아닌 오른쪽 shift를 진행함 (역순)
- Mix Columns에서 사용된 행렬의 역행렬을 사용해야함

설명 생략…

---

## 6. 이외에 궁금해서 찾아본 내용

---

[1] 비대칭키가 보안적으로 유리한데, 대칭키로 통신하는 이유

RSA로 암호화를 한다면 거의 뚫을 수 없다는 특징을 가지고 있지만 속도가 느림
(5번에 나온 대칭키 알고리즘 동작 방식만 하더라도 패킷 하나에 이런 암호화 / 복호화 연산이 진행되는데 RSA는 그만큼 느리게 작동함)

[2] 4-1 구체적인 SSL/TLS 통신과정에서 CA의 공개키는 이미 모두에게 공개된 키인건지

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/625e90ba-4849-404a-99fa-a6033da59ac3)

이 값으로 모두 똑같음

확인 방법(MacOS 기준): 크롬 → 설정 → 개인 정보 보호 및 보안 → 보안 → 인증서 관리 → 키체인 접근 → 검색창에 Digicert 입력 → Digicert Global Root CA 확인

[3] 네이버는 TLS 버전을 어떤걸 사용하는지 확인해보기

- 터미널에서 openssl을 다운하면 확인해볼 수 있음 (`brew install openssl`)
- `openssl s_client -connect www.naver.com:443`을 입력하면 SSL/TLS 정보 확인이 가능함

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/8ae8de3f-7857-40f7-b0da-e707543ca568)

- TLS 1.3인 최신 버전을 사용함

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/aeb86b4b-34f1-4c8b-8660-84ae98b24dd8)

- 80번 포트로 openssl 명령어를 보내면 당연하게도 보안을 지원해주지 않음

---

## 7. 출처

---

- [CLOUDFLARE - TLS 핸드셰이크의 원리는 무엇일까요?](https://www.cloudflare.com/ko-kr/learning/ssl/what-happens-in-a-tls-handshake/)
- [HTTP 완벽 가이드 정리해둔 깃허브](https://github.com/Tech-Book-Learning/HTTP-Perfect-Guide/blob/master/%5B14%5D%20%EB%B3%B4%EC%95%88%20HTTP/README.md)
- [블로그 - HTTPS 동작과정 이해하기](https://mark-kim.blog/HTTPS/) (여기 다른 글들도 진짜 정리 잘해둠)
- [블로그 - AES 암호화](https://junstar92.tistory.com/192)
- GPT - 4o
