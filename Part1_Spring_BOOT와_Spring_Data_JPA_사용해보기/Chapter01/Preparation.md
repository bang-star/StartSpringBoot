# 01. 프로젝트를 위한 준비

  - 스프링 부트는 기존의 스프링 프레임워크를 이용해왔던 사용자들에게 좀 더 빠른 개발이 가능하고, 손쉽게 프로젝트를 구성할 수 있는 다양한 방식을 제공합니다.

  - 스프링 부트 자체보다는 Spring Data JPA나 Thymeleaf이 주 내용으로 이루어져 있다.
  
  - 개발 버전
     
     - 데이터베이스 : MySQL 5.7
     - JDK 버전 : 1.8 이상
     - Spring Boot Version : 1.5.4/2.0.0(M2)

  <br />

## 1.1 통합 개발 도구 설정

  - 스프링 부트 프로젝트를 생성하는 방식
    
    1. 명령어 기반 프로젝트 생성 방식(CLI)

       주로 리눅스나 Mac 환경에서 유용

    2. 통합 개발 도구를 이용한 프로젝트 생성 방식

       Windows 계열의 환경에서 편리

<br />

### 1.1.1 Eclipse와 스프링 플러그인

  - Spring Boot 1.5.4는 Java7이 필요하지만, Spring Boot 2.0은 Java 8을 기본으로 삼고 있다.

  - Spring 플러그인 설치 : Lombok, Spring Web

  - Type : Maven / Packaging : War

  - XML 파일 대신 application.properties에서 필요한 설정을 지정할 수 있다.

  - ServletInitializer.java는 서블릿을 실행하는 역할을 수행합니다.

<br />
<hr />
<br />

## 1.2 스프링 프레임워크와 스프링 부트

  - 스프링 프레임워크의 시작은 기존의 EJB라는 무겁고 복잡한 개발을 벗어난 경량(light-weight)화된 개발을 추구하는 것으로 시작되었다.

  - 로드 존슨(Rod Johnson)이 자신의 책에서 작성한 소스코드에서부터 시작되어 스프링 프레임워크는 빠른 속도로 발전해왔다.

  - 기본적으로 Spring은 Servlet/JSP라는 기술을 지켜가면서 개발에 생산성을 높일 수 있는 다양한 방법을 제시하였습니다.

  - 스프링 부트는 기존의 스프링 개발 방식에서 불편했던 설정이나 버전 충돌 등의 불편했던 점들을 없애는 대신에 빠르고 쉬운 서버 환경과 테스트 환경 등을 한번에 제공해서 훨씬 간편한 개발 환경을 마련하게 되었습니다. 

  - 스프링 부트를 사용하면서 얻을 장점

    - 자동화된 라이브러리 관리

       기존에도 Maven이나 Gradle을 이용해 라이브러리를 추가하는 작업 등을 해왔다면 스프링 부트에서는 이 작업을 더욱 간단히 처리할 수 있습니다.

    - Spring Boot Auto Configure(자동 설정)

       스프링 부트에서 현재 프로젝트에 추가된 라이브러리를 이용해 자동으로 실행에 필요한 환경을 구성합니다. 예를 들어, 화면을 구성할 때 특정 라이브러리를 사용하기로 결정했다면 이에 관련된 설정을 자동으로 구성해 주기 때문에 개발자는 별다른 설정 없이도 개발할 수 있도록 구성됩니다.

    - 적당한 라이브러리 자동 결정과 XML 없는 환경 구축

      스프링 부트를 이용하면 해당 버전에 맞는 관련 라이브러리들을 자동으로 결정합니다. 이 기능 덕분에 라이브러리의 버전이 높거나 낮아서 정상적으로 동작이 안되는 상황을 겪을 필요가 없게 되었습니다. 이전처럼 XMl을 이용해서 라이브러리를 매번 설정하는 과정을 줄이고 개발자들이 수순한 개발에만 집중할 수 있는 환경을 제공합니다.

    - 테스트 환경과 내장 Tomcat
      
      스프링 부트를 이용해서 생성하는 프로젝트는 기본적으로 Tomcat을 내장하고 있습니다. 실행 역시 별도의 설정 없이 main 메소드를 실행하는 방식으로 서버를 구동하기 때문에 빠르게 결과를 볼 수 있는 환경을 제공합니다.

<br />

## 1.2.1 스프링 부트 내의 빈(Bean) 테스트 하기

<br />

```Java
@RestController
public class SampleController {
    
}
```

  - 기존에 스프링 프레임워크를 이용할 때에는 어노테이션을 이용하더라도 component-scan 등과 같은 별도의 설정 작업이 필요했었지만, 스프링 부트의 경우에는 자동으로 처리됩니다.(단, 패키지명을 전혀 다르게 지정한다면 스프링의 빈으로 등록되지 않으니 주의할 필요가 있습니다.)

