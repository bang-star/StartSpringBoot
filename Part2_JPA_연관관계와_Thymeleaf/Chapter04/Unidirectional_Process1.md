# 4.2 회원과 프로필 사진의 관계 - 단방향 처리 1

  - 데이터베이스상의 설계를 보면 회원과 회원 프로필은 전형적인 '일대일' 혹은 '일대다'의 관게로 설정될 수 있다.

  <img src="https://user-images.githubusercontent.com/63120360/182881993-2d0249b1-8ec7-476e-864f-939ad33309c0.png">

<br />

  - 한 명의 회원이 여러 프로필(사진)을 가질 수 있다.

  - 그중에서 하나의 사진을 현재 사진의 프로필(사진)로 이용하는 경우에는 current 칼럼의 값이 true로 지정된다.

## 4.2.1 예제 프로젝트 작성

  - Spring Data JPA를 이용할 때의 개발 순서
    
    1. 각 엔티티 클래스의 설계

    2. 각 엔티티 간의 연관관계 파악 및 설정

    3. 단방향, 양방향 결정


### NOTE
  
  ```
  Optional<T>

  Optional을 가장 쉽게 이해하는 방식은 'null을 대신하다'라고 생각하는 것이다. 

  Optional은 말그대로 존재할 수도 있고, 안할 수 도있다는 의미로 받아들이면 된다. 과거에는 코드를 통해서 결과 데이터에 null 값이 나오는지 직접 확인해야 했지만, Optional을 이용하면 null에 대해서 신경 쓰지 않고 코드를 작성할 수 있다.

  Optional을 이용할 때에는 주로 get()이나 ifPresent()를 이용하는데, 이를 통해서 결과를 반환받거나 결과를 어떤 식으로 처리할 것인지를 지정하게 됩니다.
  ```

  ```Java
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/jpa_ex?useSSL=false&serverTimezone=Asia/Seoul
    spring.datasource.username=jpa_user
    spring.datasource.password=jpa_user

    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.generate-ddl=false
    spring.jpa.show-sql=true
    spring.jpa.database=mysql
    spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

    logging.level.org.hibernate=info
  ```
  - MySQL의 경우 데이터베이스의 엔진을 MyISAM과 InnoDB로 구분한다.

     - MyISAM은 예전 MySQL부터 사용되던 엔진으로 속도면에서는 좀 더 나을 수 있지만, 데이터 무결성을 제대로 체크하지 않기 때문에 InnoDB를 권장한다.

     - spring.jpa.database-platform을 지정하지 않으면 기본적으로 MyISAM으로 저정되기 때문에, 이를 사용하기 위해 MySQL5InnoDBDialect를 명시적으로 지정한다.

     - InnoDB로 지정하지 않는 경우에는 외래키 대신에 인덱스가 생성되므로 주의해야 한다.

## 4.2.2 각각의 엔티티 클래스 설계

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_members")
    @EqualsAndHashCode(of = "uid")
    public class Member {
        @Id
        private String uid;
        private String upw;
        private String uname;
    }
  ```

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_profile")
    @EqualsAndHashCode(of = "uid")
    public class Profile {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long fno;

        private String name;

        private boolean current;
    }
  ```

  - Member와 Profile에는 추가적인 속성들이 존재할 수 있다. 

     - 대표적으로는 특별한 경우가 아니라면 모든 데이터에는 '최초 생성 시간'과 '최종 수정 시간'을 기록해 주는 칼럼을 가지도록 설계해 주는 것이 좋다.

<br />

### NOTE
```
GenerationType.AUTO와 GenerationType.IDENTITY

인티티의 식별키를 처리하는 여러 방식(전략)중에서 GenerationType.AUTO는 데이터베이스에 맞게 자동으로 식별키를 처리하는 방식으로 동작한다.

MySQL의 경우 Spring Boot 1.5.4 버전을 이용할 때 AUTO로 지정하면 칼럼이 auto_increment로 지정되었다.

반면에 2.0.0 버전의 경우 AUTO로 지정하면 hibernate_sequence라는 테이블을 생성하고 번호를 유지하는 방식으로 변경되었다.

Profile 클래스의 @Id 생성 타입을 GenerationType.AUTO로 지정하면 hibernate_sequence라는 테이블이 생성되는 것을 볼 수 있다.

hibernate_sequence 테이블은 자동으로 생성되는 모든 엔티티들이 공유하는 테이블이 되어 비연속적인 데이터들이 생성될 수 있다.

hibernate_sequence 테이블이 존재하면, 이후에 GenerationType을 IDENTITY로 변경하더라도 auto_increment로 처리되지 않기 때문에, 테이블을 삭제한 후에 다시 생성해 주어야만 한다.

MySQL에서 GenerationType.IDENTITY를 이용할 경우에는 반드시 hibernate_sequence 테이블이 존재하는지 확인한 후에 실제 테이블 생성시 auto_increment가 처리되는지를 확인하는 습관을 가지는 것이 좋다.
```

