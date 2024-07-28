### CORS

---

Cross Origin Resource Sharing으로 한 출처에서 실행된 웹 어플리케이션이 다른 출처의 자원에 접근할 수 있도록 권한을 부여하는 체재

SOP (Same Origin Policy)

옛날 인터넷에서는 다른 포트번호나 호스트, 프로토콜 같이 다른 출처의 자원을 접근하지 못하도록 막아두었음

그런데 CORS를 통해 다른 출처에 대한 자원 접근이 가능해짐

### CORS 동작 방식

---

1) Preflight Request

원래 요청을 보내기 전에 간단한 요청을 보내는 방법

OPTIONS 헤더를 적어 보내면 응답으로 Access-Control-Allow-Origin 데이터가 들어있는 응답값을 받음

2) Simple Request

바로 요청을 보내는 방법

이 경우 패킷이 큰 요청을 보냈지만, CORS 에러가 발생해 거절당하면 리소스 낭비가 될 수 있음

3) Credential Request

헤더에 인증 정보를 담아보내는 요청

서버에서는 `Access-Control-Allow-Credentials : true`를 설정하고, `Access-Control-Allow-Origin` 의 값은 와일드카드(*)가 아닌 Origin 값을 명확히 적어야함

### Samesite 옵션

---

크롬에서는 쿠키의 Samesite 옵션이 Lax라 따로 옵션을 설정해주어야함

1. None
도메인 사이트가 다른 경우에도 브라우저가 쿠키를 포함해서 보낼 수 있음 (설정하려면 https가 적용되어야함)
2. Strict
도메인 사이트가 같은 경우에만 브라우저가 쿠키를 보낼 수 있음
3. Lax
도메인 사이트가 같거나, 타 도메인에서 본래 도메인 사이트 링크를 통해 이동하면 쿠키를 보낼 수 있음
