# 3.2 @Query 이용하기

  - 쿼리 메소드가 편리하게 원하는 쿼리를 작성할 수 있는 방법을 제공하기는 하지만, 현실적으로 데이터베이스에 'where bno > 0'와 같이 항상 사용되는 SQL 구문 등을 처리하기에 조금은 어색한 면이 있다.

  - 데이터베이스를 이용해서 쿼리를 수행할 때에는 실행계획(Excute Plan)이라는 것을 참고한다.

    - 실행계획은 데이터베이스상에서 해당 SQL이 어떤 방식으로 실행되는지를 말해준다.

    - (예시) 'select * from btl_boards order by bno desc'와 같은 SQL문을 실행하면 데이터베이스는 bno를 역순으로 정렬해야 한다.

    - bno가 PK로 지정되지 않을 때 데이터를 하나씩 꺼내서 bno의 순서대로 정렬을 하게 되면 엄청나게 많은 시간을 소비할 수 밖에 없다.

    - bno가 PK로 지정되었을 경우 색인 작업(indexing)을 통해서 이미 렐코드의 순서가 정해지게 된다. 이를 통해 데이터베이스에서는 정렬이 필요할 때 이미 정리된 상태의 색인(index)을 이용해서 데이터에 접근하게 된다.

    <img src="https://user-images.githubusercontent.com/63120360/182550102-e46408ed-2b46-40eb-83fc-89d4ea008d32.png">

    <br />

    - MYSQL 상에서 'where board0_.bno>? order ~ asc'로 끝나는 SQL에 대한 실행계획은 다음과 같다.

    <br />

    <img src="https://user-images.githubusercontent.com/63120360/182557975-af5fd640-9a2f-409d-9f5b-54761756b213.png">

    - 테이블을 통해서 데이터에 접근하지 않고, 이미 순서가 만들어진 인덱스를 이용해서 접근하고 있다.

    - SQL을 작성할 때에는 보다 최적화된 실행계획을 세울 수 있도록 SQL문을 조정할 필요가 있다. 

       - 예를들어, where bno > 0 이라는 조건을 주는 것과 주지 않는 경우를 비교해 보면 실행계획이 달라진다.

    <br />

    <img src="https://user-images.githubusercontent.com/63120360/182560537-9134febc-0168-4fd4-9301-856d21774126.png">

    <br />

    - 'where bno > 0'이라는 조건을 주지 않으면 테이블 전체를 스캔하는 것을 볼 수 있다.

    - 'where bno > 0'조건이 있는 경우는 PK를 이용해서 데이터를 검색해 나갈 수 있다.

    - SQL을 작성하다 보면 기본적으로 'Full Table Scan(테이블 전체 검색)'은 모든 데이터에 대한 검색을 통해서 처리되기 때문에 성능이 나쁘다고 인식한다.(병렬처리가 효과적으로 처리되면 더 빠른 경우도 있다.)

    - 'where bno > 0'과 같은 조건의 여부에 따라서 이처럼 실행계획인 결정되지만, 쿼리 메소드에서 이러한 조건을 표현하는 것은 어색하고도 불편하다.

    - 구체적인 조건등을 지정하기 위해 Spring Data JPA는 쿼리 메소드 대신에 @Query(쿼리 어노테이션)을 이용할 수 있다.

    - @Query를 이용하면 말 그대로 어노테이션 안에 쿼리를 작성할 수 있다.
    
    - 쿼리는 JPQL이라는 JPA에서 사용하는 쿼리 문법을 이용하거나, 순수한 데이터베이스에 맞는 SQL을 사용할 수 있다.

<br />

## 3.2.1 단순 게시물의 처리를 위한 @Query 작성

  - (예시) 게시물은 일반적으로 '제목(title), 내용(content) 또는 작성자(writer)'에 대한 검색이 반드시 필요하다.

### 1. 제목에 대한 검색 처리

  ```Java
  @Query("SELECT b FROM Board b WHERE b.title LIKE %?1% AND b.bno > 0 order by b.bno DESC")
    public List<Board> findByTitle(String title);
  ```
  
  - @Query에는 JPQL(객체 쿼리)을 이용한다.

     - JPQL은 JPA에서 사용하는 Query Language

     - SQL과 유사한 구문들로 구성되고, JPA의 구현체에서 이를 해석해서 실행한다.

     - JPQL의 가장 기본적인 형태는 테이블 대신에 엔티티 타입을 이용한다는 것과 칼럼명 대신에 엔티티의 속성을 이용하는 것이다.

     - SQL에서 많이 사용하는 order by 또는 group by 등은 기본적으로 지원한다.

     - '%?1%'에서 '?'는 JDBC상에서 PrepareStatement에서 사용한 것과 동일한 것이고, '?1'은 첫 번째로 전달되는 파라미터라고 생각하면 된다.

    
    ```Java
    	@Test
	public void testByTitle2(){
		boardRepository.findByTitle("17").forEach(board -> System.out.println(board));
	}
    ```

    ```sql
    select board0_.bno as bno1_0_, board0_.content as content2_0_, board0_.regdate as regdate3_0_, board0_.title as title4_0_, board0_.updatedate as updateda5_0_, board0_.writer as writer6_0_ from tbl_boards board0_ where (board0_.title like ?) and board0_.bno>0 order by board0_.bno DESC
    ```

    <img src="https://user-images.githubusercontent.com/63120360/182565730-b5c5ce53-288e-451b-b6c3-22d23052c49c.png">

<br />

