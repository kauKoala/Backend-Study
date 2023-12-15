# 1. OAuth와 사용경험

### 1. OAuth란?

서비스 사용자들이 비밀번호를 제공하지 않고, 애플리케이션의 접근 권한을 위임 받을 수 있는 방식

[OAuth 사용 이유] 클라이언트 입장에서 보안 수준이 어떤지 알 수 없는 사이트에 정보를 제공하는 것보다 신뢰가 있는 사이트에 정보를 가져와서 해당 서비스에 정보를 건네주기 위해 만들어짐

<br/>

### 2. OAuth 작동 방식

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e17016aa-b30e-468b-9e67-7114aaf8f493)

<br/>

### 3. OAuth 1.0 → OAuth 2.0로 넘어오면서 달라진 점

- OAuth 권한을 얻을 수 있는 승인 방식이 2번에서 1번으로 간소화
- 액세스 토큰 유효 기간이 생겼고, 토큰 발급이 쉬워짐

<br/>

### 4. OAuth vs OIDC

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/967e7fc0-245c-4221-b519-2575fadbede8)

OAuth의 단점은 사용자가 누군지 모르고, 액세스 토큰을 통해 권한을 제공함

하지만 OIDC의 경우에는 클라이언트의 고유한 ID 토큰을 통해 사용자가 누군지 구분이 가능함

OIDC의 장점의 경우에는 OAuth는 사용자 정보를 가져오려면 액세스 토큰을 얻고, 액세스 토큰으로 사용자 정보를 가져오는 두 번의 통신을 사용해야하는데, OIDC는 한 번의 과정으로 사용자 정보를 얻을 수 있음

<br/>
<br/>
이미지 출처 - [hudi.blog - OAuth 2.0 개념과 동작원리](https://hudi.blog/oauth-2.0/)
