# 8.6 Remember-Me 인증하기

  - 웹에서 로그인 처리는 크게 HttpSession을 이용하는 세션 방식과, 쿠기(Cookie)를 이용하는 방식이 있다.

     - 세션 방식

        모든 데이터를 서버에서 보관하고, 브라우저는 단순히 '키(key)'에 해당하는 '세션ID'만을 이용하기 때문에 좀 더 안전하다는 장점이 있지만, 브라우저가 종료되면 사용할 수 없기 때문에 모바일과 같은 환경에서 불편함이 있다.

     - 쿠키를 이용하는 방식

        브라우저에서 일정한 정보를 담은 '쿠키'를 전송하고, 브라우저는 서버에 접근할 때 주어진 쿠키를 같이 전송한다. 이때 서버에서는 쿠키에 유효기간을 지정할 수 있기 때문에 브라우저가 종료되어도 다음 접근 시 유효기간이 충분하다면 정보를 유지할 수 있다.

     - 'Remember-Me'

        최근에 웹 사이트에서는 '로그인 유지'라는 이름으로 서비스되는 기능으로 쿠키를 이용해서 사용자가 로그인했던 정보를 보관하는 것이다.

        스프링 시큐리티의  Remember-me 기능은 기본적으로 사용자가 로그인했을 때의 특정한 토큰 데이터를 2주간 유지되도록 쿠키를 생선한다. 브라우저에 전송된 쿠키를 이용해서 로그인 정보가 필요하면 저장된 토큰을 이용해서 다시 정보를 사용한다.

<br />
<hr />
<br />

## 8.6.1 로그인 화면에서 'remember-me' 체크박스 처리

  - 'remember-me'는 사용자가 선택해서 진행되어야 하므로, 로그인 페이지에서는 체크 박스를 만들어서 처리해야 한다.

    ```HTML
    <!-- login.html -->
    <form method="post">
        <p>
            <label for="username">Username</label>
            <input type="text" id="username" name="username" value="user88">
        </p>
        <p>
            <label for="password">Password</label>
            <input type="password" id="password" name="password" value="pw88">
        </p>

        <p>
            <label for="text">Remember-Me</label>
            <input type="checkbox" id="remember-me" name="remember-me">
        </p>

        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
        <button type="submit" class="btn">Log in</button>
    </form>
    ```
  
  - 새롭게 추가된 체크박스의 name 속성값 역시 'remember-me'로 지정합니다. 화면에서는 체크박스가 추가된 형태를 볼 수 있다.

<br />
<hr />
<br />

