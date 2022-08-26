# 8.3 단순 시큐리티 적용

  - 웹에서의 Spring Security는 기본적으로 필터 기반으로 동작한다.

    <img src="https://github.com/spring-guides/top-spring-security-architecture/raw/main/images/security-filters.png">

    - Spring Security의 내부에는 상당히 많은 종류의 필터들이 이미 존재하고 있기 때문에, 개발 시에는 필터들의 설정을 조정하는 방식을 주로 사용하게 된다.

<br />

## 8.3.1 로그인/로그아웃 관련 처리

### 특정 권한을 가진 사람만이 특정 URI에 접근하기

  - SecurityConfig 클래스에는 configure() 메소드를 이용해서 웹 자원에 대한 보안을 확인한다. 현재까지의 configure()는 HttpSecurity 타입을 파라미터로 사용한다.

    ```Java
    // SecurityConfig.java
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");
    }
    ```

  - HttpSecurity는 웹과 관련된 다양한 보안 설정을 걸어줄 수 있다.

  - 특정한 경로에 특정한 권한을 가진 사용자만 접근할 수 있도록 설정하고 싶을 때에는 authorizeRequests()와 antMatches()를 이용해서 경로를 설정할 수 있다.

    ```Java
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");
        
        http.authorizeRequests().antMatchers("/guest/**").permitAll();
        http.authorizeRequests().antMatchers("/manager/**").hasRole("MANAGER");
    }
    ```

    - authorizeRequest()는 시큐리티 처리에 HttpServletRequests를 이용한다는 것을 의미한다.

    - antMatches()에서는 특정한 경로를 지정한다.

    - permitAll()은 모든 사용자가 접근할 수 있다는 것을 의미

    - hasRole()은 시스템상에서 특정 권한을 가진 사람만이 접근할 수 있다는 것을 의미

    - http.authorizeRequests() 뒤의 antMatches()는 사실 빌더 패턴으로 연속적으로 '.'를 이용하는 방법을 더 많이 사용한다.

        ```Java
        http
            .authorizeRequests()
                .antMatchers("/high_level_url_A/sub_level_1").
                hasRole('USER1')
                .antMatchers("/high_level_url_A/sub_level_2").hasRole('USER2')
                .somethingElse()    // for /high_level_url_A/**
                .antMatchers("/high_level_url_A/**").authenticated()
                .antMatchers("/high_level_url_B/sub_level_1").permitAll()
                .antMatchers("/high_level_url_B/sub_level_2").hasRole('USER3')
                .somethingElse()    // for /high_level_url_B/**
                .antMatchers("/high_level_url_B/**").authenticated()
                .anyRequest().permitAll()
        ```

### NOTE

    Role과 Privilege

    사전적인 의미에서 Role은 '역할'이나 '자격'을 의미하고, Privilege는 '특권'을 의미한다. 스프링 시큐리티에서는 Role과 Privilege 외에도 Authority라는 용어를 같이 사용하기 때문에 혼란스러울 수 있다. 

    Role은 일종의 Privilege들의 묶음으로 이해하면 된다. 세부적인 권한을 Privilege나 Authority로 보고, 이를 모아서 하나의 패키지나 템플릿으로 만든 것을 Role이라고 생각할 수 있다.

<br />

<img src="https://user-images.githubusercontent.com/63120360/186452883-38975969-fcd6-4b1b-a229-608fe709632f.png">

<br />

### 로그인 페이지 보여주기

  - 인가 받은 권한이 없는 관계로 접근이 막혔다면 '/login'이라는 경로를 이용해서 로그인 페이지를 띄워, 로그인을 해야 접근할 수 있다는 것을 인식시킬 필요가 있다.

    ```Java
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");

        http.authorizeRequests().antMatchers("/guest/**").permitAll();
        http.authorizeRequests().antMatchers("/manager/**").hasRole("MANAGER");
        
        http.formLogin();
    }
    ```

    - formLogin은 form 태그 기반의 로그인을 지원하겠다느 설정이다. 이를 이용하면 별도의 로그인 페이지를 제작하지 않아도 로그인 정보를 입력하는 페이지를 볼 수 있게 된다.

<br />

