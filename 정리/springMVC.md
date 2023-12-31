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

## 뷰 리졸버
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
        return new ModelAndView("new-form");
    }
}

```
application.properties에 다음을 추가
```
spring.mvc.view.prefix=/WEB-INF/views
spring.mvc.view.suffix=.jsp
```

스프링 부트는 `InternalResourceViewResolver`라는 뷰 리졸버를 자동으로 등록하는데 이떄 `application.properties`에 등록한 설정 정보를 사용해서 등록

1. 핸들러 어댑터 호출
* 핸들러 어댑터를 통해 new-form 이라는 논리 뷰 이름을 획득
2. ViewResolver 호출
* new-form 이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
* `BeanNameViewResolver` 는 `new-form` 이라는 이름의 스프링 빈으로 등록된 뷰를 찾아야 하는데 없음
* `InternalResourceViewResolver` 가 호출

3. InternalResourceViewResolver
* 이 뷰 리졸버는 `InternalResourceView` 를 반환

4. 뷰 - InternalResourceView
* InternalResourceView 는 JSP처럼 포워드 forward() 를 호출해서 처리할 수 있는 경우에 사용
5. view.render()
* view.render() 가 호출되고 `InternalResourceView`는 forward() 를 사용해서 JSP를 실행

***
## 스프링 MVC 시작
```java
package hello.servlet.web.springmvc.v1;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class SpringMemberFormControllerV1 {
    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        System.out.println("SpringMemberFormControllerV1.process");
        return new ModelAndView("new-form");
    }
}
```
* @Controller
  * 스프링이 자동으로 스프링 빈으로 등록
  * 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식
* @RequestMapping
  * 요청 정보를 매핑, 해당 URL이 호출되면 이 메서드가 호출, 애노테이션을 기반으로 동작해 메서드의 이름은 임의로 지어도 됨
* ModelAndView
  * 모델과 뷰 정보를 담아서 반환하면 됨

