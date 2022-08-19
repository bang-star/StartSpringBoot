# 7.3 ReplyController의 설계

 - API 설계

    <img src="https://user-images.githubusercontent.com/63120360/185058447-0aa469a9-3773-4b85-bf20-cf741f9cec84.png">

<br />

 - WebReplyController
 
    ```Java
    @RestController
    @RequestMapping("/replies/")
    public class WebReplyController {
        
        @Autowired     // setter를 만들어서 처리하는 것이 정석
        private WebReplyRepository replyRepository;
    }
    ```

## 7.3.1 특정 게시물의 댓글 등록 처리

 - 화면상에서의 댓글 등록 처리는 Ajax를 이용한다. 이를 위해서 전달되는 데이터를 JSON 형태로 처리한다. 이 작업을 위해 @PathVariable와 @RequestBody 어노테이션을 활용한다.

  <img src="https://user-images.githubusercontent.com/63120360/185059881-4f698d06-b6bd-4217-89e9-46ef0d50a0b6.png">


  <br />

  - @PathVariable은 URI의 일부를 파라미터로 받기 위해서 사용하는 어노테이션이고, @RequestBody는 JSON으로 전달되는 데이터를 객체로 자동으로 변환하도록 처리하는 역할을 한다.

  - WebReplyController에 댓글을 추가하는 기능
  
    ```Java
    @RestController
    @RequestMapping("/replies/")
    @Log
    public class WebReplyController {

        @Autowired     // setter를 만들어서 처리하는 것이 정석
        private WebReplyRepository replyRepository;
        
        @PostMapping("/{bno}")
        public ResponseEntity<Void> addReply(@PathVariable("bno") Long bno, @RequestBody WebReply reply){
            log.info("addReply ..................");
            log.info("BNO : "+bno);
            log.info("REPLY : "+reply);
            
            return new ResponseEntity<>(HttpStatus.CREATED);
        }
    }
    ```

  - WebReplyController의 메소들의 리턴 타입은 특이하게 RepsonseEntity 타입을 이용한다.

    - ResponseEntity는 코드를 이용해서 직접 Http Response의 상태 코드와 데이터를 직접 제어해서 처리할 수 있다는 장점이 있다.

  - addReply()에서 HTTP의 상태 코드 중에 201을 의미하는 'created'라는 메시지를 전송하도록 한다. ㅈ

<br />
<hr />
<br />

## 7.3.2 REST 방식 테스트

  - 'Yet Another Rest Client' 도구 활용

    <img src="https://user-images.githubusercontent.com/63120360/185063193-f2fd587e-24b9-4334-ad40-da38abd3a75b.png">


### 댓글 등록 처리 호출
  
  <img src="https://user-images.githubusercontent.com/63120360/185064073-ac0fa565-62e3-4a55-abd4-9dc07c987ae9.png">

  - 전송하는 URL은 'http://localhost:8080/replies/게시물번호'와 같이 지정하고, 전송 방식은 POST 방식으로 지정한다.
  
  - Payload에 입력하는 정보는 기본적으로 JSON 데이터이다. 
    
    - JSON 데이터는 JavaScript로 객체를 표현하는 문법으로 '{키:값}'의 형태로 객체를 문자열로 표현한다.

<br />
<hr />
<br />

