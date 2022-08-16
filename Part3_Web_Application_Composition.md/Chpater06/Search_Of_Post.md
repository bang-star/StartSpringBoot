# 6.4 게시물의 조회

 - 게시물의 리스트 화면에서 게시물의 상세 내용을 볼 수 있도록 처리되어야 한다.

 - 고려 사항

     - 검색 조건이 없는 경우의 조회

        페이지 번호를 유지한 상태에서 조회로 이동

     - 검색 조건이 있는 경우의 조회

        페이지 번호 + 기타 검색 조건을 모두 유지한 상태에서 이동

 - 게시물을 고려할 때는 사용자가 원래의 리스트 화면으로 이동할 수 있는 링크를 제공해야 한다. 이를 처리하는 가장 간단한 방법은 기존의 페이지 이동 시에 사용했던 from 태그를 이용하는 것이다.

    ```HTML
    <tr class="odd gradeX" th:each="board:${result.content}">
        <td>[[${board.bno}]]</td>
        <td><a th:href='${board.bno}' class='boardLink'>[[${board.title}]]</a> </td>
        <td>[[${board.writer}]]</td>
        <td class="center">[[${#dates.format(board.regdate, 'yyyy-MM-dd')}]]</td>
    </tr>
    ```

    - a 태그에 별도의 boardLink라는 클래스를 지정한 이유는 jQuery를 이용하는 처리를 간단히 하기 위한 방법이다.

    - a 태그의 href 속성에 단순히 번호만을 주었기 때문에 실제 이동은 JavaScript를 이용해서 처리한다.

    ```Javascript
    $(".boardLink").click(function (e){
        e.preventDefault();

        let boardNo = $(this).attr("href");

        formObj.attr("action", [[@{'/boards/view'}]]);
        formObj.append("<input type='hidden' name='bno' value='"+boardNo+"'>");
        formObj.submit();
    })
    ```
  
    - jQuery에서 a 태그의 기본 동작은 막고(preventDefault()) from 태그의 action을 게시물 조회가 가능한 링크로 변경한다.

    - 게시물 조회에는 게시물의 번호가 필수적이기 때문에 별도의 hidden 태그를 생성하여 form 태그에 추가한 후에 전송하게 된다.

    - 브라우저상에서 게시물의 번호를 클릭하면 모든 정보가 'boards/view'로 전송되는 것을 확인할 수있다. 검색한 후에도 동일하게 유지되는 것을 볼 수 있다.

    - 검색된 결과에서 특정 게시물의 제목을 클릭하면 모든 정보가 전송되는 것을 볼 수 있다.

## 6.4.1 컨트롤러의 처리

 - WebBoardController에서 게시물의 조회는 '/boards/view'가 되고, 경로로 전달되는 데이터는 '게시물의 번호 + 검색 조건 + 페이지 조건'이 된다.

 - 세 가지 중에 '검색 조건'과 '페이징 조건'은 PaveVO로 처리가 가능하기 때문에 게시물의 번호만 추가로 처리하도록 파라미터를 설정하면 된다.

   ```Java
    @GetMapping("/view")
    public void view(Long bno, @ModelAttribute("pageVO") PageVO vo, Model model){
        log.info("BNO : " + bno);

        repository.findById(bno).ifPresent(board -> model.addAttribute("vo", board));
    }
   ```

   ```HTML
   <!-- view.html -->
   <html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">

    <div layout:fragment="content">
        <div class="panel-heading">View Page</div>
        <div class="panel-body">

            <div class="form-group">
                <label>BNO</label>
                <input class="form-control" name="bno" th:value="${vo.bno}" readonly="readonly" />
            </div>

            <div class="form-group">
                <label>Title</label>
                <input class="form-control" name="title" th:value="${vo.title}" readonly="readonly" />
                <p class="help-block">Title text here.</p>
            </div>

            <div class="form-group">
                <label>Content</label>
                <textarea class="form-control" rows="3" name="content" th:text="${vo.content}" readonly="readonly"></textarea>
            </div>

            <div class="form-group">
                <label>Writer</label>
                <input class="form-control" name="writer" th:value="${vo.writer}" readonly="readonly">
            </div>

            <div class="form-group">
                <label>RegDate</label>
                <input class="form-control" name="regDate" th:value="${#dates.format(vo.regdate, 'yyyy-MM-dd')}" readonly="readonly">
            </div>
        </div>
    </div>

    <!--    end fragment    -->
    <th:block layout:fragment="script">
        <script th:inline="javascript">

        </script>
    </th:block>
   ```

 - 게시물의 목록에서 특정 게시물을 클릭하면 브라우저에서 편집이 불가능한 형태로 보인다.


## 6.4.2 조회 페이지에서의 링크 처리

  - 웹에서 주로 조회 페이지와 수정, 삭제 페이지는 분리하는 것이 일반적이다.

  - 조회 페이지는 내용을 보는 용도로만 활용하고, 수정이나 삭제는 별도의 페이지에서 이루어진다.

  - 요구사항

    - 수정/삭제 페이지로 이동할 수 있는 링크

    - 다시 게시물 목록(리스트)으로 이동할 수 있는 링크

  - 조회 페이지는 반드시 현재의 '검색 조건 + 페이징 조건'을 이용해서 다시 목록 페이지로 이동할 수 있도록 링크를 작성해야 한다.

  ```HTML
    <div class="pull-right">
        <a th:href="@{ modify(page=${pageVO.page}, size=${pageVO.size}, type=${pageVO.type}, keyword=${pageVO.keyword}, bno=${vo.bno})}" class="btn btn-default">Modify/Delete</a>
        <a th:href="@{ list(page=${pageVO.page}, size=${pageVO.size}, keyword=${pageVO.keyword}, bno=${vo.bno} )}" class="btn btn-primary">Go List</a>
    </div>
  ```

  - Thymeleaf의 링크 처리에는 '(키=값)'의 형태로 파라미터를 연결해서 링크를 생성할 수 있다.

    - 만일 검색을 통해서 조회 페이지에 들어오는 경우에는 자동으로 검색 키워드 등이 URL에 맞게 인코딩되어 생성된다.