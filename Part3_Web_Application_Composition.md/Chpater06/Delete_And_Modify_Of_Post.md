# 6.5 게시물의 삭제와 수정

 - 작업 순서

    1. 게시물 수정/삭제 페이징 처리

    2. 게시물 삭제 및 이동

    3. 게시물 수정 및 이동

## 6.5.1 게시물 수정/삭제를 위한 페이지

  - 게시물의 조회 페이지에서 수정/삭제는 별도로 페이지로 이동해서 처리되므로, 해당 URL에 맞는 WebBoardController의 메소드를 작성해주어야 한다.

  ```Java
  @GetMapping("/modify")
    public void modify(Long bno, @ModelAttribute("pageVO") PageVO vo, Model model){
        log.info("MODIFY BNO : " + bno);
        
        repository.findById(bno).ifPresent(board -> model.addAttribute("vo", board));
    }
  ```

  ```HTML
  <html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">

  <div layout:fragment="content">

      <div class="panel-heading">Modify Page</div>
      <div class="panel-body">

          <form id='f1'>
              <div class="form-group">
                  <label>BNO</label>
                  <input class="form-control" name="bno" th:value="${vo.bno}" readonly="readonly" />
              </div>

              <div class="form-group">
                  <label>Title</label>
                  <input class="form-control" name="title" th:value="${vo.title}" />
                  <p class="help-block">Title text here.</p>
              </div>

              <div class="form-group">
                  <label>Content</label>
                  <textarea class="form-control" rows="3" name="content" th:text="${vo.content}"></textarea>
              </div>

              <div class="form-group">
                  <label>Writer</label>
                  <input class="form-control" name="writer" th:value="${vo.writer}" readonly="readonly" />
              </div>

              <input type="hidden" name="page" th:value="${pageVO.page}">
              <input type="hidden" name="size" th:value="${pageVO.size}">
              <input type="hidden" name="type" th:value="${pageVO.type}">
              <input type="hidden" name="keyword" th:value="${pageVO.keyword}">
          </form>

          <div class="form-group">
              <label>RegDate</label>
              <input class="form-control" name="regDate" th:value="${#dates.format(vo.regdate, 'yyyy-MM-dd')}" readonly="readonly">
          </div>
      </div>

  </div>
  <!--   end fragment     -->

  <th:block layout:fragment="script">

      <script th:inline="javascript">
      
      </script>
  </th:block>
  ```

   - form 태그의 내부에는 페이징 조건과 검색 조건을 같이 전달할 수 있도록 hidden 태그들로 작성하는데, 이는 수정 등의 작업 후에 다시 원래 사용자가 보던 리스트 페이지로 이동하기 위해서이다.

   - modify.html의 버튼은 '수정', '삭제', '취소 후 목록'으로 구분된다.

   - '수정', '삭제' 버튼은 form 태그를 이용해서 처리해야 하므로 별도의 JavaScript를 이용해서 처리하고, '취소 후 목록'버튼은 링크를 생성해서 처리할 수 있다.


   ```html
    <div class="pull-right">
        <a th:href="#" class="btn btn-warning modbtn">Modify</a>
        <a th:href="#" class="btn btn-danger delbtn">Delete</a>

        <a th:href="@{list(page=${pageVO.page}, size=${pageVO.size}, type=${pageVO.type}, keyword=${pageVO.keyword}, bno=${vo.bno})}" class="btn btn-primary">Cancel & Go List</a>
    </div>
   ```

## 6.5.2 삭제 처리

 - 게시물을 수정하는 화면은 GET 방식으로 처리되지만, 실제 수정과 삭제 작업은 POST 방식으로 처리된다는 점이 다르기 때문에 별도로 처리해야 한다.

    ```Java
    @PostMapping("/delete")
    public String delete(Long bno, PageVO vo, RedirectAttributes rttr){
        log.info("DELETE BNO: "+bno);

        repository.deleteById(bno);
        rttr.addFlashAttribute("msg", "success");

        // 페이징과 검색했던 결과로 이동하는 경우
        rttr.addAttribute("page", vo.getPage());
        rttr.addAttribute("size", vo.getSize());
        rttr.addAttribute("type", vo.getType());
        rttr.addAttribute("keyword", vo.getKeyword());

        return "redirect:/boards/list";
    }
    ```

  - 게시물이 삭제된 후에는 다시 리스트 화면으로 가도록 하고, 결과만을 보여준다.

  - 사용자의 기존 페이징 정보와 검색 정보를 활용할 수 있도록 PageVO의 정보를 파라미터로 지정할 수 있다.

  - 기존 검색과 페이징 처리를 활용하는 경우 addAttribute()를 이용하게 된다. addAttribute()는 addFlashAttribute()와 달리 URL에 추가되어서 전송된다.

  ```javascript
  <script th:inline="javascript">
    $(document).ready(function (){
        var formobj = $("#f1");

        $(".delbtn").click(function (){

            formobj.attr("action", "delete");
            formobj.attr("method", "post");

            formobj.submit();
        });
    });
    </script>
  ```

### 6.5.3 삭제 처리

 - 수정과 삭제 차이점

    - 삭제의 경우 단순히 게시물의 번호만을 이용

    - 수정의 경우 기존의 게시물에 일부분만을 변경해서 수정하고, 저장한다.

    ```Java
    @PostMapping("/modify")
    public String modifyPost(WebBoard board, PageVO vo, RedirectAttributes rttr){
        log.info("Modify WebBoard : "  +board);

        repository.findById(board.getBno()).ifPresent( origin -> {

            origin.setTitle(board.getTitle());
            origin.setContent(board.getContent());
            repository.save(origin);
            rttr.addFlashAttribute("msg", "success");
            rttr.addAttribute("bno", origin.getBno());
        });

        // 페이징과 검색했던 결과로 이동하는 경우
        rttr.addAttribute("page", vo.getPage());
        rttr.addAttribute("size", vo.getSize());
        rttr.addAttribute("type", vo.getType());
        rttr.addAttribute("keyword", vo.getKeyword());

        return "redirect:/boards/view";
    }
    ```

  - 게시물의 수정 화면에서는 게시물의 제목과 내용만을 수정하게 되므로, 기존 게시물의 데이터를 가져온 이후에 변경되는 제목과 내용 속성을 갱신하고 save()를 통해 데이터베이스에 저장한다.

  - 조회 페이지로 이동한 후에 간단히 결과를 알려주기 위해서 addFlashAttribute()를 이용해서 'msg'를 추가한다.

  