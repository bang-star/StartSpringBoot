# 8.4 다양한 인증 방식

  - 스프링 시큐리티를 이용할 때 원하는 방식으로 동작할 수 있도록 하기 위해서는 기본적인 구조에 대한 이해가 필요하다.

    <img src="https://ifh.cc/g/4tNyPc.png">


    - 외부에서 인증이 필요하다고 판단되는 경우 가장 필요한 존재는 인증에 대한 실제적인 처리를 담당하는 '인증 매니저(Authentication Manager)'이다.

    - '인증 매니저'는 결과적으로 인증과 관련된 모든 정보를 UserDetails라는 타입으로 변환하는데, 이를 위해서 자신이 어떻게 관련 정보를 처리해야 하는지를 판단할 UserDetailsService라는 존재를 활용하게 된다.

    - 만일 개발자가 인증되는 방식을 수정하고 싶다면 UserDetailsSerivce라는 인터페이스를 구현하고, 인증 매니저에 연결시켜준다.

    - 결과로 반환되는 UserDetails는 기본적인 사용자 계정과 같은 정보와 더불어 사용자가 어떤 권한들을 가지고 있는지를 Collection 타입으로 가지고 있습니다.

    - 인증 매니저와 인증 정보를 제공하는 UserDetailsService의 관계를 좀 더 쉽게 표현하면 다음과 같다.

    <br />

        <img src="https://i.ibb.co/s1z6bs7/23323.png">

<br />
<hr />
<br />

## 8.4.1 스프링 시큐리티의 용어에 대한 이해

  - 스프링 시큐리티는 사용자의 보안 정보를 확인할 수 있는 다양한 방법을 제공한다.

     - 이 방식을 이해하기 위해서 몇 개의 클래스나 인터페이스의 이름에서 사용하는 용어를 이해할 수 있다.

     - AuthenticationManager(인증 매니저)

        - AuthenticationManagerBuilder(인증 매니저 빌더)

        - Authentication(인증)

     - UserDetailsService 인터페이스

        - UserDetailsManager 인터페이스

     - UserDetails 인터페이스

        - User 클래스

<br />

  - AuthenticationManager(인증 매니저)는 이전에 작성한 코드와 관련이 있다.

    ```Java
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth){
        log.info("build Auth Global");
    }
    ```
    
    <br />

    - AuthenticationManagerBuilder는 '인증 매니저를 생성하는 빌더'라고 생각 할 수 있다. 이를 이용해서 실제로 인증을 처리하는 인증 매니저를 생성한다.

    - (예시) auth.inMemoryAuthentication()의 경우 메모리 기반의 인증 매니저를 생성하게 된다.

    - AuthenticationManagerBuilder는 여러 종류의 인증 매니저들을 생성할 수 있는데, 대표적으로 메모리를 이용하거나 JDBC, LDAP 등을 사용하는 인증 매니저들을 생성할 수 있고 각 인증 매니저는 인증(Authentication)이라는 처리를 할 수 있도록 authenticate()라는 메소드를 구현한다.

  - 인증 매니저가 매니저가 사용하는 UserDetailSerivce()는 실제로 인증/인가 정보를 처리하는 주체를 의미한다.

    - (예시) API 중에 앞의 예제에서 사용하는 메모리상에서 인증/인가 정보를 처리하는 InMemoryUserDetailsManager의 경우 UserDetailsService 인터페이스를 구현하고 있는 것을 볼 수 있다.

    <br />

    ```Java
    Class InMemoryUserDetailsManager

    java.lang.Object
    org.springframework.security.provisioning.InMemoryUserDetailsManager

    All Implemented Interfaces:
    UserDetailsService, UserDetailsManager
    ```

  - UserDetailsService 인터페이스는 인증의 주체에 대한 정보를 가져오는 하나의 메소드만이 존재한다.

    <img src="https://user-images.githubusercontent.com/63120360/187019082-4218ce6d-6a2e-48a4-9fa5-da8cb57f91cb.PNG">

    - loadUserByUsername() 메소드는 UserDetails라는 인터페이스 타입을 반환한다.

    - UserDetails는 '사용자의 계정 정보 + 사용자가 가진 권한 정보'의 묶음이다.

  - 정리

    - 모든 인증은 인증 매니저(AuthenticationManager)를 통해서 이루어진다. 인증 매니저를 생성하기 위해 인증 매니저 빌더(AuthenticationManagerBuilder)라는 존재가 사용된다.

    - 인증 매니저를 이용해서 인증(Authentication)이라는 작업이 수행된다.

    - 인증 매니저들은 인증/인가를 위한 UserDetailsService를 통해서 필요한 정보들을 가져온다.

    - UserDetails는 사용자의 정보 + 권한 정보들의 묶음이다.

  - 스프링 시큐리티에는 기본적으로 많이 사용하는 인증 매니저들과 UserDetailsService를 구현한 클래스들이 다수 존재한다. 

    - 메모리상의 인증 매니저를 사용 (O)

    - 데이터베이스를 이용하는 인증 매니저 (예정)

    - 개발자가 직접 생성한는 인증 매니저 (예정)

