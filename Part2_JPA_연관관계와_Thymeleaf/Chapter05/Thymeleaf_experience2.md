# 5.3 Thymeleaf 간단한 예제

  - 예제

    1. 객체를 화면에 출력하기

    2. 리스트를 화면에 출력하기

    3. 변수나 제어문 처리

    4.  request의 파라미터들을 사용하기

    5. 레이아웃 처리

<br />

## 5.3.1 객체를 화면에 출력하기

  - 가장 간단한 객체 출력은 JSP에서 EL을 이용해서 출력하는 것과 거의 동일하다.

    ```Java
    @Data
    @AllArgsConstructor
    public class MemberVO {
        
        private int mno;
        private String mid;
        private String mpw;
        private String mname;
        private Timestamp regdate;
    }
    ```

    ```Java
    @GetMapping("/sample2")
    public void sample2(Model model){
        MemberVO vo = new MemberVO(123, "u00", "p00","홍길동", new Timestamp(System.currentTimeMillis()));

        model.addAttribute("vo", vo);
    }
    ```

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <h1 th:text="${vo}">Thymeleaf Test Page</h1>
    </body>
    </html>
    ```
    
    - th:text는 태그의 내용물로 문자열을 출력한다.

### HTML 출력하기 - utext

  - th:text와 달리 th:utext는 문자열이 아니라 HTML 자체를 출력하는데 사용할 수 있다.

  ```HTML
  <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <h1 th:text="${vo}">Thymeleaf Test Page</h1>

        <div th:utext='${"<h3>"+vo.mid+"</h3>"}'></div>
        <div th:text='${"<h3>"+vo.mid+"</h3>"}'></div>
    </body>
  </html>
  ```

## 5.3.2 리스트를 화면에 출력하기

  - 화면에서 가장 많이 사용하는 루프의 처리는 th:each를 이용해서 처리할 수 있다.

  - th:each는 리스트나 java.util.Iterable, java.util.Map, 배열 등을 사용할 수 있다.

  - th:each에 사용하는 표현식은 th:each="var : ${list}"와 같은 방식으로 작성한다.

  ```HTML
  <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <table border="1">
            <tr>
                <td>MID</td>
                <td>MNAME</td>
                <td>REGDATE</td>
            </tr>
            <tr th:each="member : ${list}">
                <td th:text="${member.mid}"></td>
                <td th:text="${member.mname}"></td>
                <td th:text="${#dates.format(member.regdate, 'yyyy-MM-dd')}"></td>
            </tr>

        </table>
    </body>
    </html>
  ```

   - 마지막에 사용한 '#dates.format()'은 Thymeleaf의 보조 객체로, 날짜와 관련된 처리에 유용하게 사용된다.

   - th:each에는 반복된 상태에 대한 변수를 지정하여, 필요한 추가 정보들을 추출할 수 있다.

   - th:each에 현재 상태에 대한 변수를 선언하면 다음과 같은 항목들을 사용할 수 있다.

     - index
         
         0 부터 시작하는 인덱스 번호

     - count

         1부터 시작 하는 번호

     - size

         현재 대상의 length 혹은 size

     - odd/even

         현재 번호의 홀수/짝수 여부

     - first/last

         처음 요소인지 마지막 요소인지를 판단

     ```HTML
        <html xmlns:th="http://www.thymeleaf.org">
        <head>
            <title>Thymeleaf3</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
        </head>
        <body>
            <table border="1">
                <tr>
                    <td>MID</td>
                    <td>MNAME</td>
                    <td>REGDATE</td>
                </tr>
                <tr th:each="member : ${list}">
                    <td th:text="${member.mid}"></td>
                    <td th:text="${member.mname}"></td>
                    <td th:text="${#dates.format(member.regdate, 'yyyy-MM-dd')}"></td>
                </tr>

            </table>
        </body>
        </html>
     ```

### 5.3.3 지역변수의 선언, if ~ unless 제어 처리

 - Thymeleaf에서 특정한 범위에서만 유횸한 지역변수를 th:with를 이용해서 선언할 수 있다.
  
  ```Java
    @GetMapping("/sample4")
    public void sample4(Model model){
        List<MemberVO> list = new ArrayList<>();

        for(int i=0; i<10; i++){
            list.add(new MemberVO(
                    i,
                    "u000"+ i% 3,
                    "p000"+ i% 3,
                    "홍길동" + i,
                    new Timestamp(System.currentTimeMillis())
            ));
        }

        model.addAttribute("list", list);
    }
  ```
  ```HTML
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <table border="1" th:with="target='u0001'">
            <tr>
                <td>MID</td>
                <td>MNAME</td>
                <td>REGDATE</td>
            </tr>
            <tr th:each="member : ${list}">
                <td th:text="${member.mid}"></td>
                <td th:text="${member.mname}"></td>
                <td th:text="${#dates.format(member.regdate, 'yyyy-MM-dd')}"></td>
            </tr>
        </table>
    </body>
    </html>
  ```

  - th:with를 이용한 target 변수는 table 태그 내에서만 유효한 지역변수이다.

  - 삼항 연산자 사용 예) 'u0001'인 사용자만 다른 내용으로 출력하고 싶을 때

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <table border="1" th:with="target='u0001'">
            <tr>
                <td>MID</td>
                <td>MNAME</td>
                <td>REGDATE</td>
            </tr>
            <tr th:each="member : ${list}">
                <td th:text="${member.mid == target ? 'SECRET': member.mid}"></td>
                <td th:text="${member.mname}"></td>
                <td th:text="${#dates.format(member.regdate, 'yyyy-MM-dd')}"></td>
            </tr>
        </table>
    </body>
    </html>
    ```

    <img src="https://user-images.githubusercontent.com/63120360/183823194-da3498e4-f46d-44a9-9272-768f41b4209e.png">

    <br />

    - 삼항 연산자를 이용하는 방식이 간편하기는 하지만 경우에 따라서는 if ~ else 처리가 필요한 상황이 많다.

    - th:if ~ unless를 이용해서 if ~ else를 처리할 수 있다.

        - th:if는 내부 식(expression)의 결과가 true인 경우에만 th:if를 포함하는 태그 자체가 생성된다.

        <br />

        ```HTML
        <html xmlns:th="http://www.thymeleaf.org">
        <head>
            <title>Thymeleaf3</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
        </head>
        <body>

            <table border="1" th:with="target='u0001'">
                <tr>
                    <td>MID</td>
                    <td>MNAME</td>
                    <td>REGDATE</td>
                </tr>

                <tr th:each="member : ${list}">
                    <td th:if="${member.mid}">
                        <a href="/modify" th:if="${member.mid == target}">MODIFY</a>
                        <p th:unless="${member.mid == target}">VIEW</p>
                    </td>
                    <td th:text="${member.mname}"></td>
                    <td th:text="${#dates.format(member.regdate, 'yyyy-MM-dd')}"></td>
                </tr>
            </table>

        </body>
        </html>
        ```

        - member.mid가 target과 같은 경우에는 a 태그를 출력하고, 아닌 경우 p태그를 출력하게 된다.

        <br />
        
        <img src="https://user-images.githubusercontent.com/63120360/183823983-2d7c89e9-df2d-4e3d-ae5c-428327535d0d.png">

        <br />

