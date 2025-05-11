### RFC 7807 - Problem Details for HTTP APIs

상태코드만으로는 오류에 대한 정보가 충분하지 않은 상황이 되었음

만약 계좌를 인출할 때 잔액이 부족하다는 응답을 생각해보면 `403 Forbidden`이 가장 적절할 수 있는데, 해당 상태코드만으로는 서버에서 클라이언트 요청이 왜 거부 됐는지, 현재 계좌가 얼마인지 등에 대한 구체적인 정보를 알 수 없음

그래서 해당 문서에서 아래와 같이 문제 세부 정보를 포함하는 HTTP 응답을 표준화하기로 함

```kotlin
HTTP/1.1 403 Forbidden  
Content-Type: application/problem+json  
Content-Language: en  

{
  [type] 문제 유형을 URI로 식별할 수 있도록 함
  (해당 URI에 접속하면 사람이 읽을 수 있는 문서를 제공하는걸 권장함)
  "type": "https://example.com/probs/out-of-credit",
  
  [title] 403 Forbidden 에러가 나온 이유를 요약함
  "title": "You do not have enough credit.",
  
  [status] HTTP 상태번호 (integer)
  "status": 403
  
  [detail] 에러가 발생한 구체적인 설명을 해줌
  "detail": "Your current balance is 30, but that costs 50.",
  
  [instance] 오류가 발생한 특정 상황을 식별하는 URI
  (클라이언트가 보내는 요청일 수도 있고, 로깅에서 식별하기 위해 값을 커스텀할 수도 있음)
  "instance": "/account/12345/msgs/abc",
  
  [balance, accounts] 문제에 특화된 추가 정보를 담아주는 곳 (확장 필드)
  "balance": 30,
  "accounts": ["/account/12345", "/account/67890"]
}
```

### RFC 9457 - Problem Details for HTTP APIs

2023년 7월에 RFC 7807 문서에서 부족했던 내용을 보강해서 문서를 대체함

### 스프링에서의 에러 메시지 표준화

스프링에서는 ErrorAttirbutes (구현체: DefaultErrorAttributes)를 통해 에러 메시지를 세부적으로 지원함

이때는 RFC 7807을 공식적으로 지원하지 않았고, 지원하려면 직접 구현이 필요했음

```kotlin
Spring Web Servlet - DefaultErrorAttributes

[에러가 발생한 시각]
timestamp - The time that the errors were extracted

[상태 코드]
status - The status code

[에러 발생 원인]
error - The error reason

[예외 클래스]
exception - The class name of the root exception (if configured)

[예외 클래스에 설정된 메시지]
message - The exception message (if configured)

[검증 과정에서 사용하는 객체 목록 - 설정됐을 때 해당 필드에 값을 넣어서 보여줌]
- BindingResult: @Valid, @Validated
- MethodValidationResult: @NotNull, @Size, @Min, @Max, ...
errors - Any ObjectErrors from a BindingResult or MethodValidationResult exception (if configured)

[stack trace]
trace - The exception stack trace (if configured)

[예외가 발생한 URL 경로]
path - The URL path when the exception was raised
```

스프링 부트 3부터는 RFC 7807을 공식적으로 지원하면서 `ProblemDetails`라는 에러 핸들링 메시지 표준을 지원해주고 있음

```kotlin
public class ProblemDetail {
    
    // 개발자가 따로 type 값을 설정하지 않을 때 기본값
    private static final URI BLANK_TYPE = URI.create("about:blank");
    
    private URI type;
    @Nullable
    private String title;
    private int status;
    @Nullable
    private String detail;
    @Nullable
    private URI instance;
    @Nullable
    private Map<String, Object> properties; // 확장 필드
    
    ...
```

결론: 에러 메시지도 표준이 나왔으니 저기에 나온 정보를 베이스로 필요한 정보를 제공해주자
