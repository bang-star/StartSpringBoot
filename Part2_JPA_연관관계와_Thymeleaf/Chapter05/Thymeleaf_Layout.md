# 5.6 Thymeleaf의 레이아웃 기능

  - Thymeleaf를 이용하면 기본적으로 제공하는 th:insert(Thymeleaf 2번에서는 include였으나 3 버전에서는 deprecated가 되었다.), th:replace, th:include와 같은 속성3들을 이용해서 기존 페이지의 일부분을 다른 내용을 쉽게 변경할 수 있다.

  - Thymeleaf는 부분적인 화면의 처리 기능과 화면 전체의 레이아웃을 지정하고, 페이지를 작성할 떄 필요한 부분만을 교체해서 사용하는 템플릿 기능이 지원된다.

## 5.6.1 th:insert를 이용해서 페이지의 헤더와 푸터 처리하기 

   ```HTML
   // header.html
   <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    <div th:fragment="header">
        헤더입니다.
    </div>
    </body>
   </html>
   ```

   ```HTML
   // footer.html
   <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <div th:fragment="footer">
            Footer 파일입니다.,
        </div>
    </body>
   </html>
   ```

   ```HTML
   // sample8.html 수정
   <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8" http-equiv="Content-Type" content="text/html">
        <title>Thymeleaf3</title>
    </head>
    <body>
        <div th:insert="~{fragements/header::header}"></div>

        <ul>
            <li><a th:href="@{http://localhost:8080/sample1}">sample1</a></li>
            <li><a th:href="@{/sample1}">sample1</a></li>
            <li><a th:href="@{~/sample1}">sample1</a></li>
        </ul>

        <div th:insert="~{fragements/footer::footer}"></div>
    </body>
    </html>
   ```

   - header.html의 header와 footer.html의 foot라는 조각(fragement)이 포함되어 있는 것을 볼 수 있다.

## 5.6.2 Thymeleaf layout dialect를 이용한 레이아웃 재사용하기

  - Thmyleaf layout dialect

    - Thymeleaf에 기본적으로 포함된 라이브러리는 아니다.
    
    - 하나의 레이아웃을 작성하고 이를 재사용해서 여러 페이지에 동일한 레이아웃을 적용시킬 수 있다. (즉, 템플릿 상속이라 부를 수 있다.)

  - 템플릿 상속

    - 템플릿 상속을 이용하면 전체의 레이아웃을 구성한 후 레이아웃 내에서 원하는 위치에 필요한 페이지들을 끼워 넣거나, 변경하는 작업이 가능해진다.

    - 개발 시에 현재 본인이 작업하는 페이지의 내용만을 처리할 수 있다.

    ```XML
    <!-- https://mvnrepository.com/artifact/nz.net.ultraq.thymeleaf/thymeleaf-layout-dialect -->
    <dependency>
        <groupId>nz.net.ultraq.thymeleaf</groupId>
        <artifactId>thymeleaf-layout-dialect</artifactId>
        <version>2.2.1</version>
    </dependency>
    ```

  - 예제에서 레이아웃 상속 기능을 적용해 보기 위해서 HTML5 Boilerplater를 적용할 것이다.

    - HTML5 Boilerplater는 웹페이지에서 가장 기본적으로 필요한 구조를 템플릿으로 만들어 둔 것이다.

    - (예를 들어) 기본적인 js, image 폴더들의 구조나 favicon, ico 파일, index.html 파일 등을 묶음으로 만들어 둔 다음, 이를 이용해서 전체 구조를 잡고 페이지를 추가하는 방식으로 사용할 수 있다.

    - Boilerplate에서 다운 받은 파일을 static 폴더로 이동시킨다.

    - 레이아웃을 적용하기 위해서 templates 내에 layout 폴더를 작성하고, layout1.html 파일을 작성한다.

    - 변경된 layout1.html에서 매번 페이지 제작 시 변경되는 영역은 'content-body'라고 표현된 부분과 'custom javascript'라고 표현된 영역이다.