<br />

## NOTE
```Java
기본 패키지가 아닌 패키지에 작성한 코드를 이용해서 스프링을 사용하고자 한다면, @ComponentScan 어노테이션을 이용할 수 있다.

예를 들어, 'org.sample'이라는 패키지 하위에 있는 클래스들을 스프링에 인식하고자 한다면 

@SpringBootApplication
@ComponentScan(basePackages={"org.sample", "org.zerock"})
public class Boot01Application {
    public static void main(String[] args){
        ...
    }
}
```

  - @RestController를 이용하면 JSP나 HTML 같은 별도의 뷰(View)를 활용하지 않고 문자열 데이터를 브라우저에 전송합니다.

<br />
<hr />
<br />

## 1.3 Lombok 라이브러리

  - Getter/Setter 메소드를 생성하거나 toString() 혹은 생성자 함수를 생성하는 등의 작업은 일상화된 작업이니다. 다만, 이러한 작업이 너무나 반복적으로 필요하기 때문에 개발의 생산성을 높이고 싶다면 Lombok을 이용해 단순 반복 작업을 자동화함으로써 개발 시간을 단축할 수 있습니다.

  - Lombok 라이브러리는 Java 코드를 컴파일할 때 자동으로 추가 메소드를 만들어서 컴파일해 주는 라이브러리라고 할 수 있다. 

  - Lombok은 설정(표시)해 둔 어노테이션을 기준으로 '.class'파일을 만들 때 getter/setter 등을 자동으로 추가하도록 만들 수 있기 때문에 개발자의 입장에서는 약간의 어노테이션을 추가하기만 하면 됩니다.

<br />

### Lombok의 어노테이션

  - @NonNull

     Null 값이 될 수 없다는 것을 명시합니다. NullPointerException에 대한 대비책이 될 수 있습니다.

  - @Cleanup

     자동으로 cloase() 메소드를 호출하는 역할

  - @Getter/Setter
    
     코드가 컴파일될 때 속성들에 대해서 Getter/Setter 메소드들을 생성합니다.

  - @ToString

     toString() 메소드를 생성합니다.

  - @EqualsAndHashCode

     해당 객체의 equals()와 hashCode() 메소드를 생성합니다.

  - @NoArgsConstructor / @RequiredArgsConstructor / @AllArgsConstructor

     파라미터를 받지 않는 생성자를 만들어 주거나(@NoArgsConstructor), 지정된 속성들에 대해서만 생성자를 만들거나(@RequiredArgsConstructor), 모든 속성에 대해서 생성자를 만들어 냅니다(@AllArgsConstructor).

  - @Data

     @ToString, @EqualsAndHashCode, @Getter(모든 속성), @Setter(final이 아닌 속성), @RequiredArgsConstructor를 합쳐 둔 어노테이션입니다.

  - @Value

     불변(immutable) 클래스를 생성할 떄 사용합니다.

  - @Log

     자동으로 생기는 log라는 변수를 이용해서 로그를 찍을 수 있습니다.

  - @Builder

     빌더 패턴을 사용할 수 있도록 코드를 생성합니다. new AA().setAA().setB().setC()와 같이 체이닝을 할 수 있는 코드를 생성합니다.

  - @SneakyThrows

     예외 발생 시 Throwable 타입으로 반환합니다.

  - @Synchronized

     메소드에서 동기화를 설정합니다.

  - @Getter(lazy=true)

     동기화를 이용해서 최초 한 번만 getter를 호출합니다.

<br />

### 1.3.2 Lombok 실습

```Java
@Getter
@Setter
@ToString
public class SampleVO {
    private String val1;
    private String val2;
    private String val3;
}
```

```Java
@RestController
public class SampleController {

    @GetMapping("/hello")
    public String sayHello(){
        return "Hello world";
    }

    @GetMapping("/sample")
    public SampleVO makeSample(){
        SampleVO vo = new SampleVO();
        vo.setVal1("v1");
        vo.setVal2("v2");
        vo.setVal3("v3");

        System.out.println(vo);
        return vo;
    }
}
```

## @Data 어노테이션

  - @Data를 이용하면 getter/setter를 생성하고 equals(), hashCode(), toString(), 파라미터가 없는 기본 생성자까지 자동으로 만들어 주기 때문에 무척이나 편리합니다.

  ```Java

    @ToString(exclude = {"val3"})
    public class SampleVO {
        private String val1;
        private String val2;
        private String val3;
    }
  ```
  - @Data를 적용했기 때문에 자동으로 모든 속성에 대해서 getter/setter가 생성되고, toString() 역시 작성됩니다.

  - toString()은 원하는 속성들만 출력되도록 조정을 해야 하는 경우가 종종 있다.(JPA를 이용하는 경우에 특히 많다)

  - @ToString에 exclude라는 속성을 이용해서 원하지 않는 속성을 출력하지 않도록 제어할 수 있습니다.

