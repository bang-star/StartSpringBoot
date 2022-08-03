# 03. Spring Data JPA를 이용한 단순 게시물의 처리

  - Spring Data JPA의 CrudRepository만을 이용해서 단순한 create, read, update, delete 등의 작업은 가능하다.

  - 주요 학습 내용
    
    - 쿼리 메소드라는 메소드의 이름만으로 원하는 SQL을 실행하는 방법

    - @Query를 이용한 구체화된 JPQL 처리

    - 페이징과 정렬에 대한 처리

    - Querydsl을 이용한 동적 쿼리 

<br />

## 3.1 쿼리 연습을 위한 준비

  - JPA에서는 각 데이터베이스에 맞는 Dialect가 별도의 SQL에 대한 처리를 자동으로 처리해 주기 때문에 개발 시 생산성을 향산시킬 수 있다.

  - 조금 복잡한 쿼리를 작성하기 위해서는 데이터베이스를 대상으로 하는 SQL이 아니라 JPA에서 사용하는 Named Query 또는 JPAL(Java Persistence Query Language), Query dsl이라는 것을 학습할 필요가 있다.

  - Spring Data JPA는 번거로운 과정을 줄여줄 수 있는 쿼리 메소드라는 기능이 있다.

    - 쿼리 메소드는 메소드의 이름만으로 필요한 쿼리를 만들어 내느 기능으로, 별도의 학습 없이 네이밍 룰만 알고 있어도 사용할 수 있다.

<br />

```Java
@RunWith(SpringRunner.class)
@SpringBootTest
class Boot3ApplicationTests {

	@Autowired
	private BoardRepository boardRepository;

	@Test
	public void testInsert200(){
		for(int i=0; i<200; i++){
			Board board = new Board();
			board.setTitle("title - "+i);
			board.setContent("content - "+i);
			board.setWriter("writer - "+i);
			boardRepository.save(board);
		}
	}
}
```

### 3.1.1 쿼리 메소드 이용

  - Spring Data JPA는 메소드의 이름만으로 원하는 질의(query)를 실행할 수 있다.

     - 쿼리라는 용어는 'select'에만 해당한다는 것에 주의해야 한다.

     ```Java
     find...By...
     read...By...
     query...By...
     get...By...
     count...By...
     ```

     - 중간에 엔티티 타입을 지정하지 않는다면 현재 실행하는 Repository의 타입 정보를 기준으로 동작한다.

     - find와 By 중간에 엔티티 타입을 지정한다.(이하 동일)

        예시) findBoardBy

     - By 뒤에는 칼럼명을 이용해서 구성한다. 

        예시) findBoardByTitle

      - 쿼리 메소드의 리턴타입
        
        ```
        Page<T>
        Slice<T>
        List<T>
        ```

        - 위와 같이 Collection의 형태가 될 수 있다. 

        - 가장 많이 사용되는 것은 List, Page 타입이다.
    
    <br />

    예시 코드
    ```Java
    public interface BoardRepository extends CrudRepository<Board, Long> {

        public List<Board> findBoardByTitle(String title);

    }
    ```

    <br />

    ```Java
    	@Test
	public void testByTitle(){
		// before Java 8
        // List<Board> list = boardRepository.findBoardByTitle("title...177");
        //
        //for(int i=0; i<list.size();i++){
        //	System.out.println(list.get(i));
        //}

		// Java 8
		boardRepository.findBoardByTitle("title...177").forEach(board -> System.out.println(board));
	}
    ```

    - 실행되는 SQL문을 통해 원하는 결과가 제대로 불려오는지 확인할 수 있다.

    - 쿼리 메소드 작성하는 방법은 Spring Data JPA 문서를 통해 자세히 알 수 있다.

    <img src="https://websparrow.org/wp-content/uploads/2020/03/spring-data-jpa-derived-query-methods-example-1.png">

    <br />

    <img src="https://websparrow.org/wp-content/uploads/2020/03/spring-data-jpa-derived-query-methods-example-2.png">


<br />

