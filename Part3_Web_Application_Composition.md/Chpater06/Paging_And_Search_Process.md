# 6.2 페이징, 검색 처리

## 6.2.1. Repository 페이징 테스트

 - 페이징에 대한 테스트는 QuerydslPredicateExecutor의 findAll()을 이용하고, Pageable를 사용한다.

 - QuerydslPredicateExcecutor의 findAll()은 Predicate 타입의 파라미터와 Pageable을 파라미터로 전달받을 수 있다.

    - 파라미터로 전달하는 Predicate는 QWebBoard를 이용해서 작성해야 한다.

    - 간단히 검색 조건과 키워드를 전달하면 Predicate를 생성하는 코드를 작성해 주어야 한다.

    ```Java
    public interface WebBoardRepository extends CrudRepository<WebBoard, Long>, QuerydslPredicateExecutor<WebBoard> {

        public default Predicate makePredicate(String type, String keyword){
            BooleanBuilder builder = new BooleanBuilder();

            QWebBoard board = QWebBoard.webBoard;

            // type if ~ else

            // bno > 0
            builder.and(board.bno.gt(0));

            return builder;
        }
    }
    ```

  - makePredicate()는 검색에 필요한 타입(type) 정보와 키워드(keyword)를 이용해서 적당한 쿼리를 생성한다. 

    - 이 코드에는 아직 다양한 조건에 대한 처리가 없으므로 단지 'where bno >'이라느 조건만을 생성하도록 한다.

    ```Java
    @Test
    public void testList1(){
        PageRequest paging = PageRequest.of(0, 20, Sort.Direction.DESC, "bno");

        Page<WebBoard> result = repo.findAll(repo.makePredicate(null, null), paging);

        log.info("PAGE : " + result.getPageable());
        log.info("--------------------------------");

        result.getContent().forEach(board -> log.info(""+board));
    }
    ```

    - testList()은 아직 검색 조건을 지정하지 않은 상태의 단순 페이징 처리를 하도록 한다.

### 검색 조건 처리

 - 검색 조건이 없을 때 페이지 처리에 이상이 없는 것을 확인하였다면, 검색 조건에 맞게 남은 부분 구현

   ```Java
    public default Predicate makePredicate(String type, String keyword){
        BooleanBuilder builder = new BooleanBuilder();

        QWebBoard board = QWebBoard.webBoard;
       
        // bno > 0
        builder.and(board.bno.gt(0));
        
        // type if ~ else
        if(type == null){
            return builder;
        }
        
        switch (type){
            case "t":
                builder.and(board.title.like("%"+keyword+"%"));
                break;
            case "c":
                builder.and(board.content.like("%"+keyword+"%"));
                break;
            case "w":
                builder.and(board.writer.like("%"+keyword+"%"));
                break;
        }
        
        return builder;
    }
   ```

 - makePredicate()는 파라미터로 전달되는 문자열을 이용해 switch를 처리하는 단순한 구조이다.

 - 분기는 컨트롤러에서 다른 메소드를 호출하도록 처리할 수 있겠지만, 복잡한 연산을 처리해야 하는 상황에서 이와 같은 구조는 더 유용하다.

 - 검색 조건에 대한 테스트는 별도의 테스트 메소드가 필요하다.

    ```Java
    @Test
    public void testList2(){
        PageRequest pageable = PageRequest.of(0, 20, Sort.Direction.DESC, "bno");

        Page<WebBoard> result = repo.findAll(repo.makePredicate("t", "10"), pageable);

        log.info("PAGE : " + result.getPageable());
        log.info("--------------------------------");

        result.getContent().forEach(board -> log.info("" + board));
    }
    ```

    - testList2()의 검색 조건은 '제목'에 '10'이라는 문자열이 포함된 WebBoard 객체들에 대한 검색이다.

    <br />

    ```SQL
    Hibernate: select webboard0_.bno as bno1_0_, webboard0_.content as content2_0_, webboard0_.regdate as regdate3_0_, webboard0_.title as title4_0_, webboard0_.updatedate as updateda5_0_, webboard0_.writer as writer6_0_ from tbl_webboards webboard0_ where webboard0_.bno>? and (webboard0_.title like ? escape '!') order by webboard0_.bno desc limit ?
    ```

## 6.2.2 컨트롤러의 페이징 처리

 - Repository에서 페이징 처리와 검색에 대한 처리가 완료되었으므로, 컨트롤러에서 파라미터를 전달하고 연동해서 결과를 처리하도록 해야 한다.

  - 웹 화면에서 전달되는 데이터

     1. 페이지 관련

        - 페이지 번호(page - 0, 1, 2, 3 ... )

        - 페이지당 사이즈(size - PageRequest의 기본 size는 20 )
    
     2. 검색 관련

        - 검색 종류(type)

        - 검색 키워드(keyword)

