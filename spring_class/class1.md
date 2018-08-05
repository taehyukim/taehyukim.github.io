# 1회차

## 수업내용

Spring에서 중요하게 생각하는 **interface**.
(Bean 컨테이너, DI 컨테이너, IoC 컨테이너)

BeanFactory,
ApplicationContext,
WebApplicationContext,

Class의 instance 생성 및 life cycle을 spring container가 관리해준다.
관리해주는 녀석들을 **Bean** 이라고해

1. Bean을 등록하는 방법
    - xml파일
    - xml파일 <--> java config(import)
    - xml -> xml config
    - java config파일
    - java config -> 또 다른 java config
    - Component scan 등

2. 왜 이런 방식을 사용하나


Hello.java 컴파일하면 Hello.class 라는 byte code 생성.
java Hello 명령은 JVM이 Hello class를 찾아서 실행하라는 의미.
JVM이 CLASSPATH 환경변수에 등록된 경로에서 해당 class를 찾는다.
JVM은 SystemClassLoader를 통해 class를 찾는다.

WAS에서 실행되는 Web Application들 구분하는 값이 context path.
JVM도 CL(ClassLoader)를 갖고있고, WAS도 CL를 갖고있고, WebApp도 각각 CL을 갖고 있다.

OOP + AOP
- Method 시작, 종료, exception 등에 관여하는 **Advice 객체**
- Advice 객체를 어떤 JoinPoint에 연결할 것인가 설정 = **PointCut**
- Advice, JoinPoint와 PointCut을 합쳐서 **Aspect 객체**

Interface는 **Proxy 객체**를 동적으로 생성하여
각 method를 실행할 때 Aspect에 설정한대로
method 시작, 종료, exception 등의 시점에 해당 advice 객체를 호출하고
실제 구현 method를 호출하도록 동작

Interface를 사용하지 않고 바로 객체에 구현하는경우,
proxy 역할을 해주는 객체를 동적으로 생성해주는 library도 있다 (ex. CGLIB)

Spring에서 웹앱을 만들면 보통 Layered Architecture 구조.
- Controller -> View (JSP)를 호출하여 띄우고, 
- Controller -> Service -> Repository 형태로 호출하는 구조.

Business logic은 service에서 구현하고, controller가 service 호출한다.
내장된 WAS로 WebApp을 실행할 때에는 JSP를 사용할 수 없어서
다른 templete을 사용해야 View를 띄울 수 있다

여러 Controller가 하나의 Service를 사용할 수 있으며,
이러한 service가 **공용객체**

Bean들은 (생성자 역할의) method명을 id로 갖기 때문에
중복될 수 없고 고유값이어야 한다.


## 과제

### ClassLoader에 대해 조사
Java에서 ClassLoader란 run-time에 필요한 class를 .class 파일로부터 읽어들여 동적으로 메모리에 로드하는 역할을 수행하는 녀석이다.
동일한 클래스는 한번만 로드한다.

### ClassLoader의 계층구조
ClassLoader는 다음과 계층구조로 되어있으며, 부모 CL이 우선권을 갖는다. 즉, 부모 CL이 먼저 요청받은 class를 찾고, 없을 경우 그 하위 CL이 찾는 방식으로 동작한다.

![Class Loader Hierarchy](https://docs.oracle.com/cd/E19501-01/819-3659/images/dgdeploy2.gif)

참고: [The Class Loader Hierarchy](https://docs.oracle.com/cd/E19501-01/819-3659/beadf/index.html)

### Memory구조
JVM의 Class Loader가 runtime시에 Class를 로드하여 메모리 영역에 보관하는데, 이 메모리 영역을 Runtime Data Area 라고 한다.
Runtime Data Areas는 Method(Static, Class) area, Heap area, Stack area, PC register, Native method stack 으로 이루어져 있다.

![Runtime Data Areas](http://coding-geek.com/wp-content/uploads/2015/04/jvm_memory_overview.jpg)

참고: [JVM Memory Model](http://hoonmaro.tistory.com/19)
1. Method Area
2. Heap Area
3. Stack Area
4. PC Register
5. Native Method Area

### AOP(Aspect Oriented Programming)의 용어 정리
비즈니스 로직과 공통 로직으로 나누어, 핵심 로직에 영향을 끼치지 않으면서 사이사이에 공통 로직을 효과적으로 잘 호출하도록 하는 개발 방법.
- **JoinPoint** : 공통 로직을 적용 가능한 지점. 메소드 호출, 종료 부분, 예외처리 부분 등.
- **PointCut** : 공통 로직을 실제로 적용할 JoinPoint를 정의하는 것.
- **Advice(Interceptor)** : 각 JoinPoint에 삽입되어 동작할 수 있는 코드. 적용되는 JoinPoint에 따라 Before Advice, After Returning Advice, After Throwing Advice, Around Advice, Introduction 등이 있다.
- **Weaving** : Advice를 핵심 로직에 적용하는 것. 즉, 공통코드를 핵심코드에 삽입하는 것.
- **Aspect(Advisor)** : PointCut + Advice 를 결합한 코드
- **Target** : 핵심 로직을 구현한 클래스로, Advice가 적용될 대상을 가리킨다.

![AOP 용어와 개념도](https://t1.daumcdn.net/cfile/tistory/223B623D548C415631)

### Proxy 객체
AOP를 구현할 때 핵심 로직을 구현할 객체에 직접 접근하는 대신, 중간에 Proxy 라는 객체를 동적으로 생성하여 **Proxy**를 통해 핵심 로직의 객체에 접근한다.
Proxy 객체에서 각 JoinPoint에 해당하는 Advice들을 호출하고 핵심 로직을 호출하는 방식으로 동작한다.

![프록시를 이용한 AOP 구현](https://i.imgur.com/63YMS1o.png)

참고: [Spring-AOP, Proxy란](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/)

### Thread와 공용객체
**Thread**는 하나의 프로그램(Process) 내에서 실행되는 흐름의 기본 단위를 말한다.
메모리 영역을 공유하기 때문에 프로세스간 통신보다 간단하며 효율적이다. (동일 Process 내의 Thread 들은 Code, Data, Heap 영역을 공유하며, Stack 영역만 각자 관리)

참고: [Process와 thread에 대한 정리](https://magi82.github.io/process-thread/)

**공용객체**는 여러 Thread가 공용으로 사용하는 객체를 말한다.
여러 Controller가 하나의 Service를 사용할 수 있으며, 이러한 service는 공용객체

### Gradle, Maven이 무엇인가 - Build Tool (CoC)