### 로그인 정보 설정

  - 화면에 로그인 페이지가 뜨기는 하지만, 어떤 값을 입력하여도 에러가 발생하면서 달라지는 내용은 없다.

    <img src="https://user-images.githubusercontent.com/63120360/186454175-38ab8509-2679-493e-94e5-d4eafbd64137.png">

  - 로그인이 되기 위해서는 SecurityConfig에 AuthenticationManagerBuilder를 주입해서 인증에 대한 처리를 해야 한다.

    ```Java
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception{
        log.info("build Auth global ...... ");
        
        auth.inMemoryAuthentication()
                .withUser("manager")
                .password("1111")
                .roles("MANAGER");
    }
    ```

    - AuthenticationManagerBuilder는 단어의 뜻 그대로 인증에 대한 다양항 설정을 생성할 수 있다. 예를 들어, 메모리상에 있는 정보만을 이용한다거나, JDBC나 LDAP 등의 정보를 이용해서 인증 처리가 가능하다.

<br />

### 로그인 관련 정보 삭제하기

  - 웹과 관련된 로그인 정보는 기본적으로 HttpSession을 이용한다.

  - HttpSession은 세션 쿠키라는 것을 이용하기 때문에 기존의 로그인 정보를 삭제해야 하는 상황에서는 브라우저를 완전 종료하거나 세션 쿠키를 삭제하는 방법을 이용한다.

<br />

### 커스텀 로그인 페이지 만들기

  - 스프링 시큐리티가 기본적으로 태그가 있는 웹 페이지를 만들어 주기는 하지만 좀 더 제대로 만들기 위해서 사용자가 수정할 수 있는 웹 페이지를 만들어주는 것이 좋다.
  
  - formLogin() 이후에 loginPage()메소드를 이용해서 URI를 지정

    ```Java
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");

        http.authorizeRequests().antMatchers("/guest/**").permitAll();
        http.authorizeRequests().antMatchers("/manager/**").hasRole("MANAGER");

        http.formLogin().loginPage("/login");
    }
    ```

  - 'login'이라는 URI는 더 이상 스프링 시큐리티가 이용하는 것이 아니기 때문에, 직접 컨트롤러를 작성해야 한다.

    ```Java
    @Controller
    public class LoginController {
        
        @GetMapping("/login")
        public void login(){
            
        }
    }
    ```

    ```HTML
    <!-- login.html -->
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf3</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <h2>CUSTOM LOGIN PAGE</h2>
        <div th:if="${param.error != null}">
            <h2>Invalid Username or password</h2>
            <h2 th:value="${param.error}"></h2>
        </div>

        <div th:if="${param.logout != null}">
            <h2>You have been logged out.</h2>
        </div>
        
        <form method="post">
            <p>
                <label for="username">Username</label>
                <input type="text" id="username" name="username" value="user88">
            </p>
            <p>
                <label for="password">Password</label>
                <input type="password" id="password" name="password" value="pw88">
            </p>
            <input type="hidden" th:name="${_csrf.parameterName}" th:value="_csrf.token" />
            <button type="submit" class="btn">Log in</button>
        </form>

    </body>
    </html>     
    ```

      - Spring Security는 기본적으로 username과 password라는 이름을 이용하므로, input 태그의 name 속성값을 마음대로 변경할 수 없다.

      - form 태그의 action 속성을 지정하지 않았기 때문에 사용자가 버튼을 클릭하면 '/login'으로 이동하고, POST 방식으로 데이터를 전송하게 된다.

      - form 태그의 내부에는 hidden 속성으로 작성된 '_csrf'라는 속성이 존재한다. 이 속성은 '사이트 간 요청 위조(Cross-site request forgery, CSRF, XSRF)'를 방지하기 위한 것으로, 요청을 보내는 URL에서 서버가 가진 동일한 값과 같은 값을 가지고 데이터를 전송할 때에만 신뢰하기 위한 방법이다.


     - 실제로 모든 작업은 여러 종류의 Filter들과 Interceptor를 통해서 동작하기 때문에, 개발자의 입장에서는 적절한 처리를 담당하는 핸들러(Handler)들을 추가하는 것만으로 모든 처리가 완료된다.

     - 스프링 시큐리티가 적용되면 POST 방식으로 보내는 모든 데이터는 CSRF 토큰 값이 필요하다는 점을 명시해야 한다(만일 CSRF 토큰을 사용하지 않으려면 application.properties에 security.enable-csrf속성을 이용해서 CSRF 토큰을 사용하지 않도록 설정할 수 있다.)

<br />