### @PageableDefault를 이용한 페이징 처리

  - WebBoardController의 list()에 페이지 관련 처리를 위한 파라미터를 추가해야한다.

    ```Java
    @GetMapping("/list")
    public void list(
            @PageableDefault(
                    direction = Sort.Direction.DESC,
                    sort = "bno",
                    size = 10,
                    page = 0
                    
            ) Pageable page)
            {
        log.info("list() called ... ");
    }
    ```

   - 페이징 처리에는 Pageable 타입을 이용하느 것이 가장 간단하지만, 매번 페이지 처리를 할 때마다 size나 direction 등을 지정해야 하는 경우에 매번 여러 개의 파라미터를 전닳해서 불편할 수 있다.

   - Spring Data 모듈에서는 컨트롤러에서 파라미터 처리에 편리하도록 만들어진 @PageableDefault 어노테이션을 이용하면 간단하게 Pageable 타입의 객체를 생성할 수 있다.

## PageVO를 생성하는 방식

 - @PageableDefault에 주의사항

    1. 페이지 번호가 0부터 시작하기 때문에 일반 사용들에게 직관적이지 않다.

    2. 파라미터를 이용해서 size를 지정할 수 있기 떄문에 고의적으로 size 값을 크게 주는 것을 막을 수 없다.

    3. 기타 정렬 방향이나 속성이 모두 브라우저에서 전달되는 값을 통해서 조절할 수 있기 때문에 고의적인 공격에 취약하다는 단점들이 있다. 

 - @PageableDefualt를 이용하는 방식보다는 별도로 파라미터를 수집해서 처리하는 Value Object를 생성하는 방식이 문제를 조금은 줄여줄 수 있다.

 - PageVO는 브라우저에서 전달되는 값은 페이지 번호(page)와 게시물의 수(size)만을 받도록 설계하고, 일정 이상의 값이 들어올 수 없도록 제약을 둔다.

 - 정렬 방향이나, 정렬 기준이 되는 속성은 컨트롤러에서 지정한다.

    ```Java
    public class PageVO {
        private static final int DEFAULT_SIZE = 10;
        private static final int DEFAULT_MAX_SIZE = 50;

        private int page;
        private int size;

        public PageVO(){
            this.page = 1;
            this.size = DEFAULT_SIZE;
        }

        public int getPage(){
            return page;
        }

        public void setPage(int page){
            this.page = page < 0 ? 1: page;
        }
        
        public int getSize(){
            return size;
        }
        
        public void setSize(int size){
            this.size = size < DEFAULT_SIZE || size > DEFAULT_MAX_SIZE ? DEFAULT_SIZE : size;
        }
        
        public Pageable makePageable(int direction, String... props){
            Sort.Direction dir = direction == 0 ? Sort.Direction.DESC : Sort.Direction.ASC;
            
            return PageRequest.of(this.page - 1, this.size, dir, props)''
        }
    }
    ```

   - makePageable() 메소드
    
     - 전달되는 파라미터를 이용해서 최종적으로 PageReuqest로 Pageable 객체를 만들어낸다.

     - 브라우저에서 전달되는 page 값을 1 줄여서 Pageable 타입을 생성한다.

  - WebController의 list()

     ```Java
     @GetMapping("/list")
     public void list(PageVO vo) {
        Pageable page = vo.makePageable(0, "bno");
        log.info("list() called ... " + page);
     }
     ```

     - 브라우저에서 전달되는 파라미터들은 자동으로 PageVO로 처리된다.

     - 페이지 번호(page)와 페이지당 개수(size)가 처리되고, 정렬 방향과 정렬 대상의 칼럼은 컨트롤러에서 처리하게 한다.

     - Pageable을 얻기 위해 vo.makePageable() 코드를 추가해야 하지만, 어노테이션을 사용하는 것에 비해 간결할 수 있다.


### Repository와의 연동 처리

  - WebBoardController에 @Autowired를 이용해서 WebBoardRepository를 주입하고, list()에서 호출하도록 변경
  
    ```Java
    @Controller
    @RequestMapping("/boards/")
    @Log
    public class WebBoardController {
        
        @Autowired
        private WebBoardRepository repository;

        @GetMapping("/list")
        public void list(PageVO vo, Model model) {
            Pageable page = vo.makePageable(0, "bno");
            
            Page<WebBoard> result = repository.findAll(repository.makePredicate(null, null), page);
            
            
            log.info("" + page);
            log.info("" + result);
            
            model.addAttribute("result", result);
        }
    }
    ```

<br />

  - 변경된 list()는 파라미터로 Model을 전달받고 WebBoardRepository를 이용해서 페이지 처리를 진행한 결과를 'result'라는 이름의 변수로 처리한다.


