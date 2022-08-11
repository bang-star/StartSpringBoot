# 5.5 Thymeleaf 링크 처리

  - 일반적인 웹 페이지의 링크는 두 가지로 구분된다.

     - 절대 경로(absolute path) : http://www.~

     - 상대 경로(context-relative path) : 현재 URL을 기준으로 이동

  - 스프링 부트에서는 프로젝트를 실행하면 '/'를 기준으로 동작하기 때문에 경로에 대한 스트레스 없이 작성할 수 있지만, WAS 상에서는 특정 경로에서 프로젝트가 실행되는 경우에 문제가 될 수 있다. 

    - Thymeleaf는 이러한 문제를 해결하기 위해서 '@{}'를 이용해 경로에 대한 처리를 자동으로 처리할 수 있다.

    ```Java
    @GetMapping("/sample8")
    public void sample(Model model){
        
    }
    ```

    ```HTML
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8" http-equiv="Content-Type" >
        <title>Thymeleaf3</title>
    </head>
    <body>
        <ul>
            <li><a th:href="@{http://localhost:8080/sample1}">sample1</a></li>
            <li><a th:href="@{/sample1}">sample1</a></li>
            <li><a th:href="@{~/sample1}">sample1</a></li>
        </ul>
    </body>
    </html>
    ```

    - '@{/sample1}'에는 현재 실행되는 컨텍스트의 경로가 반영된다.

        - 컨텍스트의 경로가 '/'라면 '/sample1'과 같은 경로가 되지만, 컨텍스트의 경로가 'boot05'와 같이 된다면 'boot05/sample1'과 같은 경로가 된다.

        - '@{~/sample1}'의 경우는 현재 프로젝트가 '/'경로에서 실행되었기 때문에 '@{/sample1}'과 차이가 없게 된다.

     - Thymeleaf의 링크 처리에서 특이한 부분은 파라미터를 전달하는 경우이다.

     - 일반적으로 직접 링크를 생성하기 위해서 '파라미터의 이름=값'의 형태로 작성하지만, Thymeleaf를 이용하면 (이름(키)=값)의 형태로 링크를 생성할 수 있다.

        ```HTML
        <li><a th:href="@{/sample1(p1='aaa', p2='bbb')}">sample1</a></li>
        ```

        - Thymeleaf에서는 이와 같이 '키'와 '값'의 형태로 사용하면 문자열을 생성한다.

            ```HTML
            <li><a href="/sample1?p1=aaa&amp;p2=bbb"></a></li>
            ```