### 4.2.3. 연관관계의 설정과 단방향/양방향

  - 회원과 프로필 사진의 관계는 일대다

  - 프로필 사진과 회원의 관계는 다대일

  - 관계형 데이터베이스에서는 단순히 하나의 FK를 이용해서 지정되는 상황이 JPA를 이용하는 복잡해진 상황을 볼 수 있다.

  - 판단

    - '회원'에서 '프로필'로의 접근만을 사용하는가?

        즉, Member 클래스에만 Profile타입의 인스턴스 변수를 추가하는가?

    - '프로필'을 통해서 '회원'정보를 조회할 필요가 있는가?

        즉, Profile 클래스에도 Member 타입의 인스턴스 변수를 추가할 것인가?

    - 판단은 @Query에서 중요한 역할을 하게 된다.

- JPA에서는 이러한 설정을 '단반향/양방향' 관계로 구분해 준다.

    - 단반향

        '일방 통행'의 참조

        예를들어, 회원 정보를 통해서만 프로필 정보를 볼 수 있는 구조

    - 양방향

        프로필 사진을 먼저 알아내고, 이를 통해서 회원 정보를 알아내는 경우도 있는 상황에 해당

- 회원과 프로필 관계를 보면 프로필 테이블에 회원 정보가 입력되어야 하므로, 가장 쉽게 생각할 수 있는 구조는 Profile 클래스에 Member 클래스 타입을 인스턴스 변수로 설계하는 것이다.

```Java
@Getter
@Setter
@ToString(exclude = "member")
@Entity
@Table(name = "tbl_profile")
@EqualsAndHashCode(of = "fname")
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long fno;

    private String fname;

    private boolean current;

    private Member member;
}
```

### JPA의 연관관계 어노테이션 처리

  - JPA에서의 연관관계를 크게 @OneToOne, @OneToMany, @ManyToOne, @ManyToMany의 어노테이션을 이용해서 처리한다.

   ```Java
    @Getter
    @Setter
    @ToString(exclude = "member")
    @Entity
    @Table(name = "tbl_profile")
    @EqualsAndHashCode(of = "fname")
    public class Profile {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long fno;

        private String fname;

        private boolean current;

        @ManyToOne
        private Member member;
    }
   ```

### 생성된 테이블 구조의 확인

 - JPA의 연관관계 설정은 JPA를 이용할 때 가장 중요한 부분이므로, 가능하면 연관관계를 맺은 후 프로젝트를 실행하여 올바른 구조로 생성되는지를 중간중간 확인해 주는 것이 좋다.

### 4.2.4 Repositry

  - 엔티티 클래스마다 Repository를 설계해 줄 수도 있지만, 연관관계의 설정에 따라서는 Repository를 설계할 필요가 없을 수도 있다.

     - Member를 처리하는 Repository를 생성해서 회원 데이터를 처리하는 것이 명확하다.

     - Profile을 저장할 때는 Member 객체를 통해서 Profile을 처리할 수 없기 때문에 Profile을 처리하는 Repository를 설계하는 것이 보편적이다.

  ```Java
  public interface MemberRepository extends CrudRepository<Member, String> {}
  public interface ProfileRepository extends CrudRepository<Profile, Long> {}
  ```

### 4.2.5 테스트를 통한 검증

 ```Java
 @RunWith(SpringRunner.class)
@SpringBootTest
@Log
@Commit     // 테스트 결과 commit
public class ProfileTests {
    
    @Autowired
    MemberRepository memberRepository;
    
    @Autowired
    ProfileRepository profileRepository;
}
 ```

  - @Log는 Lombok의 로그를 사용할 때 이용하는 어노테이션

  - @Commit은 테스트 결과를 데이터베이스에 commit하는 용도

  ```Java
  // 더미 회원 데이터의 추가
    @Test
    public void testInsertMembers(){
        IntStream.range(1, 101).forEach(i -> {
            Member member = new Member();
            member.setUid("user"+i);
            member.setUpw("pw"+i);
            member.setUname("name"+i);
            memberRepository.save(member);
        });
    }
  ```

### 특정 회원의 프로필 데이터 처리

  - Profile 데이터는 반드시 Member 객체에 대한 참조가 필요하다.

    - MemberRepository를 이용해 실제 Member 객체를 가져와서 처리할 수 있지만, 이 경우 Member를 읽고 난 후 Profile을 읽어야 하기 때문에 좋은 방법이라고 할 수 없다.

    - Member의 식별자인 uid 속성뿐이므로 Member 객체만 잠시 생성해서 uid만을 지정하는 방식이 더 효율적이다.