### 접근 권한 없음 페이지 처리

  - 만일 사용자가 정상적으로 로그인을 하고 접근할 수 있는 권한을 가진 경로를 이용한다면 문제가 없다. 하지만, 로그인한 사용자라고 하더라도, 접근 권한이 없는 특정 경로로 접근하려 할 때는 여전히 'Access is Denied' 라는 메시지를 보게 된다.

  - 예를 들기 위해서 '/admin'으로 접근할 때에는 'ADMIN'이라는 권한이 있어야만 하는 설정을 추가하겠다.

    ```Java
    // SecurityConfig.java
    
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");

        http.authorizeRequests().antMatchers("/guest/**").permitAll();
        http.authorizeRequests().antMatchers("/manager/**").hasRole("MANAGER");
        http.authorizeRequests().antMatchers("/admin/**").hasRole("ADMIN");

        http.formLogin().loginPage("/login");
    }
    ```

  - 브라우저에서 '/admin' 경로로 접근하면 브라우저는 자동으로 '/login' 경로로 이동하게 된다.

  - 지정한 아이디와 패스워드인 'manager/1111'을 입력하고 로그인하면 여전히 'Access is denied'라는 메시지를 보게 된다.

  - 사용자에게 권한이 없음을 알려주고, 로그인 화면으로 이동할 수 있도록 안내 페이지를 작성할 필요가 있다. 이 설정은 HttpSecurity에서 exceptionHandling()을 이용해서 지정할 수 있다.

    ```Java
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        log.info("security config ...... ");

        http.authorizeRequests().antMatchers("/guest/**").permitAll();
        http.authorizeRequests().antMatchers("/manager/**").hasRole("MANAGER");
        http.authorizeRequests().antMatchers("/admin/**").hasRole("ADMIN");

        http.formLogin().loginPage("/login");
        
        // 사용자에게 권한이 없음을 알려주기 위한 방법
        http.exceptionHandling().accessDeniedPage("/accessDenied");
    }
    ```

    - exceptionHandling() 이후에 메소드는 acessDeniedPage()나 accessDeniedHandler()를 이용하는 것이 일반적이다.

<br />

  - '/accessDenied' URI 처리

    ```Java
    @Controller
    public class LoginController {

        @GetMapping("/login")
        public void login(){

        }

        @GetMapping("/accessDenied")
        public void accessDenied(){

        }
    }
    ```

    ```HTML
    <!-- accessDenied.html  -->

    <div class="panel panel-default">
        <div class="panel-body">
            <div class="alert alert-danger" role="alert">
                <h2>Sorry, you do not have permission to view this page.</h2>
                Click Login at <a th:href="@{/login}">here</a>
            </div>
        </div>
    </div>
    ```

<br />

### 로그아웃 처리

  - 스프링 시큐리티가 웹을 처리하는 방식의 기본은 HttpSession이므로 브라우저가 완전히 종료되면, 로그인한 정보를 잃게 된다.

    - 사용자가 브라우저를 종료하지 않을 경우 사용자는 로그아웃을 통해서 자인이 로그인했던 모든 정보를 삭제해야 한다. 이를 위해서 HttpSession의 정보를 무효화시키고, 필요한 경우 모든 쿠키를 삭제한다.

    ```Java
    // SecurityConfig.java

    @Override
    protected void configure(HttpSecurity http) throws Exception{
        ... 

        // 사용자에게 권한이 없음을 알려주기 위한 방법
        http.exceptionHandling().accessDeniedPage("/accessDenied");
        
        // HttpSession의 정보를 무효화
        http.logout().invalidateHttpSession(true);
    }
    ```

  - logout() 뒤에 invalidateHttpSession()과 deleteCookies()를 이용해서 이러한 처리를 할 수 있다.

  - 로그아웃을 특정한 페이지에서 진행하고 싶다면 먼저 로그아웃을 처리하는 URI를 처리해야하고, POST 방식으로 로그아웃을 시도해야 한다.

    ```Java
    // LoginController.java
    @Controller
    public class LoginController {
        ...
        
        @GetMapping("/logout")
        public void logout(){}
    }
    ```

  - SecurityConfig의 configure()에서 로그아웃을 위한 URI를 지정한다.

    ```Java
    // SecurityConfig.java

    @Override
    protected void configure(HttpSecurity http) throws Exception{
       ...
        // 사용자에게 권한이 없음을 알려주기 위한 방법
        http.exceptionHandling().accessDeniedPage("/accessDenied");

        // Logout을 위한 URI 지정 + HttpSession의 정보를 무효화
        http.logout().logoutUrl("/logout").invalidateHttpSession(true);
    }
    ```

    ```HTML
    <!-- logout.html -->

    <div class="panel panel-default">
        <div class="panel-body">

            <h2>CUSTOM LOGOUT PAGE</h2>

            <form method="post">
                <h3>Logout</h3>
                <input type="hidden" th:name="${_csrf.parameterName}"
                       th:value="${_csrf.token}" />
                <button type="submit" class="btn">LogOut</button>
            </form>
        </div>
    </div>
    ```