## 8.6.2 SecurityConfig에서의 설정

  - 'remember-me' 기능을 설정하기 위해서는 UserDetailsService를 이용해야 한다는 조건이 있지만, 이미 앞에서 만들어 두었기 때문에 HttpSecurity 인스턴스에 간단히 rememberMe()를 이요해 주면 처리가 가능하다.

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

        // Logout을 위한 URI 지정
        http.logout().logoutUrl("/logout").invalidateHttpSession(true);

        // http.userDetailsService(zerockUsersService);
        http.rememberMe().key("zerock").userDetailsService(zerockUsersService);
    }
    ```

  - rememberMe()에서는 쿠키의 값으로 암호화된 값을 전달하므로, 암호의 '키(key)'를 지정하여 사용한다.

  - SecurityConfig의 설정이 완료되고, 로그인 화면에서 'remember-me'를 선택한 후에 로그인하면 브라우저상에는 기본적인 JSESSIONID(톰캣의 경우에 생성된 세션의 키)의 쿠키 외에도 'remember-me'라는 이름의 쿠키가 생성되는 것을 확인할 수 있다.

    <br />

    <img src="https://user-images.githubusercontent.com/22147400/187823592-7ed84381-619c-4044-98eb-db0601c06f9d.png">

    <br />

  - 생성된 'remember-me' 쿠키의 Expires(유효기간)는 '로그인 시간 + 2주'입니다. 쿠키는 유효기간이 설정되면 브라우저 내부에 저장된다. 따라서 사용자가 브라우저를 종료하고 다시 서버상의 경로에 접근하게 되면 자동으로 생성된 JSESSIONID 쿠키(이하 세션 쿠키)는 없지만, 브라우저는 보관된 'remember-me'쿠키는 그대로 가지고 서버에 접근하게 된다.

  <br />

    <img src="https://user-images.githubusercontent.com/22147400/187829399-d7247efd-fb03-498f-9a66-f523b1b6674c.png">

  <br />

  - 브라우저에서 전송하는 정보를 보면 저장된 'remember-me' 쿠키의 유효기간이 지나지 않았으므로, 서버에 접근할 때 같이 전송되는 것을 볼 수 있다.

  - 'remember-me'쿠키는 기본적으로 로그아웃 시점에 같이 삭제되기 때문에 별도의 처리 없이도 자동으로 삭제된다.

<br />
<hr />
<br />

## 8.6.3 remember-me를 데이터베이스에 보관하기

  - 스프링 시큐리티는 기본적으로 'remember-me' 기능을 사용하기 위해서 'Hash-based Token 저장 방식'과 'Persistent Token 저장 방식'을 사용할 수 있습니다. 아무런 설정을 하지 않았으므로 'Hash-based'방식을 이용하게 된다.

  - 'remember-me' 쿠키의 생성은 기본적으로 'username'과 쿠키의 만료시간, 패스워드를 Base-64 방식으로 인코딩한 것이다.

    ```
    username + ":" + expiryTime + ":" + Md5Hex(username + ":" + expiryTime + ":" + password + ":" + key)
    ```
  
    - 이러한 방식의 문제점은 사용자가 패스워드를 변경하면, 정상적인 
    값이라도 로그인이 될수 없다는 단점을 가지고 있다.

    - 이를 해결하기 위해서 가능하면 데이터베이스를 이용해서 처리하는 방식을 권장한다. (쿠키를 이용한 안전한 로그인 방식에 대한 자세한 내용은 http://jaspan.com/improved_persistent_login_cookie_best_practice을 참고)

    - 데이터베이스를 이용하는 설정은 org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl클래스를 이용한다.

    - PersistenceTokenRepository 인터페이스를 이용해서 자신이 원하는대로 구현이 가능하지만, 관련된 모든 SQL을 직접 개발해야 하므로 사용하지 않는다.(https://github.com/spring-projects/spring-security/blob/master/web/src/main/org/springframework/security/web/authentication/rememberme/JdbcTokenRepositoryImpl.java 를 참고해서 구현할 수 있다.)

    - 이미 구현된 JdbcTokenRepositoryImpl을 그대로 활용해서 정보를 데이터베이스에 보관한다.

     - 토큰을 보관할 수 있는 테이블 생성

        ```SQL
        create table persistent_logins (
            username varchar(64) not null,
            series varchar(64) primary key,
            token varchar(64) not null,
            last_used timestamp not null
        );
        ```
     - SecurityConfig에서 rememberMe()를 처리할 때, JdbcTokenRepositoryImpl을 지정해 주어야 하는데 기본적으로 DataSource가 필요하므로 주입해 줄 필요가 있다.

        ```Java
        @Log
        @EnableWebSecurity
        public class SecurityConfig extends WebSecurityConfigurerAdapter {
            
            @Autowired
            DataSource dataSource;
            
            @Autowired
            ZerockUsersService zerockUsersService;
            
            ...
        }
        ```

     - HttpSecurity에서는 JdbcTokenRepositoryImpl을 이용할 것이므로 간단한 메소드를 이용해서 처리한다.

        ```Java
        // SecurityConfig.java

        private PersistentTokenRepository getJDBCRepository(){
        
            JdbcTokenRepositoryImpl repo = new JdbcTokenRepositoryImpl();
            repo.setDataSource(dataSource);
            return repo;
        }
        ```
     
     - HttpSecurity가 이를 이용하도록 configure() 메소드 수정

        ```Java
        // SecurityConfig.java에 configure() 메소드

        http.rememberMe()
            .key("zerock")
            .userDetailsService(zerockUsersService)
            .tokenRepository(getJDBCRepository())
            .tokenValiditySeconds(60*60*24);
        ```

         - tokenValiditySeconds는 쿠키의 유효시간을 초단위로 설정하는데 사용

         - 코드에서는 24시간을 유지하는 쿠키를 생성

  - SecurityConfig의 설정을 추가한 후에 화면에서 'remember-me'를 선택한 로그인을 하면 'persistent_logins' 테이블에 생성된 토큰의 값이 기록되는 것을 볼 수 있다.
    
    <br />

    <img src="https://user-images.githubusercontent.com/22147400/187849784-b0e0b8fc-8f67-46f9-a6f1-cf942cbcf0dd.png">

    <br />

    <img src="https://user-images.githubusercontent.com/22147400/187849950-68ab1f9f-f557-4905-bee8-160e0a61d27e.png">

    <br />

  - 브라우저에서 해당 토큰을 이용한 쿠키가 생성된다.

  - 쿠키의 생성은 패스워드가 아니라 series에 있는 값을 기준으로 하게 된다. 사용자가 'remember-me'를 한 상태에서 브라우저를 종료하고, 다시 URL로 접근하게 되면 서버에서는 다음과 같은 로그가 출력되게 된다.

    <img src="https://user-images.githubusercontent.com/22147400/187850648-4df41b3d-ffb5-4645-844f-520b20222c0a.png">