### findBy를 이용한 특정 칼럼 처리

  - SQL문에서 특정한 칼럼의 값을 조회할 때는 메소드의 이름을 findBy로 시작하는 방식을 이용한다.

  ```
   Collection<T> findBy + 속성 이름(속성 타입)
  ```

  - 예시
    
    - 게시물에서 user00이라는 작성자의 모든 데이터를 구한다는 기능을 BoardRepository에 추가한다면?

      
    ```Java
    public interface BoardRepository extends CrudRepository<Board, Long> {

        public List<Board> findBoardByTitle(String title);

        public Collection<Board> findBoardByWriter(String writer);
    }
    ```

    - findBy ... 로 시작하는 쿼리 메소드는 지정하는 속성 값에 따라 파라미터의 타입이 결정된다.

        - Board 클래스에서 writer 속성의 값이 문자열이기 때문에 파라미터 타입은 String으로 지정


### Like 구문 처리

  - findBy와 더불어서 가장 많이 사용하는 구문은 like 구문이다.

  - like에 대한 처리
     
     1. 단순한 like

     2. 키워드 + '%' 

     3. '%' + 키워드

     4. '%' + 키워드 + '%'


    <img src="https://user-images.githubusercontent.com/63120360/182284525-e3c5e9ee-7065-443d-b107-f3bc1f4a6ec7.png">

    - 예시

       - 게시물에서 작성자 이름에 '05'라는 문자가 들어 있는 게시글을 검색하자.

       ```Java
        public interface BoardRepository extends CrudRepository<Board, Long> {

            public List<Board> findBoardByTitle(String title);

            public Collection<Board> findBoardByWriter(String writer);
    
            // 작성자에 대한 like % 키워드 %
            public Collection<Board> findBoardByWriterContaining(String writer);
        }
       ```

       ```Java
       	@Test
        public void testByWriterContaining(){
            Collection<Board> results = boardRepository.findBoardByWriterContaining("05");
            // for 루프 대신 foreach
            results.forEach(board -> System.out.println(board));
	    }
       ```

### and 혹은 or 조건 처리

  - 경우에 따라 2개 이상의 속성을 이용해서 엔티티들을 검색해야 할 때 쿼리 메소드 'And'와 'Or'를 사용한다.

     - 속성이 두 개 이상일 때는 파라미터 역시 지정한 속성의 수만큼 맞춰주어야 한다.

     - 예를들어, 게시글의 title과 content 속성에 특정한 문자열이 들어있는 게시물을 검색하려면 'findBy'+'TitleContaining'+'Or'+'ContentContaining'과 같은 형태가 된다.

     ```Java
        public interface BoardRepository extends CrudRepository<Board, Long> {

        public List<Board> findBoardByTitle(String title);

        public Collection<Board> findBoardByWriter(String writer);

        // 작성자에 대한 like % 키워드 %
        public Collection<Board> findBoardByWriterContaining(String writer);
        
        // OR 조건 처리
        public Collection<Board> findByTitleContainingOrContentContaining(String title, String content);
        }
     ```


### 부등호 처리

  - 쿼리 메소드에서는 '>'와 '<' 같은 부등호는 'GreaterThan'과 'LessThan'을 이용해서 처리할 수 있다.

    - 예를 들어, 게시물의 title에 특정한 문자가 포함되어 있고 bno가 특정 숫자 이상인 데이터를 조회해라.

    ```Java
    public interface BoardRepository extends CrudRepository<Board, Long> {

        public List<Board> findBoardByTitle(String title);

        public Collection<Board> findBoardByWriter(String writer);

        // 작성자에 대한 like % 키워드 %
        public Collection<Board> findBoardByWriterContaining(String writer);

        // OR 조건 처리
        public Collection<Board> findByTitleContainingOrContentContaining(String title, String content);
        
        // title LIKE % ? % AND BNO > ?
        public Collection<Board> findByTitleContainingAndBnoGreaterThan(String keyword, Long num);
    }

    @Test
	public void testByTitleAndBno(){
		Collection<Board> results = boardRepository.findByTitleContainingAndBnoGreaterThan("5", 650L);

		results.forEach(board -> System.out.println(board));
	}
    ```

### order by 처리

  - 가져오는 데이터의 순서를 지정하기 위해서는 'OrderBy'+속성+'Asc or Desc'를 이용해서 작성할 수 있다.

     - (예시) 게시물의 bno가 특정 번호보다 큰 게시물을 bno 값의 역순으로 출력하라.