<br />
<hr />
<br />

## 8.4.2 JDBC를 이용한 인증 처리

  - 메모리상에 있는 제한적인 정보를 이용해서 로그인/로그아웃을 처리해 주었다면, 데이터베이스를 연동하고 실제 정보를 이용해서 로그인을 진행해 보아야 한다. 이 작업은 JdbcUserDetailsManagerConfigurer라는 객체를 이용해서 손쉽게 처리할 수 있다.

  - 스프링 시큐리티가 데이터베이스를 연동하는 방법
    
    1. 직접 SQL 등을 지정해서 처리하는 방법

    2. 기존에 작성된 Repository나 서비스 객체들을 이용해서 별도의 시큐리티 관련 서비스를 개발하는 방법이 있다.

  - 1번은 DataSource와 SQL을 이용해서 처리하는 방법이다. 
    
    - DataSource 타입의 객체를 주입해야 한다.

    <br />

    ```Java
    @Log
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
        ... 
        @Autowired
        DataSource dataSource;
    }
    ```

    <br />

    - 사용자에 대한 계정 정보와 권한을 체크하는 부분에는 DataSource를 이용하고, SQL을 지정하도록 작성한다.

        1) 사용자의 계정 정보를 이용해서 필요한 정보를 가져오는 SQL이 필요

        2) 해당 사용자의 권한을 확인하는 SQL이 필요

    <br />

    ```JaVA
        @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception{
        log.info("build Auth global ...... ");
        
        String query1 = "SELECT uid username, upw password, true enabled FROM tbl_members WHERE uid = ?";

        String query2 = "SELECT member uid, role_name role FROM tbl_member_role WHERE member = ?";

        // auth.inMemoryAuthentication()
        //     .withUser("manager")
        //     .password("1111")
        //     .roles("MANAGER");
        
        // JDBC를 이용한 인증 처리
        
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .usersByUsernameQuery(query1)
                .rolePrefix("ROLE_")
                .authoritiesByUsernameQuery(query2);
    }
    ```

      - auth.jdbcAuthentication() 메소드는 JdbcUserDetailsManagerConfigurer 객체를 반환하게 된다. 이 객체를 이용해서 DataSource를 주입하고, 필요한 SQL문을 파라미터로 전달하는 방식을 이용해서 단순한 몇 가지의 설정만으로 인증 매니저를 생성하게 된다.

      - 가장 핵심이 되는 메소드는 userByUsernameQuery()와 authoritiesByUsernameQuery()라는 메소드이다.

      - username을 이용해서 특정한 인증 주체(사용자) 정보를 세팅하고, username을 이용해서 권한에 대한 정보를 조회한다.

      - 스프링 시큐리티에서 가장 주의해야 하는 용어는 username이다. 설계 시에는 아이디(id)와 같은 식별 데이터를 이용하지만, 스프링 시큐리티에서는 username이라는 용어로 사용한다.

      - userByUsernameQuery()를 이용하는 경우에는 우선 username과 password, enabled라는 칼럼의 데이터가 필요하다. 
        
        - 실제 테이블의 칼럼명과 다를 경우에는 SQL에서는 칼럼에 alias(가명)를 적용해서 처리할 수 있다.

        - Enabled 칼럼은 해당 게정이 사용 가능한지를 의미한다(만일 적절한 데이터가 없다면 무조건 true를 이용하도록 설정하면 된다.)

        - authoritiesByUsernameQuery()의 파라미터로 사용되는 SQL은 실제 권한에 대한 정보를 가져오는 SQL이다.

        - SQL은 username 하나의 파라미터를 전달하고, username과 권한 정보를 처리하도록 작성한다.

      - 데이터베이스에 'ROLE_'라는 문자열은 없고, 단순히 'BASIC, MANAGER, ADMIN'과 같은 이름으로 저장되어 있기 때문에 rolePrefix() 메소드를 이용해서 'ROLE_'라는 문자열을 붙여 준다.

      - 사용자가 데이터베이스에 존재하는 username(아이디 정보)와 password(패스워드) 정보를 입력하고 로그인을 시도하면 다음과 같이 로그가 출력되는 것을 볼 수 있다.

    ```Log
    SecurityContextHolder not popuulated with anonymous token, as it already contained; 'org.springframework.security.authentication.UsernamePasswordAuthenticationToken@ff0bfa9b: Principal: org.springframework.security.core.userdetails.User@ce2b2c0b: Username: user88; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialNonExpired:true; AccountNonLocked: true; Granted Authorities: ROLE_MANAGER; Credentials:[PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@2cd90: RemoteIpAddress:0:0:0:0:0:0:0:1; SessionId: ...; Granted Authorities: ROLE_MANAGER'
    ```

     - 출력되는 내용을 자세히 보면 Username, Password, Enabled, AccountNonExpired ... 등의 다양한 속성을 가진 'org.springframework.security.core.userdetails.User'타입의 객체가 만들어진 것을 짐작할 수 있다. 

<br />
<hr />
<br />

