# 8.1 프로젝트 생성

  - 프로젝트 목적

    1. 사용자와 사용자의 권한을 관리하는 기능

     2. 해당 기능을 이용해서 스프링 시큐리티를 적용

  - 라이브러리

    - Spring Security

    - DevTools

    - Lombok

    - JPA

    - MyBatis

    - MySQL

    - Thymeleaf

    - Spring Web

  - application.properties

    ```xml
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/jpa_ex?useSSL=false
    spring.datasource.username=jpa_user
    spring.datasource.password=jpa_user

    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.generate-ddl=true
    spring.jpa.show-sql=true
    spring.jpa.database=mysql
    spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

    logging.level.org.hibernate=info

    spring.thymeleaf.cache=false

    logging.level.org.springframework.web=debug
    logging.level.org.springframework.security=debug
    ```

<br />
<hr />
<br />

## 8.1.1 시큐리티의 기본 설정 추가

 - SecurityConfig 클래스

    - Spring Bean 등록을 위해 @EnableWebSecurity 어노테이션 추가

    - 설정을 담당하는 WebSecurityConfigurerAdapter 클래스 상속

    ```Java
    @Log
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {}
    ```

  - SecurityConfig클래스에 WebSecurityConfigurerAdapter클래스의 메소드 중에 configure() 메소드를 오버라이드해서 간단한 로그 메시지를 출력하도록 작성한다.

    ```Java
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");
    }
    ```

     - configure() 메소드를 오바라이드하는 경우네느 HttpSecurity 클래스 타입을 파라미터로 처리하는 메소드를 선택해야 한다는 점을 주의해야한다.

     - 어노테이션을 추가한 후 SecurityConfig가 Spring Bean으로 등록되었는지 확인하고, 프로젝트를 실행해서 log가 적용되는지를 확인해야한다.

     <img src="https://user-images.githubusercontent.com/63120360/186097512-effba06b-5f3b-4d00-b054-90e330ce3251.png">
      
     - Spring Security는 기본적으로 하나의 사용자 정보를 가지도록 기본 세팅되어 있다. 사용자 이름은 'user', 패스워드는 로그에서 출력되는 정보이다.


<br />

### NOTE
```
만일 SecurityConfig를 생성하지 않고, 컨트롤러와 Thymeleaf를 이용하는 페이지를 제작했다면 Spring Security는 모든 URI에 대한 인증을 요구한다.

이런 경우네는 사용자 이름을 'user'로 지정하고, 서버 실행 시 생성된 패스워드를 입력해 주면 정상적으로 처리된다.

SecurityConfig를 생성하고 config()를 작성했다면 이와 같은 기본 설정은 더 이상 사용하지 않는다.
```

## 8.1.2 샘플 URI 생성

 - SampleController

    ```Java
    @Controller
    @Log
    public class SampleController {

        @GetMapping("/")
        public String index(){
            log.info("index");
            return "index";
        }

        @RequestMapping("/guest")
        public void forGuest(){
            log.info("guest");
        }

        @RequestMapping("/manager")
        public void forManager(){
            log.info("manager");
        }

        @RequestMapping("/admin")
        public void forAdmin(){
            log.info("admin");
        }
    }
    ```

  - templates에 URI에 맞는 파일들을 생성한다. 화면 자체는 스프링 시큐리티가 제대로 적용되고 있는지를 확인하는 용도이다.

    ```HTML
    <!-- index.html -->

    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <h1>Index Page for EveryOne!!</h1>
    </body>
    </html>
    ```

 