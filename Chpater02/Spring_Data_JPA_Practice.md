# 2.2 Spring Data JPA를 위한 프로젝트 생성

 - 스프링 부트 프로젝트의 가장 큰 장점 중에 하나는 기존의 스프링 프로젝트와는 달리 스프링과 사용하는 라이브러리 버전에 신경 쓸 필요가 없다.

## 2.2.1 프로젝트 실행과 DataSource 설정

  - Spring Boot에 JDBC와 같은 추가 라이브러리를 포함해서 이용했을 떄의 가장 큰 차이점은 실행에 제약이 있다는 점이다.

  - 스프링 부트 자체가 자동으로 설정을 구성하기 때문에 정상적으로 실행이 되지 않는다.

    - 스프링 부트가 동작할 때 JDBC 등과 같은 라이브러리가 포함되어 있으면 라이브러리를 이용해 필요한 객체들을 구성하려고 하는데, 이때 데이터베이스 관련 설정이 전혀 없기 때문에 프로젝트를 실행할 수 없는 것이다.

    - 프로젝트를 정상적으로 실행하기 위해서는 우선 필요한 데이터베이스를 구성하고 DataSource를 지정해 주어야 한다.

<br />

### DataSource 설정

  - 현재 프로젝트가 실행이 안되는 이유는 스프링 부트 내에 JDBC 등의 설정이 포함되어 있는데, 이에 대한 설정이 전혀 없기 때문입니다.

    - 프로젝트 생성 시 포함되어 있는 application.properties를 이용해서 필요한 구성을 설정하는 방법

    - @Bean과 같은 어노테이션을 이용해서 Java 코드를 통해 필요한 객체를 구성하는 방법

    - 이외에도 YAML 파일이라는 것을 만들거나 XML을 이용하는 방법 등을 사용할 수 있다.

    - application.properties에 대한 모든 설정은 spring.io에서 확인할 수 있다.

  - 데이터베이스 연결에 필요한 JDBC 연결 정보

    ```Java
      spring.datasource.driver-class-name = com.mysql.jdbc.Driver 
      spring.datasource.url=jdbc:mysql://localhost:3306/jpa_ex?useSSL=false
      spring.datasource.username=jpa_user
      spring.datasource.password=jpa_user
    ```

  - DataSource를 구성하기 위해 추가적으로 많은 설정을 할 수 있지만, 최소한의 설정에 최대한 빠른 테스트를 위해 우선 4개의 항목만 추가한 상태에서 프로젝트를 실행해 보고 에러가 없는 것을 확인하도록 합니다.

  - 스프링 부트에서 하나의 DataSource를 구성하는 것은 단순하지만, 2개 이상의 DataSource를 구성하기 위해서는 보다 복잡한 설정을 추가해야 한다.

<br />
<hr />
<br />

## 2.2.2 엔티티 클래스 설계

  - 관계형 데이터베이스에 데이터를 보관하려면 테이블을 생성해 주어야 한다. 이는 Java에서 특정한 구조를 설계하기 위해 클래스를 설계하는 것과 유사하다.

  - JPA는 자동으로 테이블을 생성할 수 있는 기능을 가지고 있으므로 1) SQL을 이용해서 테이블을 먼저 생성하고 엔티티 클래스를 만드는 방식이나, 2) JPA를 이용해서 클래스만 설계하고 자동으로 테이블을 생성하는 방식을 모두 사용할 수 있다.

  - JPA의 엔티티 클래스를 생성 과정

     1. 객체지향의 설계대로 클래스들을 설계한다.

     2. @Id, @Column 등을 이용해서 각종 제약 조건을 추가하고, 설정합니다.

     3. 엔티티 간의 연관관계를 설정합니다.

<br />

### Step 1. 엔티티 클래스 설계

