# 9.5 Ajax의 시큐리티 처리

 - 게시물 조회 화면에서의 댓글은 Ajax로 처리되기 때문에 별도의 처리가 필요하다.

    - 가장 신경 써야 하는 부분은 Ajax로 호출하는 작업에 CSRF 값이 같이 전송되어야 한다는 것이다.

 - 댓글에 관련된 모든 작업은 기본적으로 로그인한 사용자들만이 가능하므로, WebReplyController를 수정한다.

     ```Java
     //WebReplyController.java

     @RestController
    @RequestMapping("/replies/")
    @Log
    public class WebReplyController {
        ... 중간 생략

        @Secured(value = {"ROLE_BASIC", "ROLE_MANAGER", "ROLE_ADMIN"})
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

        @Secured(value = {"ROLE_BASIC", "ROLE_MANAGER", "ROLE_ADMIN"})
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

        @Secured(value = {"ROLE_BASIC", "ROLE_MANAGER", "ROLE_ADMIN"})
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

        ... 중간 생략
    }
     ```

  - 댓글 관련 기능에서는 등록, 수정, 삭제 작업에 시큐리티 설정을 추가한다.

<br />
<hr />
<br />

## 9.5.1 댓글 추가

  - 댓글을 추가하는 작업은 현재 사용자가 로그인을 해야만 가능한다.

    - 화면상에 댓글을 추가하는 버튼을 누르면 로그인을 하도록 유도하기 위해 JavaScript를 이용해야 한다.

  - view.html에 CSRF 토큰이 처리되도록 의미 없는 태그를 하나 추가해야한다.

    ```HTML
    <!-- view.html -->

    <div layout:fragment="content">
        <form th:action="${'/login'}"></form>
    ```

  - 댓글의 작성자는 현재 로그인한 사용자로 처리해 준다.

    ```JavaScript
    // view.html
    var uid = [[${#authentication.principal} eq 'anonymousUser'? null : ${#authentication.principal.member.uid}]];

    $("#addReplyBtn").on('click', function(){
        
        if(uid == null){
            if(confirm("로그인할까요?")){
                self.location = [[@{/login}]];
            }
                return;
        }
            
        replyerObj.val(uid);
            
        $("#myModal").modal("show");
        $(".modal-title").text("Add Reply");

        $("#delModalBtn").hide();

        mode = "ADD";
    });
    ```

  - 로그인을 하지 않은 사용자는 로그인 페이지로 이동시키고, 로그인한 사용자는 사용자의 아이디가 댓글 입력창에 고정되도록 한다. (댓글의 작성자는 readonly로 지정한다.)

  - 댓글을 저장할 때에는 댓글의 제목과 작성자 정보 외에도 CSRF 파라미터를 같이 전달해야 한다.

  - JavaScript에서 CSRF 값을 이용하기 위해서 '${_crsf}'객체를 JavaScript의 객체로 변환할 필요가 있다. Thymeleaf의 인라인 기능을 이용하면 이를 쉽게 처리할 수 있다.

  - Thymeleaf의 인라인 JavaScript는 객체를 JavaScript의 객체 문법으로 변환해 준다.

    ```JavaScript
    // view.html 

    var csrf = JSON.parse('[[${_csrf}]]');
    ```

<br />