## NOTE
```
 화면상에서 ${result}로 출력되는 부분은 페이지 번호에 주의할 필요가 있다. 만일 '/boards/list'를 호출하면 내부적으로 Pageable은 0번 페이지를 의미하지만, 결과 화면에서는 1로 출력된다.

 실제 객체인 Pagelmpl의 코드의 toString() 부분 때문이다.
 ("Page %s of %d containing %s instances", getNumber() + 1, getTotalPages(), contentType)

 Pageable 인터페이스의 구현체인 PageableImpl 클래스의 toString() 이용 시, 실제 페이지 번호에 1을 더해서 출력하기 때문에 0이 아닌 1부터 출력되게 된다.
```

### 화면의 출력과 페이징 처리

 - 화면에 내용을 출력하는 부분은 부트스트랩의 화면 컴포넌트 중에서 List를 이용하고, 페이징 처리는 Pagination을 이용한다.

<br />

   ```HTML
   <html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">

    <div layout:fragment="content">

        <div class="panel-heading">List Page</div>
        <div class="panel-body">
            <p>[[${result}]]</p>
            <div>

                <ul class="list-group">
                    <li class="list-group-item " th:each="board:${result.content}">[[${board}]]</li>
                </ul>
            </div>
        </div>
    </div>

    <th:block layout:fragment="script">

        <script th:inline="javascript">
        </script>
    </th:block>
   ```