### 5.3.4 인라인 스타일로 Thymeleaf 사용하기

  - JSP 페이지를 작성하다 보면 가끔 EL을 이용해서 자바스크립트 코드를 작성하는 경우가 종종 있다.

    - th:inline 이라는 것을 이용해서 'javascript'나 'text'를 지정해서 사용하는 것이다.

    - (예시) 화면에 result라는 결과를 문자열로 전달하는 경우

    ```Java
    @GetMapping("/sample5")
    public void sample5(Model model){
        String result = "SUCCESS";
        model.addAttribute("result", result);
    }
    ```

    - th:inline을 이용해 script 태그 내에 Thymeleaf를 사용한다는 것을 명시한다.

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" >
    </head>
    <body>
        <script th:inline="javascript">

            var result = [[${result}]];

        </script>

        <script>
            var result = [[${result}]];
        </script>
    </body>
    </html>
    ```

     <img src="https://user-images.githubusercontent.com/63120360/183839155-9a839d79-787b-436f-87db-811b89d20b34.png">

    <br />

     - th:inline를 지정한 코드에서는 자바스크립트 문자열로 제대로 생성되지만, 그렇지 않은 경우에는 변수처럼 선언되는 것을 볼 수 있다.

     - Thymeleaf의  inlining은 기본은 'text'로 지정되므로, 일반 태그를 작성할 때도 사용할 수 있다.

    ```Java
    @GetMapping("/sample6")
    public void sample6(Model model){

        List<MemberVO> list = new ArrayList<>();

        for(int i=0; i<10; i++){
            list.add(new MemberVO(i, "u0"+i, "p0"+i, "홍길동"+i, new Timestamp(System.currentTimeMillis())));
        }

        model.addAttribute("list", list);

        String result = "SUCCESS";

        model.addAttribute("result", result);
    }
    ```

    <img src="https://user-images.githubusercontent.com/63120360/183841563-41a7ecc6-1622-4f01-9bc3-6ce024ffa580.png">

