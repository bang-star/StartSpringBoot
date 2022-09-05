# 9.2 게시물 작성 부분

  - 로그인한 사용자만 특정 URI에 접근이 가능하도록 설정해야 한다.

  - '/boards/list'는 모든 사용자가 볼 수 있도록 하고 '/boards/register'는 로그인한 사용자만 접근할 수 있도록 예제를 작성한다.

    ```Java
    // org.zerock.scurity.SecurityConfig 클래스의 일부

    @Override
    protected void configure(HttpSecurity http) throws Exception{

        log.info("security config ...... ");

        http.authorizeRequests()
            .antMatchers("/boards/list").permitAll()
            .antMatchers("/boards/register")
            .hasAnyRole("BASIC", "MANAGER", "ADMIN");

        // http.formLogin();
        http.formLogin().loginPage("/login");

        // 사용자에게 권한이 없음을 알려주기 위한 방법
        http.exceptionHandling().accessDeniedPage("/accessDenied");

        // Logout을 위한 URI 지정
        // http.logout.invalidateHttpSession(true);
        http.logout().logoutUrl("/logout").invalidateHttpSession(true);

        // http.userDetailsService(zerockUsersService);

        http.rememberMe()
            .key("zerock")
            .userDetailsService(zerockUsersService)
            .tokenRepository(getJDBCRepository())
            .tokenValiditySeconds(60*60*24);
    }
    ```
  
  - hasAnyRole()를 이용하면 여러 개의 Role을 설정할 수 있으므로, 하나 이상의 Role을 지정할 때 편하게 사용할 수 있다.

  - 프로젝트를 실행하고 새로운 게시물을 작성하느 페이지를 호출할 때 정상적으로 접근 제한이 되는지를 확인한다.

  - 현재 /login에 해당하는 웹 페이지는 존재하지 않으므로 templates 폴더에 login.html 파일을 추가한다.

<br />

  ```HTML
  <!-- login.html -->

  <html xmlns:th="http://www.thymeleaf.org"
	xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
	layout:decorate="~{/layout/layout1}">

  <div layout:fragment="content">

    <div class="panel-heading">Login Page</div>
    <div class="panel-body">

      <div th:if="${param.error != null}">
        <h2>Invalid Username or password</h2>
        <h2 th:value="${param.error}"></h2>
      </div>

      <div th:if="${param.logout != null}">
        <h2>You have been logged out.</h2>
      </div>

      <div class='container'>
        <form method="post">
          <p>
            <label for="username">Username</label>
            <input type="text" id="username" name="username" value="user88" />
          </p>
          <p>
            <label for="password">Password</label>
            <input type="password" id="password" name="password" value="pw88" />
          </p>

          <p>
            <label for="text">Remember-Me</label>
            <input type="checkbox" id="remember-me" name="remember-me" />
          </p>

          <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
          <button type="submit" class="btn">Log in</button>
        </form>

      </div>

    </div>

  </div>

  <!--  end fragment -->
  <th:block layout:fragment="script">

    <script th:inline="javascript">
      
    </script>

  </th:block>
  ```
  
  - 주소창에서 '/login'을 직접 호출해서 로그인을 시도하는 경우에 Spring security의 기본 설정에 따라 '/' 경로로 이동하게 된다. 반면에 특정 페이지에서 '/login'으로 이동한 경우에는 로그인 후에 자동으로 기존 페이지로 이동하게 된다.

  - 로그인하지 않은 사용자가 브라우저에서 'boards/register'를 호출하게 되면 이전과 동일하게 '/login'으로 이동하게 된다. 로그인 후에는 자동으로 'boards/register'로 이동하는 것을 볼 수 있다.

<br />
<hr />
<br />

## 9.2.1 게시물 작성 시 사용자 아이디 편집

  - URI에 대한 접근 제어 처리가 완료되었다면, 게시물 등록 화면에서 로그인한 사용자의 아이디를 작성자의 입력란에 자동으로 처리하도록 한다.

  - Thymeleaf-security를 이용해서 처리한다. 
    
    - boards 폴더 내 register.html 내에서 Thymeleaf-security를 사용할 수 있도록 네임스페이스를 편집한다.

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org"
  	xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"
	xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
	layout:decorate="~{/layout/layout1}">
    ```

  - 작성자의 입력 부분은 'readonly'를 지정하고, 현재 로그인한 사용자의 uid가 출력되도록 수정한다.

    ```HTML
    <div class="form-group" th:with="member=${#authentication.principal.member}">
      <label>Writer</label> 
      <input class="form-control" name="writer"	th:value="${member.uid}" readonly="readonly" />
    </div>
    ```

<br />

  - 스프링 시큐리티가 적용되면 별도의 설정이 없는 이상 GET 방식을 제외한 POST 등의 방식으로 전송할 경우 CSRF 토큰 값이 반드시 필요하다.

  - form 태그에 Thymeleaf의 속성을 추가하면 자동으로 'CSRF'필터가 적용되어 있다. 이 때문에 직접 CSRF 값을 지정할 필요 없이 form 데이터를 전송할 수 있다.

  - CSRF 토큰까지 처리된 것을 확인한 후에 게시물을 등록하면 정상적으로 새로운 게시물이 등록되는 것을 볼 수 있다.

<br />
<hr />
<br />

## 9.3 게시물 조회

  - 게시물 조회 화면에서 다음과 같은 부분에 시큐리티 적용이 필요하다.

    - 현재 게시물의 작성자만이 수정/삭제가 가능하도록 제어

    - 게시물의 댓글 처리 시에 대한 제어

  - 댓글 처리는 Ajax를 이용하여 처리하고, 현재 게시물의 작성자와 현재 페이지를 보는 사용자가 동일한 사용자인 경우네는 '수정/삭제'를 처리할 수 있도록 제어한다.


<br />
<hr />
<br />

## 9.3.1 게시물 수정/삭제 버튼의 제어

  - 수정/삭제 버튼을 제어하는 방식

    1. 화면상에서 버튼 자체를 안 보이도록 처리하는 방식

    2. 버튼은 보이고, JavaScript를 이용해서 로그인을 유도하는 방식이 일반적

  - 버튼 자체를 감추는 방식에서는 현재 사용자가 로그인을 한 사용자인지 아닌지를 구분해서 처리해야 한다.

  - view.html에 Thymeleaf-security 네임스페이스 처리

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org"
  	xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"
	xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
	layout:decorate="~{/layout/layout1}">
    ```