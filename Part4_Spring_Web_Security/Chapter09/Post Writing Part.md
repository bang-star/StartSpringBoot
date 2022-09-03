# 9.2 게시물 작성 부분

  - 가장 먼저 로그인한 사용자만 특정 URI에 접근이 가능하도록 설정하는 것이다.

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