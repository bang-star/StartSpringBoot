# 9.1 프로젝트 구성

  - 6, 7장에서 작성한 게시물 관리 시스템과 8장에서 작성했더 ㄴ시큐리티 프로젝트를 하나로 묶는 작업이다.

  - 프로젝트 

    - 프로젝트 명 : boot09

    - 설치 모듈
        
      - Security
      
      - DevTools

      - Lombok

      - JPA

      - MySQL

      - Thymeleaf

      - Spring Web

    <br />

     <img src="https://user-images.githubusercontent.com/22147400/188074286-d128b5db-a67f-440f-be73-e7e6d203f710.png">

    <br />

  - 작성된 프로젝트의 모든 기능을 사용하기 위해서 pom.xml에 추가 라이브러리들이 필요하다.

    - Thymeleaf-layout 라이브러리

    - Querydsl 관련 라이브러리

    - Thymeleaf-security 라이브러리

    <br />

    ```XML
    <dependency>
        <groupId>nz.net.ultraq.thymeleaf</groupId>
        <artifactId>thymeleaf-layout-dialect</artifactId>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.querydsl/querydsl-apt -->
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras-springsecurity5 -->
    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-springsecurity5</artifactId>
        <version>3.0.4.RELEASE</version>
    </dependency>

    <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>apt-maven-plugin</artifactId>
        <version>1.1.3</version>
        <executions>
            <execution>
                <goals>
                    <goal>process</goal>
                </goals>
                <configuration>
                    <outputDirectory>target/generated-sources/java</outputDirectory>
                    <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                </configuration>
            </execution>
        </executions>
    </plugin>
    ```
    
<br />
<hr />
<br />

## 9.1.1 기존 프로젝트 통합

  - 프로젝트를 통합할 때 가장 우선적으로 application.properties를 처리한다.

    ```Java
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/jpa_ex?useSSL=false&serverTimezone=Asia/Seoul
    spring.datasource.username=jpa_user
    spring.datasource.password=jpa_user

    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.generate-ddl=false
    spring.jpa.show-sql=true
    spring.jpa.database=mysql
    spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

    logging.level.org.hibernate=info

    spring.thymeleaf.cache=false

    logging.level.org.springframework.web=info
    logging.level.org.springframework.security=debug

    logging.level.org.zerock=info
    ```

  - 만들어진 프로젝트에 7장의 게시물과 댓글 처리와 관련된 내용이 필요하므로 우선적으로 동작하도록 한다.

    기존 프로젝트의 org.zerock 패키지를 제외한 모든 하위 패키지를 복사해서 현재 프로젝트로 넣어준다.

    <img src="https://user-images.githubusercontent.com/22147400/188084811-3d739feb-5b34-4506-807b-5a3fa27b48be.png">

    <br />

  - 'src/main/resources'의 static 폴더의 내용물을 복사하고, templates 폴더의 파일들을 복사해서 추가한다.

    <img src="https://user-images.githubusercontent.com/22147400/188085302-dc6ae07f-84ae-435b-b23b-e319c35cfd30.png">

<br />

  - 현재까지 복사한 프로젝트가 정상적으로 동작하는지 프로젝트를 실행해서 확인해야 한다. 브라우저에서 '/boards/list'를 호출했을 때 정상적으로 호출되고, 로그인 페이지로 이동되는 것을 볼 수 있다.

    - 스프링 부트에 자동 설정 기능이 동작하기 때문이다. 프로젝트 생성 시 'security'를 추가한 상태이므로 기본 인증 처리가 설정되기 때문에 로그인 창이 보인다.

    - 기본 인증 매니저는 'user'라는 계정을 가지고, 패스워드는 프로젝트의 로딩 시 출력되는 기본 패스워드이다.

        ```Java
        Using generated security password: d50b4d3b-2213-4900-885d-80283be67db8
        ```

     - 이를 이용해서 로그인을 선택하면 정상적으로 화면이 출력된다.

<br />
<hr />
<br />

## 9.1.2 시큐리티 설정

  - 기본 계정과 패스워드를 이용해서 로그인을 하고, 정상적인 화면이 나오는 것을 확인했다면, 8장에 사용했던 설정을 이용해서 시큐리티 설정을 추가한다.

    - org.zerock.domain 패키지에 Member와 MemberRole 클래스를 복사 

    - org.zerock.persistence 패키지에 MemberRepository 복사

    - org.zerock.controller 패키지에 LoginController와 MemberController 복사

    - security 패키지는 그대로 복사

  - 코드를 통합한 이후에는 게시물 관리의 GET 방식으로 동작하는 기능들이 정상적으로 동작하는지를 확인해야 한다.

  - 8장에서 사용했던 화면을 추가하지 않은 이유는 Thymeleaf의 레이아웃 기능 등을 적용해서 새로 작성할 필요가 있기 때문이다.