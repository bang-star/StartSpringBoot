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


  