### NOTE
```Java
  - @Data는 주의해서 사용해야 한다.

  - @Data는 @ToString, @EqualsAndHashCode, @Getter, @Setter과 @RequireArgsConstructor의 묶음이다. 하나의 어노테이션만으로 개발에 관련된 대부분의 메소드가 자동으로 생성되기 때문에 편리한 것이 사실이지만, ORM에서 주의해야 합니다.

  - ORM은 객체와 객체가 관계를 가지는 조합의 형태로, 테이블 간의 연관관계를 표현합니다. 단, 이런 경우에는 부모 객체와 자식 객체의 toString()에서 문제가 생기게 됩니다.

[예시 코드]
public class Member{
    private String id;
    private String pw;
    private Address addr;

    @Override
    public String toString(){
        return "Member [id="+id+", pw="+pw+", addr="+addr+"]";
    }
}

public class Address {
    private String zipcode;
    private Member member;

    @Override
    public String toString(){
        return "Address [zipcode="+zipcode+", member="+member+"]";
    }
}

  - 바깥쪽(Member)의 인스턴스의 toString()을 호출하면 안쪽(Address) 객체의 toString()을 호출하고, 다시 안쪽 객체는 바깥쪽의 toString()의 toString()을 호출하는 무한 반복 호출이 진행되기 때문에 조금만 주의를 기울이지 않아도 무한 루프에 빠지면서 StackOverflow를 볼 수 있습니다.

  - @toString은 include나 exclude 같은 속성을 이용해서 toString() 작성 시에 포함되거나, 빠져야하는 인스턴스 변수를 성정할 수 있지만 @Data는 설정이 불가능합니다.

  - 몇 라인이 길어지는 한이 있더라도 차라리 @Getter, @Setter 등을 이용하는 편이 코드를 작성할 때 더 안전합니다.
```

<br />
<hr />
<br />

## 1.4 스프링 부트 프로젝트의 실행과 테스트

  - 스프링 부트의 경우 Tomcat을 내장해서 사용할 수 있으므로, 스프링 프로젝트를 진행할 때와 같이 별도의 서버를 세팅하지 않고도 프로젝트를 바로 실행할 수 있습니다.

  - 스프링 부트를 이용할 때 가장 편리한 점 중 하나는 테스트 환경을 미리 갖추고 있다는 점이다.

  - 스프링 부트에서 테스트를 진행할 수 있는 방법이 좀 더 세분화되어서 @DataJpaTest나 @JsonTest와 같이 다양한 상황에 맞는 테스트 방식을 만들 수 있습니다.

### 1.4.1 Controller의 테스트 진행

<br />

  ```Java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class SampleControllerTest {

        
    }
  ```

  - 주의 사항

    - 테스트 클래스에 @WebMvcTest 어노테이션을 추가해서 특정 컨트롤러를 지정해 줍니다.
      
       @WebMvcTest 어노테이션을 사용하면 @Component, @ControllerAdvie 등이 작성된 코드를 인식할 수 있습니다.

    - 컨트롤러를 테스트하려면 org.springframework.test.web.servlet.MockMvc 타입의 객체를 사용해야만 합니다.

       @WebMvcTest와 같이 사용하면 별도의 생성 없이 주입(@Autowired)만으로 코드를 작성할 수 있기 때문에 편리합니다.

    ```Java
    @RunWith(SpringRunner.class)
    @WebMvcTest(SampleController.class)     // @SpringBootTest
    public class SampleControllerTest {
        @Autowired
        MockMvc mock;

        @Test
        public void testHello() throws Exception{
            mock.perform(get("/hello")).andExpect(content().string("Hello World")) ;
        }
    }
    ```

    - MockMvc 객체의 경우 perform()을 이용해 객체를 브라우저에서 서버의 URL을 호출하듯이 테스트를 진행할 수 있고 결과는 andExpect()를 이용해서 확인할 수 있습니다.

    - andExpect()는 결과 확인 외에도 Response에 대한 정보를 체크하는 용도로 사용할 수 있습니다. 예를 들어, 정상적인 응답 상태이고, 응답으로 전성되는 결과를 보고 싶다면 다음과 같은 코드를 작성할 수 있습니다.

    ```Java
    MvcResult result = mock.perform(get('/hello'))
                        .andExpect(status().isok())
                        .andExpect(content().string("Hello World")).andReturn();
    System.out.println(result.getResponse().getContentAsString());
    ```

    - 스프링은 테스트 환경을 준비하려면 별도의 라이브러리를 설치해야 하고 JUnit의 버전 등 여러 가지를 신경 써야 하지만 스프링 부트는 이러한 설정이 모두 자동으로 갖추어집니다.