```Java
@Getter
@Setter
@ToString
public class Board {
    private Long bno;
    private String title;
    private String writer;
    private String content;
    
    private Timestamp regdate;      // LocalDateTime
    private Timestamp updatedate;   // LocalDateTime 
}

```

  - 엔티티 클래스의 시간을 처리할 때 LocalDateTime을 이용하는 것이 일반적이지만, 화면 처리 등에서 좀 더 편리하도록 Timestamp 타입을 이용하였다.

### Step 2. JPA를 위한 어노테이션 추가

  - 클래스에 적절한 어노테이션을 적용해 JPA에 필요한 정보를 설정한다.

  - 설정에 주로 사용하는 어노테이션

  <img src="https://user-images.githubusercontent.com/63120360/181416701-3240f3c0-889f-41a8-aecf-31d9b8e7d9d2.png">

  <img src="https://user-images.githubusercontent.com/63120360/181416768-3de5fbab-5585-4224-880c-303715090cad.png">

  - 클래스 선언부에는 반드시 @Entity가 설정되어야 합니다.

     - @Entity 설정은 해당 클래스가 엔티티 클래스임을 명시

  - @Table을 설정하는 경우에는 기본적으로 데이터베이스에 클래스명과 동일한 이름으로 생성됩니다.

     - 만일 클래스명과 다른 이름으로 테이블 이름을 지정하고 싶을 때 사용

  - @Id는 가장 중요한 어노테이션이다.

     - @Id는 해당 컬럼이 식별키(Primary Key)라는 것을 의미한다.

     - 모든 엔티티에는 반드시 @Id를 지정해 주도록 한다.

     - 식별키를 지정하는 방식은 다양하다.

     - 데이터베이스마다 식별키 지정 방식

        1) 사용자가 직접 지정

        2) 자동으로 생성되는 번호 등을 이용

        3) 별도의 방법으로 필요한 데이터를 생성하는 방식을 이용(Oracle의 경우 Sequence를 사용, MySQL은 Auto Increment를 이용하는 등 데이터베이스마다 약간의 차이가 있음을 주의)

     - @Id는 주로 @GeneratedValue라는 어노테이션과 같이 이용해서 식별키를 어떤 전략으로 생성하는지를 명시합니다.

  - @GeneratedValue는 strategy 속성과 generator 속성으로 구분
    
     - strategy : AUTO, TABLE, SEQUENCE, IDENTITY

     - generator : @TableGenerator, @SequenceGenerator

    ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_boards")
    public class Board {
        
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        private Long bno;
        
        private String title;
        private String writer;
        private String content;
        
        @CreationTimestamp
        private Timestamp regdate;      // LocalDateTime
        @UpdateTimestamp
        private Timestamp updatedate;   // LocalDateTime
    }
    ```

    - @GeneratedValue 어노테이션은 식별키의 생성 전략을 지정
      
      - strategy는 속성을 이용해서 식별키를 어떤 방식으로 부여하는지를 결정할 수 있다.

      - AUTO : 특정 데이터베이스에 맞게 자동으로 생성되는 방식
      
      - IDENTITY : 기본 키 생성 방식 자체를 데이터베이스에 위임하는 방식, 데이터베이스에 의존적인 방식, MySQL에서 주로 많이 사용
    
      - SEQUENCE : 데이터베이스의 시퀀스를 이용해서 식별키 생성(오라클에서 사용)

      - TABLE : 별도의 키를 생성해 주는 채번 테이블(번호를 취할 목적으로 만든 테이블)을 이용하는 방식

    - 게시물 작성 시간과 최종 수정 시간을 의미하는 @CreationTimestamp와 @UpdateTimestamp라는 어노테이션을 사용한다.

      - orb.hibernate로 시작하는 패키지의 것입니다.

      - Hibernate의 고유한 기능으로, 엔티티가 생성되거나 업데이트되는 시점의 날짜 데이터를 기록하는 설정입니다.(javax.persistence의 설정이 아니라 Hibernate만의 고유한 설정이지만, 코드의 양을 줄일 수 있는 효과적인 방법입니다.)

<br />

### Step 3. application.properties에 JPA 설정

   - 스프링 부트에서는 별다른 설정이 없어도 기본 패키지 하위에 포함된 패키지들을 자동으로 조사하기 때문에 작성된 엔티티용 클래스나 프로젝트와 관련된 추가적인 설정은 필요하지 않다.

   - JPA와 관련된 약간의 설정을 application.properties 파일에 추가한다.

   ```XML
    # 스키마 생성(create)
    spring.jpa.hibernate.ddl-auto=create
    # DDL 생성 시 데이터베이스 고유의 기능을 사용하는가? 
    spring.jpa.generate-ddl=false
    # 실행되는 SQL문을 보여줄 것인가?
    spring.jpa.show-sql=true
    # 데이터베이스는 무엇을 사용하는가? 
    spring.jpa.database=mysql
    # 로그 레벨
    logging.level.org.hibernate=info
    # MySQL
    spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
   ```

   - 설정을 추가한 후에 프로젝트를 실행해보면 스프링 부트가 동작하며서 테이블을 생성하는 SQL문이 동작하는 모습을 볼 수 있습니다.

   - application.properties에 추가된 설정 중 spring.jpa.hibernate.ddl-auto는 데이터베이스에 구조를 생성하는 DDL(Data Definition Language)는 처리하는 옵션을 지정합니다.

     - create : 기존 테이블 삭제 후 다시 생성

     - create-drop : create와 같으나 종료 시점에 테이블 DROP

     - update : 변경된 부분만 반영

     - validate : 엔티티와 테이블이 정상적으로 매핑되었는지만 확인

     - none : 사용하지 않음

  - spring.jpa.show-sql은 프로그램을 실행할 떄 동작하는 SQL문의 로그를 보여주기 때문에 개발을 할 때 true를 지정하고 사용하는 것이 좋습니다.

## NOTE
  
  - 테이블을 수동 생성할 것인가? 자동 생성할 것인가?

    - JPA로 프로젝트를 하기 위해서는 테이블 생성을 자동으로 할 것인지, 수동으로 할 것인지 고민해야만 합니다.

    - 일반적인 JDBC난 MyBatis 등을 이용한다면 수동으로 테이블을 생성하지만, JPA는 자동으로 테이블을 생성할 수 있습니다.

    - 큰 규모의 프로젝트라면 테이블을 별도로 생성하고, 코드를 작성하는 것이 일반적이지만, JPA는 학습 곡선이 가파르기 때문에 익숙하지 않은 상태라면 테이블 생성을 create 혹은 update 등을 이용해서 처리하고, 충분히 연습한 후에 테이블을 생성하고 코드를 작성하는 방식으로 익숙해지는 것이 좋다.

<br />

## 2.2.3 JPA를 담당하는 Repository 인터페이스 설계

  - Spring Data JPA를 이용하는 경우네는 별도의 클래스 파일을 작성하지 않고 원하는 인터페이스를 구현하는 것만으로 JPA와 관련된 모든 처리가 끝나게 된다.

  - 일반적으로 과거에는 DAO라는 개념을 이용했듯이, JPA를 이용하는 경우에는 Repository라는 용어를 사용한다.

  - 데이터베이스와 관련된 처리를 전통적인 JPA 처리 방식대로 EntityManager를 구성하고, 트랜잭션을 시작하고 종료하는 코드를 만들 수 있지만, Spring Data JPA는 기능이 아주 복잡한 상황이 아닌 이상 간단하게 처리할 수 있는 Repository를 구성하는 것만으로도 충분한 기능들을 제공하고 있다.

  - Spring Data JPA에서 사용하는 인터페이스 구조

    <img src="https://user-images.githubusercontent.com/63120360/181425915-513d5f90-5c86-4cb2-a53e-57e776eec979.png">

    - 모든 인터페이스가 <T, ID>의 두 개의 제네릭 타입을 사용하는 것을 볼 수 있다.

       - T는 엔티티의 타입 클래스

       - ID는 식별자(PK)의 타입

       - ID에 해당하는 타입은 반드시 java.io.Serializable 인터페이스 타입어야만 한다.

    - 가장 상위의 Repository는 사실상 아무 기능이 없기 때문에 실제 주로 사용하는 것은 CRUD 작업을 위주로 하는 CrudRepository 인터페이스와 페이징 처리, 검색 처리 등을 할 수 있는 PagingAndSortingRepository 인터페이스가 있다.

<br />

## NOTE 

  - PagingAndSortingRepository의 하위 인터페이스 JpaRepository

    - JpaRepositry는 PagingAndSortingRepository의 하위 인터페이스로, JPA에 특화된 몇 개의 기능을 추가적으로 가지고 있다.
    
    <br />

    <img src="https://user-images.githubusercontent.com/63120360/181426568-4337a38c-e73c-48a1-ad5a-34cdc69e37dc.png">

    <br />
    
    - 일반적으로 단순한 작업을 위주로 한다면 CrudRepositry를 이용하고, JpaRepository를 굳이 써야하는 상황은 많지 않다.

<br />

### Repository 인터페이스

  ```Java
  public interface BoardRepository extends CrudRepository<Board, Long> {
    
  }
  ```

  - Spirng Data JPA를 이용하면 코드만으로는 아무것도 동작하지 않겠지만, 이와 같은 인터페이스를 기준으로 해서 동적으로 실행할 수 있는 클래스를 동적으로 생성합니다.

```
  * CrudRepository의 메소드

    - long count() : 모든 엔티티의 개수

    - void delete(ID) : 식별키를 통한 삭제

    - void delete(Iterable<? extends T>) : 주어진 모든 엔티티 삭제

    - void deleteAll : 모든 엔티티 삭제

    - boolean exists(ID) : 식별키를 가진 엔티티가 존재하는지 확인

    - Iterable<T> findAll() : 모든 엔티티 목록

    - Iterable<T> findAll(Iterable<ID>) : 해당 식별키를 가진 엔티티 목록 반혼

    - T findOne(ID) : 해당 식별키에 해당하는 단일 엔티티 반환
    
    - <S extends T>Iterable<S> save(Iterable<S>) : 해당 엔티티들의 등록과 수정

    - <S extends T>S save(S entity) : 해당 엔티티의 등록과 수정
