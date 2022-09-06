# 9.4 게시물의 수정/삭제

  - 게시물의 수정/삭제는 로그인한 사용자들만이 가능해야 하므로 SecurityConfig에 추가하거나, WebBoardController에 @Secure를 이용해서 처리한다.

    ```Java
    // WebBoardController.java

    @Secured(value = {"ROLE_BASIC", "ROLE_MANAGER", "ROLE_ADMIN"})
    @GetMapping("/modify")
    public void modify(Long bno, @ModelAttribute("pageVO") PageVO vo, Model model){
        log.info("MODIFY BNO : " + bno);

        repository.findById(bno).ifPresent(board -> model.addAttribute("vo", board));
    }

    @Secured(value = {"ROLE_BASIC", "ROLE_MANAGER", "ROLE_ADMIN"})
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

    @Secured(value = {"ROLE_BASIC", "ROLE_MANAGER", "ROLE_ADMIN"})
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

<br />

  - 컨트롤러에서 수정/삭제 작업에 시큐리티 설정이 추가되었기 때문에 로그인을 하지 않았다면 로그인 페이지로 이동하게 된다.

  - 만일 게시물의 작성자가 수정/삭제 작업을 하기 위해서 게시물의 조회 화면을 본다면 다음과 같이 정상적으로 출력된 페이지를 볼 수 있다.

  - 삭제 처리를 하기 위해 실제로 'Delete'버튼을 클릭하면 정상적으로 처리되지 못하고 에러 메시지만 보이게 된다.

    - 에러가 나는 이유는 시큐리티를 이용하는 경우 CSRF 파라미터가 반드시 필요하기 때문이다.

    <br />

    ```HTML
    <html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{/layout/layout1}">
    ```

  - form 태그 내에 CSRF 관련 태그를 추가한다.

    ```HTML
    <form id='f1'>
      <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    ```

  - 직접 추가하기 싫다면 th:action을 처리해 주면 자동으로 CSRF 파라미터가 설정된다.

    ```HTML
    <form id='f1' th:action="@{/}">
    ```

  - th:action을 이용하는 경우 화면상에서 form 태그 부분에 < input type="hidden" >으로 CSRF가 처리된다.