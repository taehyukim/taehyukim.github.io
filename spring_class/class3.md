# 3회차

## 수업내용

### Filter & Interceptor
Spring에서 dispatcherServlet이 HTTP request를 받아 controller에 전달하는 중간 과정에 관여하는 두 녀석이 존재한다.

![Spring MVC Request Lifecycle](http://4.bp.blogspot.com/-r9Ptpj782zU/UVvoNtbelHI/AAAAAAAAADY/eFlCBleusuw/s1600/89101625_26c5be9fd9.jpg)

**Filter**는 request를 받아 dispatcherServlet에 전달하기 전에 호출된다.
- init() : 필터 인스턴스 초기화
- doFilter() : 전/후 처리
- destroy() : 필터 인스턴스 종료

**(Handler)Interceptor**는 request가 dispatcherServlet를 지나 controller에 도달하기 전에 호출된다.
- preHandle() : 컨트롤러 들어가기 전
- postHanle() : 컨트롤러 들어갔다 나온후 뷰로 보내기전
- afterCompletion() : 뷰까지 끝나고 나서

Filter와 interceptor는 설정파일(xml 또는 java config)에 선언하여 사용한다.
Controller를 handler라고도 부르며, interceptor는 handlerInterceptor interface를 구현하여 사용한다.

```java
@Configuration // java config class 는 해당 annotation 이 붙어 있어야 한다.
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 사용자가 작성한 interceptor 를 추가한다.
        registry.addInterceptor(new LogInterceptor());
    }
}

public class LogInterceptor extends HandlerInterceptorAdapter {
    @Override
    // dispatcherServlet 이 controller 를 실행하기 전에 preHandle 을 호출
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle :" + handler + " :: " + request.getAttribute("st"));
        return true; // true 를 반환해야 정상적으로 수행됐다고 판단하고 controller 작업 수행
    }

    @Override
    // dispatcherServlet 이 controller 를 실행하고 난 후에 postHandle 을 호출
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle :" + handler + " time: " + (endTime - startTime));
    }
}
```

### HandlerMethodArgumentResolver
**HandlerMethodArgumentResolver** interface는 controller의 parameter를 binding해주는 역할을 한다.
즉, 이 interface를 구현한 argumentResolver 객체에서 request가 controller(handler)에 도달하기 전에 그 parameter를 수정, 관리할 수 있다.

```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class MemberInfo {
    private Integer id;
    private String email;
}

public class MemberInfoArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        if(parameter.getParameterType() == MemberInfo.class) {
            return true;
        }
        return false;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        MemberInfo memberInfo = new MemberInfo(35, "taehyukim@ebay.com");
//        MemberInfo memberInfo = new MemberInfo();
        return memberInfo;
    }
}
```

ArgumentResolver 역시 설정파일에 등록한 후에 사용 가능하다.

```java
@Configuration // java config class 는 해당 annotation 이 붙어 있어야 한다.
public class WebConfig implements WebMvcConfigurer {
    // ArgumentResolver 를 추가한다.
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new MemberInfoArgumentResolver());
    }
}
```

이제 controller를 호출하여 확인해보면, 아무 값을 넣지 않아도 argumentResolver에서 설정한 값이 parameter로 넘어오는 것을 확인할 수 있다.

```java
@RestController
@RequestMapping("/api/members")
public class MemberApiController {
    @GetMapping
    public String member(MemberInfo memberInfo) {
        System.out.println("members : " + memberInfo);
        return "member - id: " + memberInfo.getId() + ", email: " + memberInfo.getEmail();
    }
}
```

### ThreadLocal
일반적으로 변수는 특정 code block 안에서만 유효하지만, ThreadLocal class를 사용하면 thread 영역에 변수를 설정할 수 있으며, 이를 통해 같은 thread 안에서 정보를 공유할 수 있다.
Request scope은 하나의 thread로 동작하기 때문에, threadLocal을 통해 값을 다룰 수 있다.

![ThreadLocal 동작 방식](http://cfs14.tistory.com/image/36/tistory/2009/02/13/15/44/499516b885314)

같은 코드를 실행해도 각각의 thread에 값이 저장되고 사용된다.
- ThreadLocal 사용법
    - ThreadLocal 생성
    - ThreadLocal.set() 으로 값 저장
    - ThreadLocal.get() 으로 값 읽기
    - ThreadLocal.remove() 로 값 삭제 (삭제하지 않으면 재사용되는 thread가 올바르지 않은 값을 참조할 수 있다.)

```java
public class LogContext {
    public static ThreadLocal<Long> time = new ThreadLocal<Long>();
}

public class LogInterceptor extends HandlerInterceptorAdapter {
    @Override
    // dispatcherServlet 이 controller 를 실행하기 전에 preHandle 을 호출
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        LogContext.time.set(System.nanoTime());
        System.out.println("preHandle :" + handler + " :: " + LogContext.time.get());
        return true; // true 를 반환해야 정상적으로 수행됐다고 판단하고 controller 작업 수행
    }

    @Override
    // dispatcherServlet 이 controller 를 실행하고 난 후에 postHandle 을 호출
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        long endTime = System.nanoTime();
        long startTime = LogContext.time.get();

        System.out.println("postHandle :" + handler + " time: " + (endTime - startTime));
    }
}
```

Thread 관련 코드에서 parameter를 사용하지 않고 값을 전달하기 위해 주로 사용되며, 주 용도는 다음과 같다.
- 사용자 인증정보 전파 - Spring Security에서는 ThreadLocal을 이용해서 사용자 인증 정보를 전파한다.
- 트랜잭션 컨텍스트 전파 - 트랜잭션 매니저는 트랜잭션 컨텍스트를 전파하는 데 ThreadLocal을 사용한다.

### 기타
**[Pingendo](https://pingendo.com/)** - Free Bootstrap 4 Builder


## 과제

### Builder Pattern과 Lombok을 사용한 Effective Java 의 Builder Pattern 구현
**Builder Pattern**
Constructor를 통해 인스턴스를 생성할 때, parameter로 너무 많은 값이 들어가면 어떤 값이 무엇을 의미하는지 알아보기가 힘들다.
또한 setter가 없이 생성자로만 데이터를 입력받는 경우, 일부 parameter로만 인스턴스를 생성한다던가 parameter 순서가 바뀌는 경우 생성자를 추가해야한다.

이러한 문제들을 보완하기 위해 Builder pattern은 생성자에 필요한 parameter들을 하나씩 차례대로 입력받아 한번에 생성하는 디자인 패턴이다.

```java
public class Member {
    private String name;
    private Integer age;
    private String addr;

    // 일반적인 생성자
    public Member(String name, Integer age, String addr) {
        this.name = name;
        this.age = age;
        this.addr = addr;
    }

    public String getName() {
        return name;
    }

    public Integer getAge() {
        return age;
    }

    public String getAddr() {
        return addr;
    }
}

// builder pattern을 적용한 builder class
public class MemberBuilder {
    private String name;
    private Integer age;
    private String addr;

    public MemberBuilder setName(String name) {
        this.name = name;
        return this;
    }

    public MemberBuilder setAge(Integer age) {
        this.age = age;
        return this;
    }

    public MemberBuilder setAddr(String addr) {
        this.addr = addr;
        return this;
    }

    public Member build() {
        Member member = new Member(name, age, addr);
        return member;
    }
}

Member memeber = new MemberBuilder()
    .setName("kim")
    .setAge(32)
    .setAddr("Namyangju")
    .build();
```

**Lombok의 @Builder annotation**을 사용하면 builder pattern과 유사한 기능을 손쉽게 구현할 수 있다.

```java
// lombok의 builder annotation
@Builder
public class Member {
    private String name;
    private Integer age;
    private String addr;
}

Member.MemberBuilder builder = Member.builder();
builder.name("kim");
builder.age(32);
builder.addr("Namyangju");
Member member = builder.build();

Member memeber_new = new Member.builder()
    .name("kim")
    .age(32)
    .addr("Namyangju")
    .build();
```

### Interface의 Default Method
Interface는 일반적으로 사용할 method를 선언만 하고, interface를 구현하는 class에서 이 method를 구현한다.
Java8부터 Interface의 method가 **default**로 선언되면 interface 내부에서 method를 구현할 수 있다.
또한 해당 interface 구현하는 class에서 default method를 overriding하여 사용할 수 있다.

```java
public interface Calculator {
    public int plus(int i, int j);
    public int minus(int i, int j);
    // default로 선언하여 method를 구현할 수 있다.
    default int multiple(int i, int j) {
        return i * j;
    }
    // interface에서 선언한 static method는 interface.method 형태로 호출해야 한다.
    // ex) int result = Calculator.multiple2(2, 3);
    public static int multiple2(int i, int j) {
        return i * j;
    }
}
```

Default method를 이용하면 interface에 새로운 method를 추가해도, 기존에 해당 interface를 구현하던 class의 수정 없이 호환성을 유지하면서 라이브러리를 변경할 수 있다.
이와 비슷하게 interface에서 **static method**도 선언 및 구현이 가능하다.

### Java Web Programming의 4가지 Scope
Java Web Programming에는 4가지 객체 범위(Scope)가 존재한다.
1. **Page Scope**
    - 하나의 JSP가 호출될 때 해당 JSP 내에서만 객체를 공유하는 범위를 의미한다.
    - JSP 파일에는 pageContext 객체가 내장되어 있으며, 이 객체는 page 영역에서만 유효하다.
    - JSP 파일의 <% %> 안에 변수를 사용하면 그 변수는 해당 JSP 내에서만 유효하다.
2. **Request Scope**
    - Request를 받아서 그에 대한 response를 보내기 까지 객체가 유효한 범위를 의미한다.
    - Forward또는 include를 이용하는 경우 여러개의 page에서도 하나의 request가 유지되므로, request 범위의 객체를 여러 페이지에서 공유할 수 있다.
    - HttpServletRequest의 setAttribute()/getAttribute()를 통해 key-value 값을 처리한다.
3. **Session Scope**
    - Session scope 설명
4. **Application Scope**
    - Application scope 설명