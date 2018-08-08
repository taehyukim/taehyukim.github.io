# 3회차

## 수업내용




## 과제

### Builder Pattern과 Lombok을 사용한 Effective Java 의 Builder 패턴 구현
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
    default int multiple(int i, int j) {    // default로 선언하여 method를 구현할 수 있다.
        return i * j;
    }
}
```
Default method를 이용하면 interface에 새로운 method를 추가해도, 기존에 해당 interface를 구현하던 class의 수정 없이 호환성을 유지하면서 라이브러리를 변경할 수 있다.

### Java Web Programming의 4가지 Scope
Java Web Programming에는 4가지 객체 범위(Scope)가 존재한다.
1. **Page Scope**
    - 하나의 JSP가 호출될 때 해당 JSP 내에서만 객체를 공유하는 범위를 의미한다.
    - JSP 파일에는 pageContext 객체가 내장되어 있으며, 이 객체는 page 영역에서만 유효하다.
    - JSP 파일의 <% %> 안에 변수를 사용하면 그 변수는 해당 JSP 내에서만 유효하다.
2. **Request Scope**
    - Request를 받아서 그에 대한 response를 보내기 까지 객체가 유효한 범위를 의미한다.
    - Forward또는 include를 이용하는 경우 여러개의 page에서도 하나의 request가 유지되므로, request 범위의 객체를 여러 페이지에서 공유할 수 있다.
3. **Session Scope**
    - Session scope 설명
4. **Application Scope**
    - Application scope 설명