### NOTE
```
java.lang.illegalStateException: Cannot create a session after the response has been committed 메시지

시큐리티 설정이 필요한 페이지에서 사용할 때에는 서버에서 illegalStateException 메시지가 출력되는 경우가 있다. 이런 상황에서 브라우저에서 JavaScript 객체가 다음과 같이 생성되므로 주의해야 한다.

var csrf = JSON.parse('~');

이러한 메시지가 출력되는 이유는 CSRF 토큰이 만들어지기 전에 사용하면서 발생한다. 이를 해결하는 가장 간단한 방법은 페이지에 <form th:action="${'/login'}></form>과 같이 의미 없는 form 태그를 추가하는 것이다.

Thymeleaf의 th:action을 처리하기 위해서 반드시 CSRF 값을 생성해 내기 때문에 이와 같은 상황에서 유용하다.
```

  - JSON.parse()를 이용했기 때문에 문자열을 JavaScript의 객체로 만들어 주고, Ajax 호출 시 필요한 데이터를 추가할 수 있다.

  - 댓글 추가 버튼을 누르면 JavaScript 객체로 만들어진 csrf객체를 같이 전송한다.

    ```JavaScript
    // view.html

    $("#modalBtn").click(function (){
                
                var replyText = replyTextObj.val();
                var replyer = replyerObj.val();

                if(mode == 'ADD'){

                    var obj = {replyText:replyText, replyer:replyer, bno:bno, csrf:csrf};
    ```

  - Ajax를 전송하는 reply.js에서 전달된 csrf 객체를 처리한다.

    ```JavaScript
    var add = function (obj, callback){
        console.log("add ... ");

        $.ajax({
            type: 'post',
            url : '/replies/'+obj.bno,
            data : JSON.stringify(obj),
            dataType : 'json',
            beforeSend : function (xhr){
                xhr.setRequestHeader(obj.csrf.headerName, obj.csrf.token);
            },
            contentType : "application/json",
            success:callback
        });
    };
    ```

     - 가장 중요한 부분은 Ajax 전송 시에 'X-CSRF-TOKEN'헤더를 지정해 주는 것이다. csrf 객체에서 headerName과 token값을 이용해서 HTTP 헤더 정보를 구성한다. 이 코드를 이용해서 실제로 댓글을 추가하면 브라우저는 서버에 'X-CSRF-TOKEN'헤더를 추가한 상태에서 전송하게 된다.

     ```JavaScript
     var add = function (obj, callback){
        console.log("add ... ");

        $.ajax({
            type: 'post',
            url : '/replies/'+obj.bno,
            data : JSON.stringify(obj),
            dataType : 'json',
            beforeSend : function (xhr){
                xhr.setRequestHeader(obj.csrf.headerName, obj.csrf.token);
            },
            contentType : "application/json",
            success:callback
        });
    };
     ```

<br />
<hr />
<br />

## 9.5.2 댓글 수정/삭제

  - 댓글의 수정과 삭제는 현재 로그인한 사용자가 작성한 댓글만 가능하므로, 화면에서 버튼을 제어할 필요가 있다.

    ```javascript
    // view.html 일부

    $("#replyTable").on('click', "tr", function(e){
                
        var tds = $(this).find('td');

        console.log(tds);

        rno = tds[0].innerHTML;
        mode = 'MOD';

        replyTextObj.val(tds[1].innerHTML);
        replyerObj.val(tds[2].innerHTML);
        $("#delModalBtn").show();
        $("#myModal").modal("show");
        $(".modal-title").text("Modify/Delete Reply");
        
        if(uid != tds[2].innerHTML.trim()){
            $("#delModalBtn").hide();
            $("#modalBtn").hide();
        }
    
    });
    ```

  - 화면에서 댓글을 조회하는 것은 자유롭지만, 자신이 작성하지 않은 댓글을 수정하거나 삭제하는 작업은 제한되도록 버튼 자체가 보이지 않게 된다.

  - 댓글의 수정과 삭제 역시 CSRF 값을 같이 사용해야 하므로 reply.js를 호출하기 전에 csrf 객체를 전달하도록 수정한다.

    ```javascript
    // view.html 일부

    $("#delModalBtn").on('click', function(){
        var obj = {bno:bno, rno:rno, csrf:csrf};

        replyManager.remove(obj, function (list){
            alert("댓글이 삭제되었습니다.");
            afterAll(list);
        });
    });

    $("#modalBtn").click(function (){

        var replyText = replyTextObj.val();
        var replyer = replyerObj.val();

        if(mode == 'ADD'){

            var obj = {replyText:replyText, replyer:replyer, bno:bno, csrf:csrf};

            // console.log(obj);

            replyManager.add(obj, function (list){
                alert("새로운 댓글이 추가되었습니다.");
                afterAll(list);
            });
        }else if(mode == 'MOD'){
            var obj = { replyText:replyText, bno:bno, rno:rno, csrf:csrf};

            replyManager.update(obj, function (list){
                alert("댓글이 수정되었습니다.");
                afterAll(list);
            });
        }
    });
    ```

  - 댓글 수정/삭제 작업을 처리하는 reply.js에서도 csrf를 적용한다.

    ```javascript
    var update = function (obj, callback){
        console.log("update ... ");

        $.ajax({
            type: 'put',
            url : '/replies/'+obj.bno,
            dataType : 'json',
            data : JSON.stringify(obj),
            contentType : "application/json",
            beforeSend : function (xhr){
                xhr.setRequestHeader(obj.csrf.headerName, obj.csrf.token);
            },
            success:callback
        });
    }

    var remove = function (obj, callback){
        console.log("remove ... ");

        $.ajax({
            type: 'delete',
            url: '/replies/' + obj.bno + "/" + obj.rno,
            dataType: 'json',
            contentType: "application/json",
            beforeSend : function (xhr){
                xhr.setRequestHeader(obj.csrf.headerName, obj.csrf.token);
            },
            success: callback
        });
    };
    ```
