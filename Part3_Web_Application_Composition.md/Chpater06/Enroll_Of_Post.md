# 6.3 새로운 게시물 등록

 - 새로운 게시물을 등록하는 작업은 리스트를 처리하는 작업에 비해 수월하다. 다만 게시물의 등록 작업은 기존에 작성된 엔티티 클래스를 그대로 이용할 것인지, 별도의 Value Object용 클래스를 작성해서 처리할 것인가에 대한 고민이 필요하다.


## 6.3.1 게시물의 입력과 처리

 - 게시물을 작성하는 작업은 'GET'방식을 이용해서 입력하는 화면을 보고, 'POST'방식을 이용해서 새로운 게시물을 등록하도록 처리한다.

  ```Java
    @GetMapping("/register")
    public void registerGET(@ModelAttribute("vo") WebBoard vo){
        log.info("register get");
    }

    @PostMapping("/register")
    public String registerPOST(@ModelAttribute("vo") WebBoard vo, RedirectAttributes rttr){
        log.info("register post");
        log.info(""+vo);

        repository.save(vo);
        rttr.addFlashAttribute("msg", "success");

        return "redirect:/boards/list";
    }
  ```

   - GET 방식으로 들어오면 단순히 게시물을 작성할 수 있는 화면만을 보여주면 되므로, 굳이 WebBOard를 파라미터로 사용하지 않아도 된다.

   - POST 방식의 경우에는 새로운 게시물을 등록하고 리다이렉트를 시켜준다. 리다이렉트를 하지 않으면 사용자가 여러 번 게시물을 등록할 수 있기 때문에 이를 방지하기 위함이다.(Post - Redirect - GET 방식)

   - RedirectAttributes는 URL로는 보이지 않은 문자열을 생성해 주기 때문에 브라우저의 주소창에는 보이지 않는다.

   ```HTML
   <html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">

    <div layout:fragment="content">

        <div class="panel-heading">Register Page</div>
        <div class="panel-body">

            <form th:action="@{register}" method="post">
                <div class="form-group">
                    <label>Title</label>
                    <input class="form-control" name="title" th:value="${vo.title}" />
                    <p class="help-block">Title text here</p>
                </div>

                <div class="form-group">
                    <label>Content</label>
                    <textarea class="form-control" rows="3" name='content' th:text="${vo.content}"></textarea>
                </div>

                <div class="form-group">
                    <label>Writer</label>
                    <input class="form-control" name="writer" th:value="${vo.writer}" />
                </div>

                <button type="submit" class="btn btn-default">Submit Button</button>
                <button type="submit" class="btn btn-primary">Reset Button</button>
            </form>
        </div>
    </div>
    <!-- end fragment -->
    <th:block layout:fragment="script">
        <script th:inline="javascript">

        </script>
    </th:block>
   ```

   - 게시물을 등록하게 되면 리스트 페이지로 이동하기 때문에 새로운 게시물이 가장 상단에 등록되고 있다.

   - 리스트 페이지의 특이한 점은 RedirectAttributes의 addFlashAttribute()를 이용한 경우, URL 상에는 전달되는 'msg'라는 데이터가 보이지 않는다는 점이다.

   - URL 상에는 없지만 실질적으로 데이터가 전달되기 때문에 이를 이용해서 사용자에게 게시물이 정상적으로 등록되었다는 것을 알려주어야 한다.

   ```javascript
    $(window).load(function (){
    let msg = [[${msg}]];

    if(msg == 'success'){
        alert("정상적으로 처리되었습니다.");
        let stateObj = {msg : ""};
     }
    })
   ```

   - $(window).load()를 이용한 이유는 화면의 모든 구성이 온전해진 후에 경고창을 띄우기 때문이다.

### 게시물 입력 링크 처리

  - list 화면에서 사용자가 새로운 게시물을 작성할 수 있는 링크


     ```HTML
     <!-- list.html 일부 -->
     
    <div class="panel-heading">List Page</div>
        <div class="panel-body pull-right">
            <h3><a class="label label-default" th:href="@{register}">Register</a></h3>
        </div>
     ```