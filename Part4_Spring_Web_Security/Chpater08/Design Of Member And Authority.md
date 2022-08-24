# 8.2 회원과 권한 설계

  - Spring Data JPA를 이용해 서비스를 이용하는 회원과 각 회원이 가지는 권한을 생성할 것이다.

  - 회원에 대한 용어는 Member를 이용할 것이다. 
    
    일반적으로 User라는 용어를 사용하는 경우가 흔하지만, User라는 타입은 이미 스프링 시큐리티에서 이용하고 있기 때문에 혼란을 피하기 위해 Member라는 이름을 이용한다.

  - 회원(Member)은 등급이나 권한을 가지도록 설계한다.

    실제 서비스에서 Access Control List(접근 제한 목록 ACL)라는 것을 작성해서 특정 리소스나 작업에 대한 권한을 가진 사용자들만이 접근이나 수정 등의 작업을 할 수 있도록 제어한다.

<br />
<hr />
<br />

## 8.2.1 도메인 클래스 설계

  - 회원(Member)과 회원 권한(MemberRole)은 클래스로 설계

  - 회원(Member) 클래스

    ```Java
    // Member.java

    @Getter
    @Setter
    @Entity
    @Table(name = "tbl_members")
    @EqualsAndHashCode(of = "uid")
    @ToString
    public class Member {
        
        @Id
        private String uid;
        
        private String upw;
        
        private String uname;
        
        @CreationTimestamp
        private LocalDateTime regdate;
        @UpdateTimestamp
        private LocalDateTime updatedate;
    }
    ```

    - Member 클래스는 회원의 아이디를 의미하는 uid와 upw, uname 등의 속성을 가지도록 설계한다.

    - Spring Security에서 username, password 등의 용어를 사용하므로 가능하면 충돌이 나지 않도록 이름을 지정하는 것이 좋다.

  - 회원 권한(MemberRole) 클래스

    ```Java
    // MemberRole.java
    
    @Getter
    @Setter
    @Entity
    @Table(name = "tbl_member_roles")
    @EqualsAndHashCode(of = "fno")
    @ToString
    public class MemberRole {
        
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long fno;
        
        private String roleName;
    }
    ```

### 연관관계의 설정

  - 클래스를 관계

    - Member와 MemberRole은 '일대다', '다대일'의 관계

    - MemberRole 자체가 단독으로 생성되는 경우는 거의 없으므로, Member가 MemberRole을 관리하는 방식의 설계

  - MemberRole은 자체로는 별다른 의미를 가지지 못하기도 하고, Member에 대한 정보의 라이프사이클과 강하게 묶여있기 때문에, Member가 MemberRole에 대한 참조를 가지도록 한다.

    ```Java
    // Member.java

    public class Member {
        ...

        @OneToMany
        @JoinColumn(name = "member")
        private List<MemberRole> role;
    }
    ```

  - MemberRole은 정보에 대한 접근 방식 자체가 '회원'을 통한 접근이므로, 별도의 연관관계를 설정하지 않는 단방향 방식을 선택한다.

    ```SQL
     <!-- 실행결과 -->
    Hibernate: create table tbl_member_roles (fno bigint not null auto_increment, role_name varchar(255), member varchar(255), primary key(fno)) engine=InnoDB

    Hibernate: create table tbl_members (uid varchar(255) not null, regdate datetime, uname varchar(255), updatedate datetime, upw varchar(255), primary key(uid)) engine=InnoDB

    Hibernate: alter table tbl_member_roles add constraint FKiy7scif7laghe23vht49x2aka foreign key (member) references tbl_members (uid)
    ```

<br />
<hr />
<br />

## 8.2.2 Repository 생성

  - MemberRepository

    ```Java
    public interface MemberRepository extends CrudRepository<Member, String> {

    }
    ```

<br />
<hr />
<br />

## 8.2.3 테스트를 통한 데이터 추가/조회

  - MemberTests

    ```Java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @Log
    @Commit
    public class MemberTests {
        
        @Autowired
        private MemberRepository repo;

        @Test
        public void testInsert(){
            
            for(int i=0; i<=100; i++){
                
                Member member = new Member();
                member.setUid("user"+i);
                member.setUpw("pw"+i);
                member.setUname("사용자"+i);

                MemberRole role = new MemberRole();
                if(i<=80){
                    role.setRoleName("BASIC");
                }else if(i<=90){
                    role.setRoleName("MANAGER");
                }else{
                    role.setRoleName("ADMIN");
                }
                
                member.setRoles(Arrays.asList(role));
                repo.save(member);
            }
        }
    }
    ```

  - testInsert()는 Member 엔티티와 MemberRole 엔티티를 동시에 저장하기 때문에 에러가 발생한다.

    ```Log
    org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: org.zerock.domain.MemberRole; nested exception is java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: org.zerock.domain.MemberRole
    ```

    - 에러의 원인은 엔티티들의 영속관계를 한 번에 처리하지 못해서 발생했으므로, 이에 대한 처리는 cascade 설정을 추가해야한다.

        ```Java
        // Member.java

        public class Member {
            ...
            @OneToMany(cascade = CascadeType.ALL)
            @JoinColumn(name = "member")
            private List<MemberRole> roles;
        }
        ```

  - 조회

    ```Java
    @Test
    public void testRead(){

        Optional<Member> result = repo.findById("user85");

        result.ifPresent(member -> log.info("member : "+ member));
    }
    ```

    - testRead() 에러

        ```Error
        org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: org.zerock.domain.Member.roles, could not initialize proxy - no Session
        ```

     - 에러의 원인

        tbl_members와 tbl_member_roles 테이블을 둘 다 조회해야 하기 때문이다. 트랜잭션 처리를 해주거나, 즉시 로딩을 이용해서 조인을 하는 방식으로 처리해야 한다.

    - 해결 방법

        권한 정보는 회원 정보와 같이 필요한 경우가 많기 때문에, fetch 모드를 즉시 로딩으로 설정

        ```Java
        // Member.java
        public class Member {
            ...
            @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
            @JoinColumn(name = "member")
            private List<MemberRole> roles;
        }
        ```