<br />     

## 3.1.2 페이징 처리와 정렬

  - 쿼리 메소드는 마지막 파라미터로 페이지 처리를 할 수 있는 Pageable 인터페이스와 정렬을처리하는 Sort 인터페이스를 사용할 수 있다.

    - Pageable 인터페이스는 페이징 처리에 필요한 정보를 제공

        - org.springframework.data.domain.Pageable 인터페이스를 구현한 클래스 중에 PageRequest 클래스를 많이 사용한다.

        - (예시) 'bno > 0 order by bno desc'라는 조건을 구현한 findByBnoGreaterThanOrderByBnoDesc() 메소드에 Pageable을 적용해라.

        ```Java
        // bno > ? ORDER BY bno DESC limit ?, ?
        public List<Board> findByBnoGreaterThanOrderByBnoDesc(Long bno,    Pageable paging);
        ```

       - 코드는 동일하지만, 파라미터에 Pageable이 적용되어 있고, 리턴  타입으로 Collection 대신 List로 변경되었다.

       - Pageable 인터페이스가 적용되는 경우, 리턴 타입은 3가지다.
       
            - org.srpingframework.data.domain.Slice 타입
            
            - org.springframework.data.domain.Page 타입

            - java.util.List 타입

        ```Java
        // bno > ? ORDER BY bno DESC limit ?, ?
        public List<Board> findByBnoGreaterThanOrderByBnoDesc(Long bno, Pageable paging);

        @Test
        public void testBnoOrderByPaging(){
            Pageable paging = PageRequest.of(0, 10);

            Collection<Board> results = boardRepository.findByBnoGreaterThanOrderByBnoDesc(0L, paging);

            results.forEach(board -> System.out.println(board));
        }
        ```

        - SQL을 보면 MySQL이기 때문에 자동으로 limit가 적용된다.

        <br />

     - 정렬처리에는 Pageable 인테이스와 같이 Sort 클래스를 이용한다. 
        
        - Sort는 쿼리 메소드에서 OrderBy로 처리해도 되지만, Sort를 이용하면 원하는 방향으로 파라미터를 결정할 수 있다.

        - PageRequest 생성자

           - PageRequest(int page, int size) : 페이지 번호(0부터 시작), 페이지당 데이터의 수
           - PageRequest(int page, int size, Sort.Direction direction, String ... props) : 페이지 번호, 페이지당 데이터의 수, 정렬 방향, 속성(칼럼)들
            - PageRequest(int page, int size, Sort sort) : 페이지 번호, 페이지당 데이터의 수, 정렬 방향
        
    - PageRequest의 생성자 중에는 Sort 인터페이스 타입을 파라미터로 전달할 수 있는 생성자가 있다. 이를 이용하면 페이징 처리와 정렬에 대한 처리를 동시에 지정할 수 있다.

<br />

## 3.1.3 Page<T> 타입
  
  - Spring Data JPA에서 결과 데이터가 여러 개인 경우 List<T> 타입을 이용하기도 하지만, Page 타입을 이용하면 Spring MVC와 연동할 때 상당한 편리함을 느낄 수 있다.

  - Page 타입은 단순 데이터만을 추출하는 용도가 아니라, 흔히 웹에서 필요한 데이터들을 추가적으로 처리해 준다.

    <img src="https://user-images.githubusercontent.com/63120360/182401918-421c6dd7-d964-44c4-a2f3-48e56a1d5d0a.png">

  - SQL문이 두번 실행된다(만일 데이터의 수가 적어서 지정된 사이즈 이하인 경우에는 한 번만 실행된다.)

    - 첫 번째 SQL은 데이터를 추출하기 위해 실행

    - 두 번째 SQL은 데이터의 개수를 파악하기 위해서 select count()가 실행

    - 리턴 타입을 Page로 하게 되면 웹 페이징에서 필요한 데이터를 한 번에 처리할 수 있기 때문에 데이터를 위한 SQL과 개수를 파악하기 위한 SQL을 매번 작성하는 불편함이 없어진다.

## 참조

  - [SpringDataJPAMethod](https://websparrow.org/spring/spring-data-jpa-derived-query-methods-example)