```
  - 개발 시에는 상속받은 기능에 필요한 메소드를 Repository 인터페이스에 추가하는 형태로 개발하게 된다.

<br />

### 2.2.4 엔티티 테스트

  - 스프링 부트에서는 프로젝트 생성 시점에서 이미 테스트와 관련되 ㄴ 구성이 완료되기 때문에 별도의 작업 없이 바로 테스트 코드를 추가해서 작업할 수 있다.

  - 테스트 코드에는 클래스 선언 밑에 BoardRepository를 주입하고, @Test 어노테이션이 붙은 테스트 코드를 작성하면 된다.

  ```Java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class BaordRepositoryTests {
    }
  ```

### BoardRepository의 실체

  - 우선적으로 BoardRepository 인터페이스를 구현한 객체를 주입한다.

  - 아래 코드는 인터페이스만 선언된 상태에서 만들어진 클래스의 정보를 조사하기 위한 목적이다.

  ```Java
  @RunWith(SpringRunner.class)
    @SpringBootTest
    public class BaordRepositoryTests {

        @Autowired
        private BoardRepository boardRepository;

        @Test
        public void inspect() {
            // 실제 객체의 클래스 이름
            Class<?> clz = boardRepository.getClass();

            System.out.println(clz.getName());

            // 클래스가 구현하고 있는 인터페이스 목록
            Class<?>[] interfaces = clz.getInterfaces();

            Stream.of(interfaces).forEach(inter -> System.out.println(inter.getName()));

            // 클래스의 부모 클래스
            Class<?> superClass = clz.getSuperclass();

            System.out.println(superClass.getName());
        }
    }
  ```

  ```
    실행결과
    
    com.sun.proxy.$Proxy111
    
    org.zerock.boot2.persistence.BoardRepository
    
    org.springframework.data.repository.Repository
    
    org.springframework.transaction.interceptor.TransactionalProxy
    
    org.springframework.aop.framework.Advised
    
    org.springframework.core.DecoratingProxy
    
    java.lang.reflect.Proxy
  ```

  - 실제 클래스가 com.sum.proxy .. 로 시작하는 경우에는 Java 언어의 Dynamic Proxy(동적 프록시) 기능에 의해 동적으로 생성된 클래스를 의미한다.

  - 클래스 이름의 중간에 있는 '$'는 주로 내부 클래스임을 알려준다.

  - BoardRepository 인터페이스를 구현한 객체라는 사실을 파악할 수 있다.

<br />

### 등록 작업 테스트

  - CrudRepository 인터페이스는 save() 라는 메소드를 이용해서 데이터베이스에 인티티의 정보를 추가하거나 수정하는 작업을 한다.

  - 동일한 메소드로 insert와 update 작업도 이루어지는데 이부분을 이해하기 위해서는 엔티티 매니저의 존재를 인식할 필요가 있다.

     - Repository 쪽에서 save()라는 메소드가 호출되면, 내부에서는 엔티티 매니저가 영속 컨텍스트에 해당 식별키를 가진 엔티티가 존재하는지를 확인한다.

     - 만일 동일한 식별 데이터를 가지는 엔티티가 없다면 엔티티 매니저는 영속 컨텍스트에 저장하고, 데이터베이스에 추가하게 된다.

     - 식별 데이터를 가지는 엔티티가 이미 존재한다면 메모리에 보관되는 엔티티를 수정하는 작업과 데이터베이스를 갱신(update)하는 작업을 진행한다.

    ```Java
        @Test
        public void testInsert() {
            Board board = new Board();
            board.setTitle("게시글의 제목");
            board.setContent("게시글 내용 넣기 ...");
            board.setWriter("user00");
            boardRepository.save(board);
        }
    ```

    - testInsert()에 Board 타입의 객체를 하나 생성하고 title, content, writer 속성값만을 지정하고 있다.

    - Board의 식별 데이터인 bno는 자동 생성 방식을 이용하도록 설정되어 있으므로 지정하지 않는다.

<br />

### 조회 작업 테스트

  - 등록 이외에 작업을 시작하기 이전에 반드시 한 가지 명심해야 하는 점은 application.properties파일의 설정을 확인하는 것입니다.

  - 파일에 있는 spring.jpa.hibernate.ddl-auto=create 설정은 매번 테이블이 drop되고 생성되기 때문에 기존의 데이터를 확인할 수 없다는 점이다.

  ```
    # 스키마 생성(create)
    spring.jpa.hibernate.ddl-auto=update
  ```

  - 조회 테스트의 코드는 findOne()를 이용한다.
     
     - findOne()은 파라미터로 식별 데이터를 사용한다.

     - Spring Data JPA는 CrudRepository에서 <T, ID>와 같이 제네릭을 이용하기 때문에 별도의 변환 없이 사용할 수 있다.

  ```Java
    @Test
    public void testRead(){
        Board board = boardRepository.findById(1L).orElseThrow(NullPointerException::new);

        System.out.println(board);
    }
  ```

   - Spring Data JPA는 기본적으로 Hibernate라는 JPA 구현체를 이용합니다.

   - Hibernate는 내부적으로 지정되는 DB에 맞게 SQL문으 생성하는 Dialect(방언)가 존재한다.

      - Dialect는 Hibernate가 다양한 데이터베이스를 처리하기 위해 각 데이터베이스마다 다른 SQL 문법을 처리하기 위해 존재하는 것이다.

      - JPA를 통해서 호출하면 설정된 데이터베이스에 맞게 SQL문이 생성되는데 이 역할을 Dialect라 한다.

      - 내부적으로는 OracleDialect, MySQLDialct 같은 클래스들이 다수 존재한다.

      - application.properties에 데이터베이스의 종류를 지정하면 기본적으로 해당 데이터베이스에 맞는 Dialect가 지정되지만 특정 버전을 명시해 줄 수도 있다.

      ```
        * application.properties의 일부

        spring.jpa.database = mysql
        spring.jpa.database-plateform = org.hibernate.dialect.MySQL5InnoDBDialect
      ```

     - 예를 들어, Hibernate 5.2 버전이 지원하는 Dialect의 목록은 https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/dialect/MySQL5InnoDBDialect 통해 확인할 수 있다.

  - MySQL을 사용하는 경우 MySQL용 Dialect가 동작하면서 앞서 본 것과 같은 SQL문을 생성해낸다.(흔히 ORM을 프로젝트 도입에 꺼리는 가장 큰 이유로, 이와 같이 생성되는 SQL문의 성능 개선을 위한 튜닝 작업의 어려움 등을 들기도 한다.)

  - 조회 작업에는 내부적으로 1차 캐시라는 존재가 있다. 외부에서 특정한 엔티티를 조회하면 내부에서는 1차 캐시 안에 엔티티가 존재하는지를 살펴보고 없는 경우에 SQL문을 통해 데이터베이스에 가져오게 된다.

<br /> 

### 수정 작업 테스트

  - 수정 작업은 등록 작업과 동일하게 save() 메소드를 호출한다. 

  ```Java
    @Test
    public void testUpdate(){
        System.out.println("Read First ... ");
        Board board = boardRepository.findById(1L).orElseThrow(NullPointerException::new);

        System.out.println("Update Title ... ");
        board.setTitle("updated title");

        System.out.println("Call Save() ... ");
        boardRepository.save(board);
    }
  ```

  - JPA는 데이터베이스에 바로 작업을 하는 JDBC와 달리 스스로 엔티티 객체들을 메모리상에서 관리하고, 필요한 경우에 데이터베이스에 작업을 하게 된다. 

     - 수정과 삭제 작업은 직접 데이터베이스에 바로 SQL문을 실행하는 것이 아니라, 엔티티 객체가 우선적으로 메모리상에 존재하고 있어야 한다.

     - 테스트 코드는 매번 새롭게 실행되기 때문에 관리되고 있는 엔티티 객체가 없어서 현재 작성하는 수정과 삭제 작업에는 'select'가 우선으로 실행된다.

     - 만일 조회한 엔티티와 수정된 엔티티가 동일한 값들을 가지고 있어서 수정할 필요가 없다면 'update'문은 실행되지 않는다.

<br />

### NOTE
``` 
  - testUpdate() 메소드는 처음 실행할 때와 반복해서 실행할 때의 결과에 차이가 있다. 처음 실행하면 컨텍스트상에 엔티티의 상태가 변하기 때문에 데이터비에스에도 업데이트가 실행되지만, 이후에는 데이터베이스와 동일하기 때문에 업데이트가 실행되지 않는다.
```

### 삭제 작업 테스트

  - Spring Data JPA에서 delete()를 이용해서 삭제 처리를 한다.

  - delete() 시에 파라미터
     
     1. 식별키 값을 파라미터로 전달

     2. 삭제하려는 엔티티 객체를 전달

  - 어떤 형식을 이용하든 삭제는 식별 데이터를 이용해서 SQL문을 구성한다.

<br />

  - 삭제 작업 테스트 결과는 수정과 마찬가지로 삭제하기 전ㄴ에 엔티티 객체가 관리되지 않았기 때문에 'select'를 통해서 엔티티 객체를 보관하고 이후에 'delete'가 실행되는 것을 확인할 수 있다.

<br />

### NOTE
```
application.properties 파일에 loggin.level.org.hibernate=info 설정을 debug로 변경한 후에 코드를 테스트하면 로그를 자세히 볼 수 있다.
```