### th:block과 layout:fragment

  - layout1.html에서 매번 변경되는 영역은 화면의 Thymeleaf 코드를 사용하는 'content-body'영역과 매 페이지에서 필요한 Javascript 코드를 넣어주는 'custom javascript'영역이다.

     - layout:fragement는 div 와 같이 실제로 보이는 영역에 적용해 주고, th:block은 아무런 태그가 없는 영역을 표시할 때 사용한다.

     ```HTML
     <body>
        <!--  content body  -->
        <div layout:fragment="content">

        </div>

        <!-- Add your site or application content here -->
        <p>Hello world! This is HTML5 Boilerplate.</p>

        <script src="https://code.jquery.com/jquery-1.12.0.min.js"></script>
        ...
        <script src="js/main.js"></script>

        <!-- custom javascript -->
        <th:block layout:fragment="script"></th:block>

        <!-- Google Analytics: change UA-XXXXX-X to be your site's ID. -->
        ...
    </body>
     ```

### 페이지에 레이아웃 적용하기

   ```Java
    @GetMapping("/sample/hello")
    public void hello(){
        
    }
   ```

   - 레이아웃을 적용하려면 네임스페이스를 추가해 주어야 한다.

        ```HTML
        <html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{/layout/layout1}">
        ```

  - hello.html에서 layout의 layout1.html을 이용하도록 설정되었고, layout1.html에서 필요한 부분의 코드를 추가해야 한다.

    
        ```HTML
        <html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{/layout/layout1}">

         <div layout:fragment="content">
            <h1>Hello Page</h1>
         </div>
         <th:block layout:fragment="script">
            <script th:inline="javascript">
                alert("Java Script Block!!");
            </script>
        ```

     - 이를 통해 브라우저에서 호출하면 레이아웃이 적용되면서 다른 리소스들을 호출하는 것을 볼 수 있다.

     - 현재 경로가 '/sample'로 시작했기 때문에 최종적으로 layout1.html에서 링크의 경로를 수정해 주어야 한다. '@{/}'의 경로를 이용해서 수정해주어야 한다.

     ```HTML
     // layout 폴더 내에 layout1.html
     <!doctype html>
    <html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="x-ua-compatible" content="ie=edge">
        <title>Thymeleaf3</title>
        <meta name="description" content="">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <link rel="apple-touch-icon" href="apple-touch-icon.png">
        <!-- Place favicon.ico in the root directory -->

        <link rel="stylesheet" th:href="@{/css/normalize.css}">
        <link rel="stylesheet" th:href="@{/css/main.css}">

        <script th:src="@{/js/vendor/modernizr-2.8.3.min.js}"></script>
    </head>
    <body>
        <!--  content body  -->
        <div layout:fragment="content">

        </div>

        <!-- Add your site or application content here -->
        <p>Hello world! This is HTML5 Boilerplate.</p>

        <script src="https://code.jquery.com/jquery-1.12.0.min.js"></script>
        <script>window.jQuery || document.write('<script th:src="@{/js/vendor/jquery-1.12.0.min.js}"><\/script>')</script>
        <script th:src="@{/js/plugins.js}"></script>
        <script th:src="@{/js/main.js}"></script>

        <!-- custom javascript -->
        <th:block layout:fragment="script"></th:block>

        <!-- Google Analytics: change UA-XXXXX-X to be your site's ID. -->
        <script>
            (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
                function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
                e=o.createElement(i);r=o.getElementsByTagName(i)[0];
                e.src='https://www.google-analytics.com/analytics.js';
                r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
            ga('create','UA-XXXXX-X','auto');ga('send','pageview');
        </script>
    </body>
    </html>
     ```

  - 브라우저에서 'sample/hello'를 호출하면 작성된 페이지와 레이아웃, 리소스들이 모두 적용된 페이지를 볼 수 있게 된다.

  
    <img src="https://user-images.githubusercontent.com/63120360/184072183-ac525739-a7c6-4697-85b6-cc6ac797adb7.png">

    <br />

  - 

## 참고

  - [Boilerplate](https://html5boilerplate.com/)

    
  