### 2. 내용에 대한 검색 처리 - @Param

  - 내용에 대한 검색 처리는 제목에 대한 검색 처리와 동일한 방식으로 처리할 수 있다.

  ```Java
      @Query("SELECT b FROM Board b WHERE b.content LIKE %:content% AND b.bno > 0 ORDER BY b.bno DESC")
    public List<Board> findByContent(@Param("content") String content);
  ```

  - 작성된 코드를 보면 기존의 검색 처리 결과와 달리 '%:content%'와 같이 처리되는 것을 볼 수 있습니다.

  - 파라미터에서는 @param이라는 어노테이션을 이용하는 것을 볼 수 있습니다.

  - @Param은 org.springframework.data.repository.query.Param 클래스를 이용한다.

  - 이를 통해 여러 개의 파라미터를 전달할 때 이름을 이용해 쉽게 구분해서 전달할 수 있다.

### 3. 작성자에 대한 검색 처리("#{entityName}")

  - 작성자에 대한 검색 코드는 두 가지 검색 방식을 이용해서 작성할 수 있다.


  ```Java
  @Query("SELECT b FROM #{#entityName} b WHERE b.writer LIKE %?1% AND b.bno > 0 ORDER BY b.bno DESC")
  List<Board> findByWriter(String writer);
  ```

  - "#{#entityName}"은 Repository 인터페이스를 정의할 때 <엔티티 타입, PK타입>에서 엔티티 타입을 자동으로 참고한다.

  - 유사한 상속 구조의 Repository 인터페이스를 여러 개 생성하는 경우 유용하게 사용할 수 있다.

<br />

## 3.2.2 @Query의 활용

  - 리턴 값이 반드시 엔티티 타입이 아니라 필요한 몇 개의 칼럼 값들만 추출할 수 있다.

     엔티티 타입의 경우 테이블의 모든 칼럼을 조회하지만, JPQL을 이용하면 필요한 몇 개의 컬럼들만을 조회할 수 있다.

  - nativeQuery 속성을 지정해서 데이터베이스에 사용하는 SQL을 그대로 사용할 수 있다.

     SQL 자체를 그대로 사용하고 싶을 때나 별도의 SQL에 대한 튜닝이 이루어진 경우에 유용하게 사용할 수 있다.

  - Repository에 지정된 에티티 타입뿐 아니라 필요한 엔티티 타입을 다양하게 사용할 수 있다.

     JPQL의 경우 여러 엔티티 타입을 조회할 수 있기 때문에 한 번에 여러 엔티티 타입을 조회하는 경우에 유용하게 사용할 수 있다.

### 1. 필요한 컬럼만 추출하는 경우

  - @Query를 이용하면 'select'로 시작하는 JPQL 구문을 작성하는데, 이때 필요한 칼럼만을 추출할 수 있다.

     - 예들 들어, content 칼럼의 내용을 제외한 칼럼을 가져오게 해라

       ```Java
           @Query("SELECT b.bno, b.title, b.writer, b.regdate "+" FROM Board b WHERE b.title LIKE %?1% AND b.bno > 0 ORDER BY b.bno DESC ")
        public List<Object[]> findByTitle2(String title);
       ```

     - 여러 칼럼을 지정하는 경우 특이한 점은 리턴 타입이 엔티티 타입이 아니라, Object 타입의 리스트 형태여야 한다.

     ```Java
     @Test
	public void testByTitle17(){
		boardRepository.findByTitle2("17").forEach(arr -> System.out.println(Arrays.toString(arr)));
	}
     ```

### NOTE
```
@Query를 이용할 때 주의해야 하는 점은 @Query에 대한 해석이 프로젝트 로딩 시점에 이루어진다 점입니다. 즉 @Query의 내용은 프로젝트가 시작되면서 검증되기 때문에 만일 @Query의 내용물이 잘못된 경우에는 프로젝트가 정상적으로 실행되지 않으니 주의해야 합니다.
```

<br />

### nativeQuery

  - @Query는 말 그대로 데이터베이스에 종속적인 SQL문을 그대로 사용할 수 있기 때문에 복잡한 쿼리를 작성할 때 유용하게 사용할 수 있다.

    - 데이터베이스에 독집적이라는 장점을 어느 정도 포기해야 하므로 남용하지 않도록 주의해야 한다.

  - @Query에 nativeQuery 속성을 true로 지정하면 메소드 실행 시 @Query의 value 값을 그대로 실행하게 된다.

  ```Java
      @Query(value = "select bno, title, writer from tbl_boards where title like CONCAT('%', ?1, '%') and bno > 0 order by desc", nativeQuery = true)
    public List<Object[]> findByTitle3(String title);
  ```

### 3.2.3 @Query와 Paging 처리/정렬

  - @Query를 이용하더라도 페이징 처리를 하는 Pageable 인터페이스는 그대로 활용 가능하다.

  - 만일 메소드의 파라미터에 Pageable 타입을 사용하게 되면 '@Query로 작성한 내용 + 페이징 처리'의 형태가 된다.

    - (예시) 전체 게시물에 대한 페이징 처리

      ```Java
          // 전체 게시물에 대한 페이징처리
        @Query("SELECT b FROM Board b WHERE b.bno > 0 ORDER BY b.bno DESC ")
        public List<Board> findBypage(Pageable pageable);
      ```

     - @Query에 다른 조건은 없고 단지 페이징 처리에 필요한 'bno > 0' 조건과 'order by'조건만을 부여하고 있다.

    ```Java
    @Test
	public void testByPaging(){
		PageRequest paging = PageRequest.of(0, 10);

		boardRepository.findBypage(paging).forEach(board -> System.out.println(board));
	}
    ```

    - Spring Data JPA를 이용하면 약간의 처리만으로도 SQL이나 JDBC 코드 작성 없이 원하는 결과를 얻을 수 있다.

    - 편리함은 관계가 복잡할수록 난이도가 높아진다.