### 4.2.6 단방향의 문제와 Fetch Join

  - Member의 인스턴스는 Profile 인스턴스와 아무런 관계가 없이 설정되어 있기 때문에 단순 CRUD 작업을 하기에는 편리하지만, 실제로는 여러 문제가 발생할 수 있다.

    - '회원 정보를 조회하면서 회원의 현재 프로필 사진도 같이 보여주어야 한다'와 같은 복합적인 요구사항이 있는 경우

    - 회원 정보 조회는 MemberRepository를 확인할 수 있지만, 회원의 프로필 사진은 별도의 처리가 필요하다.

    - 이에 대한 해결책은 ProfileRepository에 findByMember()와 같은 쿼리 메소드를 설계하는 방법도 고려할 수 있지만, 좋은 방법이 아니다.

        - 회원 정보가 리스트의 형태로 페이징 처리가 된다고 가정하면 한 페이지에 보이는 모든 Member 인스턴스에 대해서 ProfileRepository의 메소드를 호출하게 된다.

        - (예시) 정보를 조회할 회원이 20명이라면 프로필 정보를 얻기 위해 20번의 SQL을 실행

    - 순수한 SQL을 이용할 경우 INNER JOIN이나 OUTER JOIN, 서브 쿼리 등을 이용해서 쉽게 처리할 수 있지만, JPA를 이용할 때는 많은 제약이 따르게 된다.

    - JPA에서 @Query를 이용하여 nativeSQL을 그대로 이용하는 것이 가능하지만, 데이터베이스에 종속되기 때문에 권장하지 않는다.

    - JPA에서 'Fetch Join"이라는 기법을 통해서 SQL에서 조인을 처리하는 것과 유사하게 작업을 처리할 수 있다. 

        - 'Fetch Join'은 SQL과 달리 JPQL을 이용해서 처리하는 점이 다르다고 생각하면 된다.

### JPA의 Join 처리

  - @Query에서 사용하는 JPQL 자체가 테이블을 보는 것이 아니라, 클래스를 보고 작성하기 때문에 Hibernate 5.0의 경우에는 참조 관계가 없는 다른 에티티를 사용하는 것은 불가능하다.

  - Spring 2.0 이상에서 Hibernate 5.2 이상에서는 참조관계가 없어도 'ON' 구문을 이용해서 LEFT OUTER JOIN을 처리할 수 있다.

    - 예를 들어 "uid가 'user1'인 회원의 정보와 더불어 회원의 프로필 사진 숫자를 알고 싶다"고 요구했을 때

    <br />

    - 순수 SQL
        ```SQL
        select member.uid, count(fname)
        from tbl_members member LEFT OUTER JOIN tbl_profile profile ON profile.member_uid = member.uid
        where member.uid = 'user1'
        group by member.uid
        ```
  
    - @Query를 이용한 방법

        ```Java
        public interface MemberRepository extends CrudRepository<Member, String> {
        @Query("SELECT m.uid, count(p) FROM Member m LEFT OUTER JOIN Profile p " + "ON m.uid = p.member WHERE m.uid = ?1 GROUP BY m")
        public List<Object[]> getMemberWithProfileCount(String uid);
        }
        ```

        - @Query 안의 JPQL의 경우 SQL과 유사하지만, 테이블 대신에 인티티 클래스를 이용한다는 것이 가장 큰 차이다.

        - 메소드의 리턴 타입은 List<Object[]>로 처리된다.

        - JPQL에서 엔티티 타입뿐 아니라 다른 자료형들도 반환할 수 있기 때문에 List는 결과의 Row 수를 의미하고, Object[]는 칼럼들을 의미한다.

        - 테스트 코드 
        ```Java
        @Test
        public void testFetchJoin1(){
            List<Object[]> result = memberRepository.getMemberWithProfileCount("user1");

            result.forEach(arr -> System.out.println(Arrays.toString(arr)));
        }
        ```

        - 실행결과

            ```SQL
            Hibernate: select member0_.uid as col_0_0_, count(profile1_.fno) as col_1_0_ from tbl_members member0_ left outer join tbl_profile profile1_ on (member0_.uid=profile1_.member_uid) where member0_.uid=? group by member0_.uid
            
            [user1, 4]
            ```

    - '회원 정보와 현재 사용 중인 프로필에 대한 정보'를 얻고자 한다면 JPQL을 작성할 수 있다.

        ```Java
        public interface MemberRepository extends CrudRepository<Member, String> {
            
            @Query("SELECT m, p FROM Member m LEFT OUTER JOIN Profile p " + "ON m.uid = p.member WHERE m.uid = ?1 AND p.current = ture")
            public List<Object[]> getMemberByProfile(String uid);
            }
        ```