# Bean 등록이 가능한 Filter

과거에는 Servlet Context로 관리되는 Filter는 Bean 등록이 불가능했고, Bean을 주입받을 수도 없었음

But,

Spring 기반의 개발에서 Filter에 DI 기술이 필요한 상황이 생겼음

## DelegatingFilterProxy

- Spring 1.2부터 등장 (2005년)
- Servlet Filter에서 관리되는 프록시용 Filter
- 개발자가 정의한 Filter 정보를 가지고 있음
- DelegatingFIlterProxy가 요청을 먼저 받고, 개발자가 정의한 Filter에게 요청 위임

```java
public class MyWebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

   public void onStartup(ServletContext servletContext) throws ServletException {
      super.onStartup(servletContext);
      servletContext.addFilter("loginFilter", DelegatingFilterProxy.class);
   }
}
```

- SpringBoot에서는 위와 같이 DelegatingFilterProxy에 Filter를 등록해줄 필요도 없음

## ServletRequest 직접 조작하기 - Filter

```java
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

public class MultipartRequestFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;

        if (httpRequest.getContentType() != null && httpRequest.getContentType().startsWith("multipart/form-data")) {
            // 멀티파트 요청 처리 
            handleMultipartRequest(httpRequest);
        }

        chain.doFilter(request, response);
    }

    private void handleMultipartRequest(HttpServletRequest request) {
        
        System.out.println("Handling multipart request");
    }
}
```

- (HttpServletRequest의 InputStream은 한 번만 읽을 수 있음)
- 요청이나 응답의 직접적인 조작이 필요할 때
- 요청 Body를 여러 번 읽는 저수준 작업이 필요할 때

## afterCompletion 활용하기 - Interceptor

```java
public class RequestProcessingTimeInterceptor implements HandlerInterceptor {

    private static final String START_TIME = "startTime";

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        
        long startTime = (Long) request.getAttribute(START_TIME);
        long endTime = System.currentTimeMillis();
        long processingTime = endTime - startTime;

        
        System.out.println("Request to " + request.getRequestURI() + " took " + processingTime + " ms.");
    }
}
```

- Controller 로직과 View 렌더링이 모두 완료된 후에 호출되기 때문에, 전체 요청 처리 시간을 정확하게 측정할 수 있음
- Controller에서 발생한 예외 처리 로직을 구현하기가 용이함