## 7.3.3

  - 댓글의 데이터가 정상적으로 수집되는 것을 확인했다면 WebReplyRepository와 연동해서 실제로 댓글을 추가한다.

  - 댓글을 추가한 후에는 데이터베이스에서 현재 게시물의 댓글을 새롭게 조회해서 처리해야 한다. 이유는 댓글을 추가하는 동안 다른 사용자들이 댓글을 추가할 수 있기 때문에, 홤녀에서 댓글의 리스트를 새롭게 갱신할 필요가 있기 때문이다.

    ```Java
    public interface WebReplyRepository extends CrudRepository<WebReply, Long> {
    @Query("SELECT r FROM WebReply r WHERE r.board = ?1 "+ "AND r.rno > 0 ORDER BY r.rno ASC")
        public List<WebReply> getRepliesOfBoard(WebBoard board);
    }
    ```

    ```Java
    @RestController
    @RequestMapping("/replies/")
    @Log
    public class WebReplyController {

        @Autowired     // setter를 만들어서 처리하는 것이 정석
        private WebReplyRepository replyRepository;

        @Transactional
        @PostMapping("/{bno}")
        public ResponseEntity<List<WebReply>> addReply(@PathVariable("bno") Long bno, @RequestBody WebReply reply){
            log.info("addReply ..................");
            log.info("BNO : "+bno);
            log.info("REPLY : "+reply);

            WebBoard board = new WebBoard();
            board.setBno(bno);

            reply.setBoard(board);
            replyRepository.save(reply);

            return new ResponseEntity<>(getListByBoard(board), HttpStatus.CREATED);
        }

        private List<WebReply> getListByBoard(WebBoard board) throws RuntimeException{
            log.info("getListByBoard ... " + board);
            return replyRepository.getRepliesOfBoard(board);
        }
    }
    ```

  - addReply()는 WebRepository에 save() 작업과 findBoard()를 연속해서 호출하기 때문에 @Transational처리가 필요하다.

  - 게시물의 댓글의 목록이 필요할 수 있으므로, getListByBoard()라는 메소드로 분리하였다.

<br />
<hr />
<br />

## 7.3.4 댓글 삭제

  - 댓글 삭제 처리에는 기본적으로 댓글의 번호(rno)만 필요하지만, 댓글이 삭제된 후에 다시 해당 게시물의 모든 댓글을 갱신할 필요가 있기 때문에 게시물의 번호가 같이 필요하다.

     ```Java
    @Transactional
    @DeleteMapping("/{bno}/{rno}")
    public ResponseEntity<List<WebReply>> remove(
            @PathVariable("bno") Long bno, 
            @PathVariable("rno") Long rno){
        
        log.info("delete reply: " + rno);
        
        replyRepository.deleteById(rno);
        
        WebBoard board = new WebBoard();
        board.setBno(bno);
        
        return new ResponseEntity<>(getListByBoard(board), HttpStatus.OK);
    }
     ```

  - 삭제 처리에는 '/{bno}/{rno}'

    - URI에 댓글 번호가 포함되기는 하지만, 사용자는 브라우저에서 DELETE 방식으로 호출하기 어렵기 때문에 큰 문제는 없다. 댓글 삭제 후에는 다시 현재 게시물의 댓글 목록을 반환한다.

<br />
<hr />
<br />

## 7.3.5 댓글 수정

 - 댓글 수정 처리 과정은 댓글 등록 처리 과정과 유사하다.

   ```Java
    @Transactional
    @PutMapping("/{bno}")
    public ResponseEntity<List<WebReply>> modify(@PathVariable("bno") Long bno, @RequestBody WebReply reply) {
         
        log.info("modify reply : "+reply);
        
        replyRepository.findById(reply.getRno()).ifPresent(origin -> {
            
            origin.setReplyText(reply.getReplyText());
            replyRepository.save(origin);            
        });
        
        WebBoard board = new WebBoard();
        board.setBno(bno);
        
        return new ResponseEntity<>(getListByBoard(board), HttpStatus.CREATED);
    }
   ```

  - 댓글 수정은 실제로는 댓글의 내용을 수정하는 것이지만, 다른 칼럼의 값은 그대로 유지해야 하기 때문에 원래의 댓글에서 내용만을 변경해 처리한다.

## 7.3.6 댓글 목록

 - 댓글 목록은 GET 방식으로 처리하고, 게시물의 번호를 이용한다.

    ```Java
    @GetMapping("/{bno}")
    public ResponseEntity<List<WebReply>> getReplies(
            @PathVariable("bno") Long bno){

        log.info("get All Replies .................. ");

        WebBoard board = new WebBoard();
        board.setBno(bno);
        return new ResponseEntity<>(getListByBoard(board), HttpStatus.OK);
    }
    ```