## 8.4.3 사용자 정의(custom) UserDetailsService

  - 스프링 시큐리티가 필요한 객체는 'org.springframework.security.core.userdetails.User'타입의 객체를 만들어내는 것임을 알 수 있다.

  - User 클래스는 UserDetails라는 인터페이스를 구현해서 보다 상세한 사용자 정보를 처리한다.

    <img src="https://user-images.githubusercontent.com/63120360/187032620-d40c4126-1c56-4e61-8bbc-0c63d84331fb.png">

<br />

  - 모든 인증 매니저는 결과적으로 UserDetails타입의 객체를 반환하도록 구현하는데, 개발자가 인증 매니저를 커스터마이징하려면 UserDetailsService 인터페이스를 구현하고, 이를 HttpSecurity 객체가 사용할 수 있도록 지정하면 된다.

  - API에서 UserDetailsService를 보면 이미 여러 클래스가 UserDetailsService를 구현해 두었다.

    <img src="https://user-images.githubusercontent.com/63120360/187032724-946964f9-ecf9-4cd0-8841-930e8e29b7f9.png">

  - 본인이 원하는 방식으로 인증을 처리하고 싶다면 가장 먼저 UserDetailsService 인터페이스를 구현하는 새로운 클래스를 정의해야 한다.

    ```Java
    @Service
    @Log
    public class ZerockUsersService implements UserDetailsService {

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            // TODO Auto-generated method stub
            return null;
        }
    }
    ```

  - ZerockUsersService는 @Service 어노테이션이 추가되어 있으므로, 스프링에서 빈으로 처리된다.

  - SecurityConfig에서 만들어진 ZerockUsersService 객체를 주입받을 수 있도록 처리한다.(기존의 DataSource는 더 이상 사용하지 않으므로 제거한다.)

  - ZerockUsersService를 이용하게 되면 기존의 설정은 의미가 없으므로 configureGlobal() 메소드는 사용하지 않도록 제거할 수 있습니다.

    ```Java
    @Log
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {

        @Autowired
        DataSource dataSource;

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
        }

        @Bean
        public PasswordEncoder passwordEncoder(){
            return new PasswordEncoder() {
                @Override
                public String encode(CharSequence rawPassword) {
                    return rawPassword.toString();
                }

                @Override
                public boolean matches(CharSequence rawPassword, String encodedPassword) {
                    return rawPassword.equals(encodedPassword);
                }
            };
        }
    }
    ```

  - SecurityConfig의 configure()에서 사용하는 HttpSecurity 객체는 ZerockUsersService를 이용하게 한다.

    ```Java
    @Log
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {

        @Autowired
        ZerockUsersService zerockUsersService;
        
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

            http.userDetailsService(zerockUsersService);
        }

        @Bean
        public PasswordEncoder passwordEncoder(){
            return new PasswordEncoder() {
                @Override
                public String encode(CharSequence rawPassword) {
                    return rawPassword.toString();
                }

                @Override
                public boolean matches(CharSequence rawPassword, String encodedPassword) {
                    return rawPassword.equals(encodedPassword);
                }
            };
        }
    }
    ```

<br />

### 간단한 커스텀 UserDetailsService 사용하기

  - 인증을 처리할 때 새롭게 생성한 ZerockUsersService를 이용하므로, 간단한 코드를 추가해서 인증이 정상적으로 처리되는지를 확인해야 한다.

  - UserDetailsService의 loadUserByUsername()은 사용자의 계정 정보(아이디)를 이용해서 UserDetails 인터페이스를 구현한 객체를 반환해야 한다.

  - Spring Security API에는 UserDetails 인터페이스를 구현한 클래스들이 존재한다.

    <img src="https://user-images.githubusercontent.com/63120360/187036452-d8a0fe58-b24f-4d20-b8ab-1f8bfe9a1c46.png">

    <br />

    - UserDetails 인터페이스를 직접 구현하는 방식을 사용하는 것도 좋지만, UserDetails 인터페이스는 생각보다 많은 메소드가 존재하므로, 간편하게 UserDetails를 구현해둔 User 클래스를 이용할 것이다.

  - User 클래스는 username과 password, Authority라는 정보만을 이용하는 간단한 생성자를 제공한다. 이때 Authority는 '권한'을 의미한다.

    ```Java
    @Service
    @Log
    public class ZerockUsersService implements UserDetailsService {

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            // TODO Auto-generated method stub
            User sampleUser = new User(username, "1111", Arrays.asList(new SimpleGrantedAuthority("ROLE_MANAGER")));
            return sampleUser;
        }
    }
    ```

    <br />

    - loadUserByUsername()의 내부에서 User 클래스를 이용해서 객체를 생성한다. 이때 User 클래스의 인스턴스는 Collection< Authority>를 가질 수 있으므로, Arrays.asList()를 이용해서 여러 권한을 부여할 수 있다.

    <br />

  - SampleGrantedAuthority 클래스는 GrantedAuthority라는 인터페이스의 구현체이다.

    <img src="https://user-images.githubusercontent.com/63120360/187036718-706af419-c790-4b4d-a436-1cd25b3b4972.png">

  - GrantedAuthority 인터페이스는 문자열을 반환하는 getAuthority()메소드 하나만을 가지고 있다.