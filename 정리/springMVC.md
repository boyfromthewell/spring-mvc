* 스프링 MVC도 프론트 컨트롤러 패턴으로 구현
* 스프링 MVC의 프론트 컨트롤러가 바로 디스페쳐 서블릿(DispatcherServlet)
* 이 디스페쳐 서블릿이 스프링 MVC의 핵심

### DispacherServlet 등록
* DispacherServlet도 부모 클래스에서 HttpServlet을 상속 받아 사용, 서블릿으로 동작
* 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로 (`urlPatterns="/"`)에 대해서 매핑

### 요청 흐름
* 서블릿이 호출되면 `HttpServlet`이 제공하는 `service()`가 호출
* 스프링 MVC는 DispatcherServlet의 부모인 `FrameworkServlet`에서 `service`를 오버라이드 함
* `FrameworkServlet.service()`를 시작으로 여러 메서드가 호출 `DispathcerServlet.doDispatch()`가 호출

**동작 순서**
1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러를 조회
2. 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회
3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행
4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행
5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 *변환*해서 반환
6. viewResolver 호출 : 뷰 리졸버를 찾고 실행
7. View 반환 : 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고 렌더링 역할을 담당하는 뷰 객체를 반환
8. 뷰 렌더링 : 뷰를 통해 뷰를 렌더링

**가장 큰 강점은 DispatcherServlet 코드의 변경 없이 원하는 기능을 변경 확장 가능 - > 대부분 확장 가능할 수 있게 인터페이스로 제공**<br/>
인터페이스만 구현해 DispathcerServlet에 등록하면 나만의 컨트롤러를 만드는것도 가능

```java
package hello.servlet.web.springmvc.old;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}

```
## 핸들러 매핑과 핸들러 어댑터
1. 핸들러 매핑으로 핸들러 조회
* `HandlerMapping`을 순서대로 실행 핸들러를 찾음 
* 빈 이름으로 핸들러를 찾아야해 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping`가 실행에 성공, 핸들러인 `OldController` 반환

2. 핸들러 어댑터 조회
* `HandlerAdapter`의 `supports()`를 순서대로 호출
* `SimpleControllerHandlerAdapter`가 `Controller`인터페이스를 지원하므로 대상이됨

3. 핸들러 어댑터 실행
* 디스패쳐 서블릿이 조회한 `SimpleControllerHandlerAdapter`를 실행하면서 핸들러 정보도 함께 넘겨줌
* SimpleControllerHandlerAdapter는 핸들러인 `OldController`를 내부에서 실행, 그 결과를 반환

```java
package hello.servlet.web.springmvc.old;

import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.HttpRequestHandler;

import java.io.IOException;

@Component("/springmvc/request-handler")
public class MyHttpRequestHandler implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MyHttpRequestHandler.handleRequest");
    }
}

```
1. 핸들러 매핑으로 핸들러 조회
* `HandlerMapping`을 순서대로 실행 핸들러를 찾음
* 빈 이름으로 핸들러를 찾아야해 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping`가 실행에 성공, 핸들러인 `MyHttpRequestHandler` 반환

2. 핸들러 어댑터 조회
* `HandlerAdapter`의 `supports()`를 순서대로 호출
* `HttpRequestHandlerAdapter`가 `HttpRequestHandler`인터페이스를 지원하므로 대상이됨

3. 핸들러 어댑터 실행
* 디스패쳐 서블릿이 조회한 `HttpRequestHandlerAdapter`를 실행하면서 핸들러 정보도 함께 넘겨줌
* HttpRequestrHandlerAdapter는 핸들러인 `MyHttpRequestHandler`를 내부에서 실행, 그 결과를 반환

