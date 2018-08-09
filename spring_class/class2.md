# 3회차

## 수업내용
- HttpServletRequest, httpServletResponse
- DispatcherServlet 과 Controller
- Servlet, default servlet

실습
- @RestController, @GetMapping
- @PathVariable, @RequestParam, @ModelAttribute

**@PathVariable**
- /api/boards/1 같은 요청을 처리
```java
@RestController
@RequestMapping("/api/boards")
public class BoardAPIController {
    @GetMapping
    public String boards() {
        return "boards";
    }

    @GetMapping("/{id}")
    public String boards(@PathVariable(name = "id")int id) {
        return "board - " + id;
    }
}
```

**@RequestParam**
- /hello2?name=kim 같은 요청을 처리
```java
@RestController
public class HelloController {
    public HelloController(){
        System.out.println("HelloController!");
    }

    @GetMapping("/hello")
    public String SayHello(){
        return "<h2>hello<h2>";
    }

    @GetMapping("/hello2")
    public String SayHello2(@RequestParam(name = "name", required = true)String name,
                            @RequestParam(name = "age", required = false, defaultValue = "5")int age){
        return "<h2>hello " + name + "(" + age + ")<h2>";
    }
}
```

**@ModelAttribute**
- type에 맞는 parameter가 전달되면 각 변수에 자동으로 값을 할당하여 받아들인다.

## 과제