### 페이지 번호 출력

  - 페이징 처리에는 Page< WebBoard >의 getPageable()을 이용해서 Pageable 타입의 객체를 활용한다.

   - Pageable 객체의 구조
    
   <br />

   <img src="https://user-images.githubusercontent.com/63120360/184498830-c10eacf8-7a8b-4bc6-af70-2d0ff7ad8f49.png">

   <br />

  - 주어지는 Pageable 객체를 이용해서 화면에 간단하게 이전 페이지와 다음 페이지만을 보여준 것이라면 문제가 없지만, 일반적인 웹페이지처럼 페이지의 번호를 보여주려면 시작 페이지까지 몇 번 prevOrFirst()를 해야하고, next()를 해야하는지를 계산해야 한다.

  - 페이지 번호를 출력하려면 별도의 클래스를 이용해, 페이지 번호 출력에 필요한 정보들을 처리하도록 해야한다.

  ```JAVA
    @Getter
    @ToString(exclude = "pageList")
    @Log
    public class PageMaker<T> {

        private Page<T> result;

        private Pageable prevPage;
        private Pageable nextPage;

        private int currentPageNum;
        private int totalPageNum;

        private Pageable currentPage;

        private List<Pageable> pageList;

        public PageMaker(Page<T> result){
            this.result = result;
            this.currentPage = result.getPageable();
            this.currentPageNum = currentPage.getPageNumber() + 1;
            this.totalPageNum = result.getTotalPages();
            this.pageList = new ArrayList<>();

            calcPages();
        }

        private void calcPages(){
            int tempEndNum = (int)(Math.ceil(this.currentPageNum/10.0)*10);
            int startNum = tempEndNum - 9;

            Pageable stratPage = this.currentPage;

            // move to start Pageable
            for(int i = startNum; i<this.currentPageNum; i++){
                stratPage = stratPage.previousOrFirst();
            }

            this.prevPage = stratPage.getPageNumber() <= 0 ? null : stratPage.previousOrFirst();

            // log.info("tempEndNum : " + tempEndNum);
            // log.info("total : " + totalPageNum);

            if(this.totalPageNum < tempEndNum){
                tempEndNum = this.totalPageNum;
                this.nextPage = null;
            }

            for(int i = startNum; i<=tempEndNum; i++){
                pageList.add(stratPage);
                stratPage = stratPage.next();
            }

            this.nextPage = stratPage.getPageNumber() + 1 < totalPageNum ? stratPage : null;
        }
    }
  ``` 
  
  - PageMaker는 화면에 출력할 결과 Page< T >를 생성자로 전달받고, 내부적으로 페이지 계산을 처리하게 된다. 

  - PageMaker가 처리하는 데이터

      - prevPage

        페이지 목록의 맨 앞인 '이전'으로 이동하는 데 필요한 정보를 가진 Pageable

      - nextPage

         페이지 목록 맨 뒤인 '다음'으로 이동하는데 필요한 정보를 가진 Pageable

      - currentPage

         현재 페이지의 정보를 가진 Pageable

      - pageList

         페이지 번호의 시작부터 끝까지의 Pageable들을 저장한 리스트

      - currentPageNum

         화면에 보이는 1부터 시작하는 페이지 번호

    ```Java
    @GetMapping("/list")
    public void list(PageVO vo, Model model) {
        Pageable page = vo.makePageable(0, "bno");

        Page<WebBoard> result = repository.findAll(repository.makePredicate(null, null), page);
        
        log.info("" + page);
        log.info("" + result);

        log.info("TOTAL PAGE NUMBER" + result.getTotalPages());

        model.addAttribute("result", new PageMaker(result));
    }
    ```
    
    ```HTML
    <html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">

    <div layout:fragment="content">

        <div class="panel-heading">List Page</div>
        <div class="panel-body">
            <p>[[${result}]]</p>
            <div th:with="result=${result.result}">

                <ul class="list-group">
                    <li class="list-group-item " th:each="board:${result.content}">[[${board}]]</li>
                </ul>
            </div>
        </div>
    </div>

    <th:block layout:fragment="script">

        <script th:inline="javascript">
        </script>
    </th:block>
    ```
  
  - 목록을 출력하는 div에 th:with를 이용해서 result 라는 변수가 PageMaker 객체안의 result 임을 명시한다. 이후의 처리는 그대로 이용할 수있다.

  - 브라우저에서 페이지의 시작 번호에서 끝 번호의 수만큼 'Page'라는 글자가 출력되는 것을 볼 수 있다.

  - 만일 페이지의 번호가 모자라는 경우에는 끝 페이지의 번호까지만 출력한다.

        - 예를들어, 300개의 데이터를 20개씩 출력하는 경우 마지막 페이지는 15페이지가 되므로 11페이지를 호출하는 경우 11 페이지부터 15 페이지까지 5개만 출력되는 것을 볼 수 있다.

     <br />

     <img src="https://user-images.githubusercontent.com/63120360/184526824-1e57ff39-87c7-4da9-ab09-c03e490b71bd.PNG">

     <br />

    ```HTML
    <!--   paging    -->
    <nav>
        <div>
            <ul class="pagination">
                <li th:each="p:${result.pageList}">
                    <a href="#">[[${p.pageNumber}+1]]</a>
                </li>
            </ul>
        </div>
    </nav>
    ```

  - Pageable 객체의 pageNumber는 1이 작게 되어 있으므로 화면에는 1을 더해 주어야 한다.

  - '이전'과 '다음' 페이지 처리는 th:if를 이용해서 검사하고, result.prevPage와 result.nextPage를 이용해서 처리한다.

  <br />

  <img src="https://user-images.githubusercontent.com/63120360/184527334-b48cb469-c4cd-4958-a52a-c4ce9e6d8890.png">

  - 남은 작업은 'Page'라는 글자 대신에 페이지 번호를 출력해 주는 것이다.

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">

    <div layout:fragment="content">

        <div class="panel-heading">List Page</div>
        <div class="panel-body">
            <p>[[${result}]]</p>
            <div th:with="result=${result.result}">

                <ul class="list-group">
                    <li class="list-group-item " th:each="board:${result.content}">[[${board}]]</li>
                </ul>
            </div>

            <!--   paging    -->
            <nav>
                <div>
                    <ul class="pagination">
                        <li class="page-item" th:if="${result.prevPage}">
                            <a href="#">PREV [[${result.prevPage.pageNumber} + 1]]</a>
                        </li>

                        <li th:each="p:${result.pageList}">
                            <a href="#">[[${p.pageNumber}+1]]</a>
                        </li>

                        <li class="page-item" th:if="${result.nextPage}">
                            <a href="#">NEXT [[${result.nextPage.pageNumber} + 1]]</a>
                        </li>
                    </ul>
                </div>
            </nav>

        </div>
    </div>

    <th:block layout:fragment="script">

        <script th:inline="javascript">
        </script>
    </th:block>
    ```

### 현재 페이지 번호 구분하기

  - 현재 페이지 번호를 구분하기 위해 Thymeleaf의 th:classapeend를 적용해서 특별한 경우에만 특정 CSS의 클래스를 추가하도록 할 수 있다.

  ```HTML
  <li class="page-item" th:classappend="${p.pageNumber == result.currentPageNum - 1}?active: '' " th:each="p:${result.pageList}">
        <a href="#">[[${p.pageNumber}+1]]</a>
  </li>
  ```

  - th:classappend와 삼항 연산자를 이용해서 간단하게 'page-item-acive'라는 클래스를 지정할 수 있다.

### 페이지 이동 처리

  - 브라우저상에서는 제대로 페이지의 정보가 보이지만, a 태그의 링크 내용은 그렇지 못한다.

  - 페이지 이동이 제대로 동작하기 위해 우선 태그의 링크 내용에서 페이지 정보가 제대로 보이도록 a 태그의 href 속성값을 페이지 번호로 지정한다.

    ```HTML
    
    ```