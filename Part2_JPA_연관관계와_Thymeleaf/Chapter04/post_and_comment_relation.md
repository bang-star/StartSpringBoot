# 4.4 게시물과 댓글의 관계 - 양방향

  - 양방향 참조는 관계 있는 객체들의 참조를 양쪽에서 가지고 있는 것이다.

    - 단뱡항에 비해서 설정은 수월하지만 데이터의 관리 측면에서 조금 더 신경 써야 하는 점들이 존재한다.

  - 게시물과 댓글의 관계는 전형적인 '일대다', '다대일'이다.

     - 데이터베이스상의 관계는 동일하지만, JPA를 이용하는 경우에는 동일하지 않습니다.

    <br />

    <img src="https://user-images.githubusercontent.com/63120360/183348947-98e8fc5b-b98c-43a3-bcb5-01f84441d286.png">

  <br />

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_freeboards")
    @EqualsAndHashCode(of = "bno")
    public class FreeBoard {

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

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_free_replies")
    @EqualsAndHashCode(of = "rno")
    public class FreeBoardReply {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long rno;
        private String reply;
        private String replyer;

        @CreationTimestamp
        private Timestamp replydate;
        @UpdateTimestamp
        private Timestamp updatedate;
    }
  ```

## 4.4.1 연관관계의 설정

  - 게시물과 댓글의 관계는 '일대다', '다대일'로 지정하고, 양방향으로 지정할 것이므로, FreeBoard 클래스와 FreeBoardReply 클래스에 상호 참조가 가능하도록 설정해 주어야 합니다.

  - FreeBoard 클래스는 ERD 상의 선분으로, '일대다' 관계이므로 @OneToMany를 이용해서 지정한다.

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_freeboards")
    @EqualsAndHashCode(of = "bno")
    public class FreeBoard {

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

        @OneToMany
        private List<FreeBoardReply> replies;
    }
  ```
  
  - FreeBoardReply는 '다대일'의 관계이므로 @ManyToOne을 지정한다.

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_free_replies")
    @EqualsAndHashCode(of = "rno")
    public class FreeBoardReply {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long rno;
        private String reply;
        private String replyer;

        @CreationTimestamp
        private Timestamp replydate;
        @UpdateTimestamp
        private Timestamp updatedate;
        
        @ManyToOne
        private FreeBoard board;
    }
  ```

  - 이 상태에서 프로젝트를 실행할 경우 3개의 테이블이 생성되는 것을 볼 수 있다.

  ```SQL
    Hibernate: create table tbl_free_replies (rno bigint not null auto_increment, reply varchar(255), replydate datetime, replyer varchar(255), updatedate datetime, board_bno bigint, primary key (rno)) engine=InnoDB
    Hibernate: create table tbl_freeboards (bno bigint not null auto_increment, content varchar(255), regdate datetime, title varchar(255), updatedate datetime, writer varchar(255), primary key (bno)) engine=InnoDB
    Hibernate: create table tbl_freeboards_replies (free_board_bno bigint not null, replies_rno bigint not null) engine=InnoDB
  ```

  - 양쪽 테이블 중간에 지정하지 않은 테이블 하나가 생성되는 이유는 @OneToMany 관계를 저장하려면 중간에 '다(Many)'에 해당하는 정보를 보관하기 위해서 JPA의 구현체는 별도의 테이블을 생성한다.

### mappedBy 속성
  
  - 데이터베이스상에서 관계를 맺는 방법이 PK, FK만을 사용해서 지정하지만, JPA에서 양쪽이 모두 참조를 사용하는 경우에는 어떤 쪽이 PK가 되고, 어떤 쪽이 FK가 되는지를 명시해줄 필요가 있다.

    - JPA에서는 관계를 설정할 때 PK 쪽이 mappedBy라는 속성을 이용해서 자신이 다른 객체에게 '매여있다'는 것을 명시하게 됩니다. 즉, '해당 엔티티가 관계의 주체가 되지 않는다는 것을 명시한다'고 한다.

    - (예시) 댓글이 있는 상태에서 게시글 삭제가 불가능하기 떄문에 '게시글이 댓글에 매여있다'라고 볼 수 있다.

  ```Java
    @OneToMany(mappedBy = "board")
    private List<FreeBoardReply> replies;
  ```

  - 'mappedBy'는 '~에 매이게 된다'이므로 종속적인 클래스의 인스턴스 변수를 지정한다.

     - 현재 FreeBoardReply 클래스에는 board라는 변수가 FreeBoard 인스턴스를 의미하므로, 이를 지정한다.

<br />

### 양방향 설정과 toString()

  - Lombbok을 이용해서 toString()을 자동으로 처리하는 것은 편리한 기능이지만, 양방향 참조를 이용하는 경우에는 양쪽에서 toString()을 실행하기 떄문에, toString()을 반복 실행하는 문제가 생긴다.

  - 양반향 참조를 사용하는 경우에는 반드시 한쪽은 toString()에서 참조하는 객체를 출력하지 않도록 수정해 주어야 합니다.

  ```Java
    @Getter
    @Setter
    @ToString(exclude = "replies")
    @Entity
    @Table(name = "tbl_freeboards")
    @EqualsAndHashCode(of = "bno")
    public class FreeBoard {
        ...
    }

    @Getter
    @Setter
    @ToString(exclude = "board")
    @Entity
    @Table(name = "tbl_free_replies")
    @EqualsAndHashCode(of = "rno")
    public class FreeBoardReply {
        ...
    }

  ```

## 4.4.2 Repository

  - 여러 엔티티들의 Repository 개수를 판단하는 데 가장 중요한 영향을 미치는 것은 역시 엔티티 객체의 라이플사이클입니다. 각 엔티티가 별도의 라이프 사이클을 가진다면 별도의 Repository를 생성하는 것이 좋고, 그렇지 못하다면 Repository 역시 달라지게 된다.

     - 게시물과 댓글의 관계를 생각해 보면 게시물의 작성과 댓글의 작성은 관계없이 별도로 이루어진다는 것을 알 수 있다.

     - 댓글의 수정이나 삭제는 게시물에 영향을 주지 않으므로, 별도의 Repository로 작성하는 것이 좋다.

     ```JAVA
     public interface FreeBoardRepository extends CrudRepository<FreeBoard, Long> {}

     public interface FreeBoardReplyRepository extends CrudRepository<FreeBoardReply, Long> {}
     ```


## 4.4.3 테스트 코드

  - 게시물과 댓글의 관계를 온전하게 처리하기 위해서 여러 가지 상황에 대해서 테스트 코드를 작성할 필요가 있다.

    ```Java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @Log
    @Commit
    public class FreeBoardTests {

        @Autowired
        FreeBoardRepository boardRepository;

        @Autowired
        FreeBoardReplyRepository replyRepository;

        @Test
        public void insertDummy(){
            IntStream.range(1, 200).forEach(i -> {
                FreeBoard board = new FreeBoard();
                board.setTitle("Free Board ... " + i);
                board.setContent("Free Content ... " + i);
                board.setWriter("user" + i%10);

                boardRepository.save(board);
            });
        }
    }
    ```

  - 게시물들이 추가된 상태에서 랜덤하게 댓글을 추가해 보도록 합니다. 게시글에 댓글을 추가하는 방식은 2가지 방식을 사용할 수 있습니다.

     - 단방향에서 처리하듯이 FreeBoardReply를 생성하고, FreeBoard 자체는 새로 만들어서 bno 속성만을 지정하여 처리하는 방식

     - 양방향이므로 FreeBoard 객체를 얻어온 후 FreeBoardReply를 댓글 리스트에 추가한 후에 FreeBoard 자체를 저장하는 방식

     - 양방향을 이용하게 됨녀 어떤 방식으로 접근해야 하는가에 대한 고민을 많이 할 수 밖에 없다. 

     ```Java
    @Transactional
    @Test
    public void insertReply2Way(){

        Optional<FreeBoard> result = boardRepository.findById(199L);

        result.ifPresent(board -> {
            List<FreeBoardReply> replies = board.getReplies();

            FreeBoardReply reply = new FreeBoardReply();
            reply.setReply("REPLY ......");
            reply.setReplyer("replyer00");
            reply.setBoard(board);

            replies.add(reply);

            board.setReplies(replies);

            boardRepository.save(board);
        });
    }
     ```

     - 현재 replies.add()를 실행할 떄 문제가 발생한 것을 알 수 있다. 

        - 에러의 원인은 replies에 add()를 하려면 기존에 어떤 댓글이 존재하는지 확인해야 하는데, 데이터베이스와 연결된 이후에 'select'를 한번 실행해 FreeBoard 객체를 가져와 버렸기 때문에, 추가적으로 다시 연결이 일요한 'insert'를 실행할 수 없게 된 것이다.

         - 해결 방법

         1. 게시물이 저장될 때 댓글이 같이 저장되도록 cascading 처리가 되어야 한다.

         2. 댓글 쪽에도 변경이 있기 때문에 트랜잭션을 처리해 주어야 한다.

         - insertReply2Way()에 @Transactional 처리와 cascade 속성을 지정하면 댓글에 추가됩니다.

    <br />

    ```Java
    @OneToMany(mappedBy = "board", cascade = CascadeType.ALL)
    private List<FreeBoardReply> replies;
    ```

### 단방향 방식의 댓글 추가

   - 양방향으로 댓글을 추가하는 것도 가능하지만, 단방향으로 처리하는 것이 좀 더 쉽게 댓글을 추가할 수 있다. 

   ```Java
    @Test
    public void insertReply1Way(){
        // 단방향 댓글

        FreeBoard board = new FreeBoard();
        board.setBno(199L);

        FreeBoardReply reply = new FreeBoardReply();
        reply.setReply("REPLY ...... ");
        reply.setReplyer("replayer00");
        reply.setBoard(board);

        replyRepository.save(reply);
    }
   ```

    - 실행 결과를 보면 양방향과 달리 한 번의 'insert'만이 실행되는 것을 볼 수 있다.

<br />

## 4.4.4 게시물의 페이징 처리와  @Query

  - 양방향 처리는 하나의 엔티티 객체를 이용해서 다른 엔티티를 서로 참조하는 관계이므로, 단방향의 제한적인 접근에 비해 운용의 폭이 넓다.

  - 양방향으로 원하는 데이터들을 얻을 수 있다고 하더라도, 최종적으로 실행되는 SQL이 성능에 나쁜 영향을 주는지를 항상 체크해 주어야 한다.

  - 게시물과 댓글의 관계에서 자주 사용하는 것을 페이징 처리이므로, 다음과 같은 상황을 가정해보자.

      - 쿼리 메소드를 이용하는 경우의 '게시글 + 댓글의 수'

      - @Query를 이용하는 경우의 '게시글 + 댓글의 수'

### 쿼리 메소드를 이용하는 경우
    
  - 일반적으로 게시물은 게시물 번호의 역순으로 페이징 처리가 되므로 쿼리 메소드를 이용하는 경우 다음과 같이 작성할 수 있다.

  ```Java
  public interface FreeBoardRepository extends CrudRepository<FreeBoard, Long> {

    public List<FreeBoard> findByBnoGreaterThan(Long bno, Pageable page);

  }
  ```
  
  - 현재 FreeBoard의 객체는 댓글에 대한 리스트를 참조하고 있으므로, 테스트 코드에서는 페이징 처리에 관련된 정보뿐 아니라 댓글의 수를 파악할 수 있다.


### 지연 로딩(lazy loading)
  
  - (예제) 게시물의 '제목'옆에 댓글의 수가 표시
 
     ```Java
    // 지연로딩(게시물의 제목 + 댓글의 수)
    @Test
    public void testList2(){
        PageRequest page = PageRequest.of(0, 10, Sort.Direction.DESC, "bno");
        boardRepository.findByBnoGreaterThan(0L, page).forEach(board -> {
            log.info(board.getBno() + ": "+board.getTitle() + " : " + board.getReplies().size());
        });
    }
     ```

  - 테스트를 실행하면 정상적으로 실행되지 않고, 에러가 발생한다.

     ```
     org.hibernate.LazyInitializtionException: failed to lazily initialize a collection of role: org.~.replies, could not initialize proxy - no Session
     ```

     - JPA 는 연관관계가 있는 엔티티를 조회할 때 기본적으로 '지연 로딩(lazy loading)'이라는 방식을 이용한다.

        - 지연 로딩은 '게으른'이라는 의미로, 정보가 필요하기 전까지는 최대한 테이블에 접근하지 않는 방식을 의미한다.

        - 지연 로딩을 하는 가장 큰 이유는 성능 때문이다. 하나의 엔티티가 여러 엔티티들과 종속적인 관계를 맺고 있다면, SQL에서 조인을 이용하는데, 조인이 복잡해질수록 성능이 저하되기 때문이다.

        - JPA에서는 연관관계의 Collection 타입을 처리할 때 '지연 로딩'을 기본적으로 사용한다. 

     - 지연 로딩의 반대 개념은 '즉시 로딩(eager loading)'이다.
        
        - 즉시 로딩은 일반적으로 조인을 이용해서 필요한 모든 정보를 처리하게 된다.

        - 즉시 로딩을 사용하려면 @OneToMany에 'fetch'라는 속성값으로 'FetchType.EAGER'를 지정하면 된다.

  ```Java
    @Getter
    @Setter
    @ToString(exclude = "replies")
    @Entity
    @Table(name = "tbl_freeboards")
    @EqualsAndHashCode(of = "bno")
    public class FreeBoard {

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

        // CascadeType : 연관관계를 함께 저장
        // fetch = FetchType.Eager : 즉시 로딩
        @OneToMany(mappedBy = "board", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
        private List<FreeBoardReply> replies;
    }
  ```

  - 코드 변경 후 재실행하게 되면 정상적으로 출력되게 된다.
     
     - 실행되는 SQL을 보면 우선은 페이지의 목록을 처리하는 SQL이 실행되고, 이후에 각 게시물에 대해서 'select ...'가 이루어지는 것을 볼 수 있다.

     - 실제로 결과가 나오기 위해서 1번의 목록을 추출하는 SQL과 10번의 조회용 SQL이 실행되기 때문에 권장할 수 있는 방법이라고 할 수 없다.

     - 지연 로딩과 즉시 로딩을 사용할 때는 반드시 해당 작업을 위해서 어떠한 SQL들이 실행되는지를 체크해야 한다.

  - 지연로딩을 이용하면서 댓글을 같이 가져오고 싶다면 @Transactional을 이용해서 처리해 주어야 한다.

    ```Java
    @Getter
    @Setter
    @ToString(exclude = "replies")
    @Entity
    @Table(name = "tbl_freeboards")
    @EqualsAndHashCode(of = "bno")
    public class FreeBoard {

        ... 

        // CascadeType : 연관관계를 함께 저장
        // fetch = FetchType.Eager : 즉시 로딩
        @OneToMany(mappedBy = "board", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
        private List<FreeBoardReply> replies;
    }

    @Transactional
    @Test
    public void testList2(){
        PageRequest page = PageRequest.of(0, 10, Sort.Direction.DESC, "bno");
        boardRepository.findByBnoGreaterThan(0L, page).forEach(board -> {
            log.info(board.getBno() + ": "+board.getTitle() + " : " + board.getReplies().size());
        });
    }
    ```

    - 실행 결과를 보면 정상적을 결과가 나오지만, 이전과 마찬가지로 각 게시물마다 댓글을 가져오는 SQL이 실행되고 있다.

### @Query와 Fetch Join을 이용한 처리

  - 지연 로딩의 문제를 해결하는 가장 좋은 방법은 @Query를 이용해서 조인 처리를 하는 것이다.

     - Hibernate 5.2는 연관관계가 없는 엔티티 간에도 조인이 가능

     - Hibernate 5.0은 연관관계가 존재하는 경우에 조인을 이용하여 처리 가능

     <br />

     ```Java
    public interface FreeBoardRepository extends CrudRepository<FreeBoard, Long> {
        ...

        @Query("SELECT b.bno, b.title, count(r) " + "FROM FreeBoard b LEFT OUTER JOIN b.replies r " + "WHERE b.bno > 0 GROUP BY b")
        public List<Object[]> getPage(PageRequest page);
        
    }
     ```
    
     - @Query를 이용해서 엔티티의 일부 속성이나, 다른 엔티티를 조회할 때의 리턴 타입은 컬렉션<배열>의 형태가 된다.

     - List<Object[]>에서 List는 결과 데이터의 '행(Row)'을 의미하고, Object[]는 '열(Column)'을 의미한다.

     <br />

     ```Java
            @Test
    public void testList3(){
        PageRequest page = PageRequest.of(0, 10, Sort.Direction.DESC, "bno");

        boardRepository.getPage(page).forEach(arr -> {
            log.info(Arrays.toString(arr));
        });
    }
     ```

     - 결과는 이전과 동일하지만, SQL은 1번만 실행된다.
        
        - SQL 내부적으로 외부 조인을 이용해서 한 번에 필요한 데이터들을 가져오는 것을 볼 수 있다.

### 4.4.5 게시물 조회와 인덱스

  - 게시물을 조회하는 경우에 가장 중요한 고민은 '지연 로딩을 이용할 것인가?', '즉시 로딩을 이용할 것인가?'이다.


     - 지연 로딩은 필요할 때까지 댓글 관련 데이터를 로딩하지 않기 때문에 성능면에서 장점을 가지게 되지만, 한 번에 게시물과 댓글의 내용을 보여 주는 상황이라면, SQL이 한 번에 처리하지 않기 때문에 여러 번 데이터베이스를 호출하는 문제가 있다.

     - 이에 대한 해결책은 지연 로딩을 그대로 이용하고, 댓글 쪽에서는 필요한 순간에 데이터가 좀 더 빨리 나올 수 있도록 신경 쓰는 방식이다.

### 인덱스 처리

  - @Query를 이용해서 원하는 데이터만 처리할 수 있다.

  - 댓글의 경우는 특정한 게시물 번호에 영향을 받기 때문에 게시물 번호에 대한 인덱스를 생성해 두면 데이터가 많을 때 성능의 향상을 기대할 수 있다.

    ```Java
    @Getter
    @Setter
    @ToString(exclude = "board")
    @Entity
    @Table(name = "tbl_free_replies", indexes = {@Index(unique = false, columnList = "board_bno")})
    @EqualsAndHashCode(of = "rno")
    public class FreeBoardReply {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long rno;
        private String reply;
        private String replyer;

        @CreationTimestamp
        private Timestamp replydate;
        @UpdateTimestamp
        private Timestamp updatedate;

        @ManyToOne
        private FreeBoard board;
    }
    ```
  
  <br />

  - @Table에는 인덱스를 설계할 때 @Index와 같이 사용해서 테이블 생성 시에 인덱스가 설계되도록 지정할 수 있다.

  - 이와 같이 설정된 상황에서 'select * from btl_tree_replies whyere board_bno = 200 order by rno desc'와 같은 SQL문을 실행하고, 실행 계획을을 살펴보면 생성된 인덱스를 이용하는 모습을 확인할 수 있다. 

<br />

  <img src="https://user-images.githubusercontent.com/63120360/183570062-1c566d47-88e1-4b13-9599-2537c0b895e3.jpg">
