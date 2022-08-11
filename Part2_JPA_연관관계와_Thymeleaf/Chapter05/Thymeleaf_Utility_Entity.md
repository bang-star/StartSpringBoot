# 5.4 Thymeleaf의 유틸리티 객체

 - Thymeleaf는 기본적으로 표현식을 통해 데이터를 출력하는 역할이 중심이지만, 이를 돕기 위한 다양한 객체들을 지원합니다.

  - Expression Basic Objects(표현식 기본 객체)

     - #ctx

     - #vars

     - #locale

     - #httpServletRequest

     - #httpSession

  - Expression Utility Objects(표현식 유틸 객체)

     - #dates

     - #calendars

     - #numbers

     - #strings

     - #objects

     - #bools

     - #arrays

     - #lists

     - #sets

     - #maps

     - #aggregates

     - #messages

  - 표현식 기본 객체는 기존에 JSP에서 application이나 request, session 등을 사용할 때 변수가 된다.

     - 예를 들어, #vars의 경우 생략한 상태로 주로 사용된다.

        ```HTML
        # 동일한 의미
        <div>[[${result}]]</div>
        <div>[[${#vars.result}]]</div>
        ```

## 5.4.1 유틸리티 객체

  - Thymeleaf의 표현식은 OGNL(Object-Graph Navigation Language) 표현식을 이용해서 데이터를 추력하게 된다.

  - OGNL은 다양한 프레임워크와 언어에서 사용하지만, Java만을 이용할 때는 불편한 점들이 있다.

    - (예시) 문자열을 비교할 때 Java에서는 equals()를 이용하지만 OGNL에서는 'eq'와 같이 조금 다른 방식을 이용해서 개발에 불편할 수 있다.

  - 유틸리티 객체는 기존처럼 메소드를 호출하는 방식으로 사용할 수 있는 객체들이다.

    - (예시) 날짜를 포맷팅(formatting)하기 위해서는 다음과 같은 방식으로 사용한다.

    <br />

    ```HTML
    ${#dates.format(member.regdate, 'yyyy-MM-dd')}
    ```

    ```Java
    @GetMapping("/sample7")
    public void sample7(Model model){

        model.addAttribute("now", new Date());
        model.addAttribute("price", 123456789);
        model.addAttribute("title", "This is a just sample.");
        model.addAttribute("options", Arrays.asList("AAAA", "BBB", "CCC", "DDD"));
    }
    ```

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <h1 th:text="${now}"></h1>

        <h1 th:text="${price}"></h1>

        <h1 th:text="${title}"></h1>

        <h1 th:text="${options}"></h1>

    </body>
    </html>
    ```

### 날짜 관련 #dates, #calendars

  - 날짜 관련 기능은 java.util.Date와 java.util.Calender의 기능을 이용한다.

    ```HTML
    <h2 th:text="${#dates.format(now, 'yyyy-MM-dd')}"></h2>
        <div th:with="timeValue=${#dates.createToday()}">
            <p>[[${timeValue}]]</p>
        </div>
    ```

### 숫자 관련 #numbers 

  - Integer나 Double, Float에 대한 포맷팅 처리할 때 주로 사용된다.

  ```HTML
  <h2 th:text="${#numbers.formatInteger(price, 3, 'COMMA')}"></h2>
    <div th:with="priceValue=99.87654">
        <p th:text="${#numbers.formatInteger(priceValue, 3, 'COMMA')}"></p>
        <p th:text="${#numbers.formatDecimal(priceValue, 5, 10, 'POINT')}"></p>
    </div>
  ```

### 문자 관련 #strings

  - 문자열 관련해서는 대소문자 변환이나 contains() 등 기본적인 기능들 외에 문자열을 결합하는 join이나 리스트로 나누는 listsplit 등의 기능들이 지원된다.

  ```HTML
  <h1 th:text="${title}"><h1>
  <span th:utext="${#strings.replace(title, 's', '<b>s<b>')}"></span>
  <ul>
    <li th:each="str:${#strings.listSplit(title, ' ')}">[[${str}]]</li>
  </ul>
  ```