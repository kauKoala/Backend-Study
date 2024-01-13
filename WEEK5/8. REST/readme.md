**Representational State Transfer**

API 설계 방식 중 하나. 다른 설계 방식으론 GraphQL, GRPC 등이 있다.
REST 방식으로 설계한 API를 RESTful API라고 한다.
모든 콘텐츠는 자원(리소스)로 분류된다.

## REST 방식 특징

- HTTP 방식을 사용해 클라이언트, 서버, 자원으로 구성된 아키텍처
- 무상태
    - 이전 요청과 독립적으로 수행할 수 있다.
    다시 말하면, 서버가 이전 요청을 기억할 필요 없이 요청만 보고 이해할 수 있다.
- 캐시 제공
    - Cache-Control 헤더를 통해 머릿글, 바닥글같이 고정된 자원은 클라이언트에서 캐싱 할 수 있다.
- 계층화 시스템
    - 클라이언트 - 서버 사이의 다른 미들웨어(보안, 로드 밸런싱)에 연결될 수 있다.
    이 과정은 클라이언트에서 볼 수 없다.

## RESTful API 의 구성

### Request

- 리소스 식별자(URL, URI)
- HTTP 메서드
    - GET: 서버 자원에 대한 접근
    - POST: 서버에 자원을 생성
    - DELETE: 서버 자원을 삭제
    - PUT: 자원의 모든 부분을 수정, 멱등성 보장
        - 멱등성을 보장한단 것은 캐시의 용도로 쓸 수 있단 건가?
    - PATCH: 자원의 일부를 수정
- HTTP 헤더
    - 데이터
    - 파라미터 (경로, 쿠키, 쿼리)

### Response

- HTTP Status
- 메시지 본문 (JSON, XML 등)
- HTTP 헤더

## URL과 URI의 차이

웹 페이지 링크 뿐만 아니라, 자원의 위치를 표시하는데 사용된다.

잘 정리된 링크의 글을 대략적으로 정리했다.

https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn

example.com/test.html은 **test.html 자원의 위치를 명시하기 때문에 URL이다.**
또한, **test.html 은 식별자 역할도 해 URI다.**

반면 [example.com/t](http://example.com/RogerPate)est 의 경우 test 자원을 식별해 **URI다.**
하지만, 이 **test는 test.html 이 될 수도 있고, test.xml이 될 수도 있다.**
자원의 위치까진 명시하지 못해 **URL은 아니다.**

![1](https://github.com/kauKoala/Backend-Study/assets/7845568/6f6469a3-69eb-4fd5-a65d-95c1ef8489f8)



## 컨벤션

- **동사 지양, 명사 위주**의 사용 ex) /getUsers가 아닌 /users 사용
    - 동사는 HTTP 메서드로 표현이 가능하다.
- 버전 관리 (**v1**, **v2**를 통해 같은 요청이라도 버전을 다르게 할 수 있다.)
- 언더바(**_**) 대신 하이픈(**-**) 사용
- 계층 분리엔 슬래시(**/**)를 사용하되, 마지막엔 사용 ******X
example.com/path******
- 파일 확장자 표시 **X**
- **소문자** 사용

## Reference

https://aws.amazon.com/ko/what-is/restful-api/

https://www.redhat.com/en/topics/api/what-is-a-rest-api

https://youtu.be/RP_f5dMoHFc