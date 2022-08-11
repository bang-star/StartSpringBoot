# 6. Spring MVC를 이용한 통합

## 6.1 프로젝트의 기본 구조 생성

  - 프로젝트 이름 : boot06
  - 설치 요소

     - DevTools

     - Lombok

     - JPA
     
     - MySQL

     - Thymeleaf

     - Web

   - 레이아웃 라이브러리 추가

     ```XML
        <dependency>
            <groupId>nz.net.ultraq.thymeleaf</groupId>
            <artifactId>thymeleaf-layout-dialect</artifactId>
            <version>2.2.1</version>
        </dependency>
     ```

   - application.properties 설정

     ```XML
        spring.datasource.driver-class-name=com.mysql.jdbc.Driver
        spring.datasource.url=jdbc:mysql://localhost:3306/jpa_ex?useSSL=false&serverTimezone=Asia/Seoul
        spring.datasource.username=jpa_user
        spring.datasource.password=jpa_user

        # ??? ??(create)
        spring.jpa.hibernate.ddl-auto=update
        # DDL ?? ? ?????? ??? ??? ??????
        spring.jpa.generate-ddl=true
        # ???? SQL?? ??? ????
        spring.jpa.show-sql=true
        # ??????? ??? ??????
        spring.jpa.database=mysql
        # MySQL
        spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

        # ?? ??
        logging.level.org.hibernate=info

        spring.thymeleaf.cache=false
        logging.level.org.springframework.web=info
        logging.level.org.zerock=info
     ```

## 6.1.1 레이아웃 분리해 두기

  - static 폴더의 내용은 'boot5'프로젝트에서 사용했던 내용을 복사

  - layout 폴더는 'boot5'프로젝트에서 사용했던 layout.html 파일을 복사


### 부트스트랩 추가해 두기

  - HTML5 Bolilerplate는 프로젝트의 기본 구조를 생성할 때는 유용하지만, 각 화면을 구성하는 데에는 CSS가 필요하므로, 부트스트랩을 추가합니다.

  - BootStrap CDN 항목에 부트스트랩을 웹 페이지에 포함하는 방법이 설명되어 있다.

  ```HTML
    <script src="https://code.jquery.com/jquery-1.12.0.min.js"></script>
    <script>window.jQuery || document.write('<script th:src="@{/js/vendor/jquery-1.12.0.min.js}"><\/script>')</script>
    <script th:src="@{/js/plugins.js}"></script>
    <script th:src="@{/js/main.js}"></script>

    <!-- Latest compiled and minified CSS -->
    <link rel="stylesheet"
          href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
          integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u"
          crossorigin="anonymous">

    <!-- Optional theme -->
    <link rel="stylesheet"
          href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css"
          integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp"
          crossorigin="anonymous">

    <!-- Latest compiled and minified JavaScript -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"
            integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa"
            crossorigin="anonymous"></script>

  ```

  - layout1.html은 여러 페이지에서 사용하는 레이아웃을 의미하므로, 부트스트랩의 panel이라는 것을 이용해서 전체 화면 구성에 사용한다.

  - 상단에는 간단한 문자들을 보여주고, 레이아웃의 내용물은 부트스트랩의 panel이라는 것으로 처리하도록 변경합니다.

    ```HTML
    <div class="page-header">
        <h1>
            Boot06 Project <small>for Spring MVC + JPA</small>
        </h1>
    </div>

    <div class="panel panel-default" layout:fragment="content">
        <div class="panel-body">Web Board List Page</div>
    </div>

    <script src="https://code.jquery.com/jquery-1.12.0.min.js"></script>
    ```

## 6.1.2 컨트롤러 생성 및 화면 확인하기

  - WebBoardController 클래스
  
    ```Java
    @Controller
    @RequestMapping("/boards/")
    @Log
    public class WebBoardController {
        
    }
    ```

    - 게시물 리스트

        ```Java
        @GetMapping("/list")
        public void list(){
            log.info("list() called ... ");
        }
        ```

     - URI에 맞는 html 페이지 작성

       ```HTML
       <html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{/layout/layout1}">

        <div layout:fragment="content">

            <div class="panel-heading">List Page</div>
            <div class="panel-body">list content ... </div>
        </div>

        <th:block layout:fragment="script">
            <script th:inline="javascript"></script>
        </th:block>
       </html>
       ```

       - list.html은 부트스트랩의 간단한 스타일을 이용해서 Panel을 구성한다.

           - 'panel-heading' 등을 이용해서 화면에 간단한 스타일 지정


## 6.1.3 엔티티 클래스와 Repository 설계

   - 도메인

     ```Java
     @Getter
     @Setter
     @Entity
     @Table(name = "tbl_webboards")
     @EqualsAndHashCode(of = "bno")
     @ToString
     public class WebBoard {
        
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long bno;
        private String title;
        
        private String writer;
        private String content;
        
        @CreationTimestamp
        private Timestamp regdate;
        @UpdateTimestamp
        private Timestamp updatedate;
     }
     ```

  - Repo

     ```Java
    public interface WebBoardRepository extends CrudRepository<WebBoard, Long> {

    }
     ```

## 6.1.4 Querydsl 설정

 - 게시물의 검색에는 동적 쿼리를 이용해서 쿼리를 처리할 것이므로 Querydsl 관련 라이브러리와 코드 생성 플러그인을 추가해야 한다.

   ```xml
    <!-- https://mvnrepository.com/artifact/com.querydsl/querydsl-apt -->
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <version>5.0.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
        <version>5.0.0</version>
    </dependency>
   ```
  
  - Qdomain을 생성하는 코드 생성 플러그인

     ```xml
    <!-- ADDED FOR Querydsl -->
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

  - 검색 기능을 구현할 때 사용할 수 있도록 QueryDsl 관련 인터페이스를 추가해야한다.

    ```Java
    public interface WebBoardRepository extends CrudRepository<WebBoard, Long>, QuerydslPredicateExecutor<WebBoard> {
    }
    ```
<br />

## 6.1.5 테스트 코드 작성

  - WebBoardRepository는 가능하면 테스트를 이용하도록 해야한다.
     