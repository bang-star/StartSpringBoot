# 5.2 Thymeleaf 동작 확인

## 5.2.1 Controller

  - JSP 대신에 Thymeleaf로 된 템플릿을 이용하더라도 Web MVC의 구성 자체가 달라지는 것은 아니기 때문에 우선 컨트롤러를 구성해 주는 것으로 시작한다.

    ```Java
    @Controller
    public class SampleController {
        
        @GetMapping("/sample1")
        public void sample1(Model model){
            model.addAttribute("greeting", "Hello World");
        }   
    }
    ```

  - @GetMapping을 이용하면 @RequestMapping을 이용하는 것보다 간단하게 get 방식의 설정을 지정할 수 있따.

## 5.2.2 템플릿 페이지 작성

  ```HTML
  <!--thymeleaf 사용 명시-->
    <html xmlns:th="http://www.thymeleaf.org">
    <html lang="en">
    <head>
        <meta charset="UTF-8" content="text/html" http-equiv="Content-Type">
        <title>Title</title>
    </head>
    <body>
        <h1>Thymeleaf Test Page</h1>
    </body>
    </html>
  ```

### 페이지 수정하기

 - 스프링 부트를 이용해서 웹을 개발할 때에는 'DevTools'를 포함한 상태에서 개발하는 것을 권장한다.

    - 'DevTools'는 컨트롤러의 소스 코드를 수정하면 자동으로 스프링 부트를 재시작해 주기 때문이다.

    ```HTML
    <!--thymeleaf 사용 명시-->
    <html xmlns:th="http://www.thymeleaf.org">
    <html lang="en">
    <head>
        <meta charset="UTF-8" content="text/html" http-equiv="Content-Type">
        <title>Thymeleaf</title>
    </head>
    <body>
        <h1 th:text="${greeting}">Thymeleaf Test Page</h1>
    </body>
    </html>
    ```

    - Thymeleaf의 장점 중 하나는 템플릿이 기본적으로 출력할 데이터가 없는 상황에서 HTML로 작성된 내용이 그대로 반영된다는 점이다.


  