# 4.3 자료실과 첨부파일의 관계 - 단방향2

  - '단방향'으로 연관관계를 처리하는 경우네느 한쪽만 참조를 하기 때문에 일대다, 다대일에서 어느 족에 참조에 대한 설정을 두는지를 세심하게 결정해야 한다.

    - Member 클래스에서 Profile에 대한 참조가 없었고, Profile에서는 Member를 참조하는 형태였다.
  
    - 단방향 설계이므로 Member에서 Profile에 대한 참조를 이용하고, Profile에서는 참조를 하지 않는 설계 가능하다. 

        - 이 경우 @JoinTable이라는 설정을 이용한다.

    <br />

    <img src="https://user-images.githubusercontent.com/63120360/182920028-b7f85c63-1ee7-495f-bf02-b8c34cb61539.png">

    <br />

## 4.3.1 엔티티 클래스 작성

```Java
@Getter
@Setter
@ToString
@Entity
@Table(name = "tbl_pds")
@EqualsAndHashCode(of = "pid")
public class PDSBoard {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long pid;
    private String pname;
    private String pwriter;
}

@Getter
@Setter
@ToString
@Entity
@Table(name = "tbl_pdsfiles")
@EqualsAndHashCode(of = "fno")
public class PDSFile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long fno;

    private String pdsfile;
}
 ```

 ### 연관관계 설정

  - 자료실의 자료와 첨부 파일의 관계는 '일대다', '다대일'의 관계이고 PDSBoard 쪽에서 단방향으로 연관관계를 설정

  ```Java
    @Getter
    @Setter
    @ToString
    @Entity
    @Table(name = "tbl_pds")
    @EqualsAndHashCode(of = "pid")
    public class PDSBoard {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long pid;
        private String pname;
        private String pwriter;
        
        @OneToMany
        private List<PDSFile> files;
    }
  ```

  - 실행을 하게 되면 테이블이 3개가 생성된다.

     - tbl_pds

     - tbl_pdsfiles

     - tbl_pds_files(자동으로 생성된 테이블)

     - 테이블이 생성된 이유는 @OneToMany를 데이터베이스가 어떻게 처리하는지 생각해 보면 알 수 있다.

        (예시) 
        
        tbl_pds에 칼럼이 생성된다면 여러 개의 tbl_pdsfiles의 fno를 저장해야 한다. 때문에 데이터베이스의 칼럼에 하나의 값이 아닌 여러 개의 값을 저장해야 한다. 

        @OneToMany가 지정되면 JPA에서는 무조건 여러 개의 데이터를 저장하기 위해 별도의 테이블을 생성하게 된다.

     - 단방향 @OneToMany에서 별도의 테이블이 생성되는 것을 원하지 않는다면, 별도의 테이블 없이 특정한 테이블을 조인할 것이라고 명시하거나(@JoinTable), @JoinColumn을 명시해 기존 테이블을 이용해서 조인한다고 표현해 주어야 한다.

     - @JoinTable은 자동으로 생성되는 테이블 대신에 별도의 이름을 가진 테이블을 생성하고자 할 때 사용하고, @JoinColumn은 이미 존재하는 테이블에 컬럼을 추가할 떄 사용한다.

        ```Java
        @Getter
        @Setter
        @ToString(exclude = "files")
        @Entity
        @Table(name = "tbl_pds")
        @EqualsAndHashCode(of = "pid")
        public class PDSBoard {

            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long pid;
            private String pname;
            private String pwriter;

            @OneToMany
            @JoinColumn(name = "pdsno")
            private List<PDSFile> files;
        }
        ```

     - @ToString()을 변경하고, @JoinColumn을 이용해서 변경되었다.

     - SQL을 실행하면 2개의 테이블만이 생성되고, tbl_pdsfiles 테이블에 pdsno라는 이름이 칼럼이 추가된 것을 볼 수 있다.

## 연관관계에 따른 Repositry

 - 연관관계의 구조에 따라서 Repository의 설계 역시 영향을 받게 된다.

 - 현재 PDSFile의 경우 PDSBoard에 대한 참조가 없기 때문에 문제가 된다.

 - PDSFile이 저장되는 tbl_pdsfiles 테이블에 tbl_pds의 pid를 참조해서 값이 들어가야 한다.

    - PDSFile 클래스에 PDSBoard에 대한 참조가 없기 때문에 단독으로 처리할 수 없다.

    - PDSBoard는 모든 PDSFile 객체들의 참조를 보관할 수 있으므로, 원하는 모든 데이터에 대한 처리가 가능하다. 

 - 불평등한 관계를 처리하기 위해서 각각 Repository를 생성하는 대신에 'One'에 해당하는 엔티티 객체에 대한 Repository만을 이용하는 것이 좋습니다.(도메인 주도 설계(Domain Driven Design)에서는 Aggregation(집합)이라고 표현합니다.)

  ```Java
    public interface PDSBoardRepository extends CrudRepository<PDSBoard, Long> {
        
    }
  ```

## 4.3.2 등록과 Cascading 처리

  - 새로운 자료가 등록될 때 자료와 첨부된 파일을 동시에 등록하는 경우를 테스트할 것이다.

  ```Java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @Log
    public class PDSBoardTests {
        @Autowired
        PDSBoardRepository pdsBoardRepository;

        @Test
        public void testInsertPDS(){
            PDSBoard pds = new PDSBoard();
            pds.setPname("DOCUMENT 1- 2");

            PDSFile file1 = new PDSFile();
            file1.setPdsfile("file1.doc");

            PDSFile file2 = new PDSFile();
            file2.setPdsfile("file2.doc");

            pds.setFiles(Arrays.asList(file1, file2));

            log.info("try to save pds");
            pdsBoardRepository.save(pds);
        }
    }
  ```

  - testInsertPds()는 1개의 자료와 2개의 첨부 파일을 저장하려고 시도하면 Repository는 PDSBoardRepository이므로 실제로 save 작업은 한 번만 호출하게 된다.

  - 문제가 발생하는 이유는 JPA에서 한 번에 여러 엔티티 객체들의 상태를 변경해주어야 하기 때문이다.
   
     - 한 번에 PDSBoard 객체로 보관해야 하고, PDSFile의 상태도 보관해야만 하기 때문이다.

  - JPA에서 처리하려는 엔티티 객체의 상태에 따라서 종속적인 객체들의 영속성도 같이 처리되는 것을 '영속성 전이'라고 한다.

     - 영속성 전이라는 개념이 겉으로 보기엔 데이터베이스의 트랜잭션과 유사한 것 같지만, JPA에서는 메모리상의 객체가 엔티티 매니저의 컨텍스트에 들어가는 '영속, 준영속, 비영속'등의 개념이 존재하기 때문에 더 복잡한 상태가 된다.

    <br />

     (예시) 
     
     날짜(부모 엔티티)와 일정(자식 엔티티)을 생각해 보면 된다.
     특정한 날짜가 데이터베이스에서 사라지게 되면 거기에 해당하는 일정들 역시 같이 삭제되어야 한다.

     JPA에서는 엔티티들이 기본적으로 메모리상의 관계이므로, 날짜 객체가 사라질 때 일정 객체 역시 삭제될 필요가 있다. 영속성 전이는 부모 엔티티나 자식 엔티티의 상태 변호가 자신과 관련 있는 엔티티에 영향을 주는 것을 의미한다.

  - JPA에서 종속적인 엔티티의 영속성 전이에 대한 설정

    - ALL
      
       모든 변경에 대한 전이

    - PERIST

       저장 시에만 전이

    - MERGE

       병합 시에만 전이

    - REMOVE

       삭제 시에만 전이

    - REFRESH

       엔티티 매니저의 refresh() 호출 시 전이

    - DETACH

       부모 엔티티가 detach되면 자식 엔티티 역시 detach

     - 영속성 전이에 대한 설정 중 ALL은 모든 엔티티의 상태 변화에 같이 처리되는 옵션이므로 가장 손쉽게 사용할 수 있다.

     - ALL을 사용하려면 PDSBoard에서 @OneToMany 속성에 cascade 속성을 지정해야 한다.

    ```Java
    @Getter
    @Setter
    @ToString(exclude = "files")
    @Entity
    @Table(name = "tbl_pds")
    @EqualsAndHashCode(of = "pid")
    public class PDSBoard {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long pid;
        private String pname;
        private String pwriter;

        @OneToMany(cascade = CascadeType.ALL)
        @JoinColumn(name = "pdsno")
        private List<PDSFile> files;
    }
    ```

    ```SQL
    Hibernate: insert into tbl_pds (pname, pwriter) values (?, ?)
    Hibernate: insert into tbl_pdsfiles (pdsfile) values (?)
    Hibernate: insert into tbl_pdsfiles (pdsfile) values (?)
    Hibernate: update tbl_pdsfiles set pdsno=? where fno=?
    Hibernate: update tbl_pdsfiles set pdsno=? where fno=?
    ```

## 4.3.3 첨부 파일 수정과 @Modifying, @Transcational

 - 데이터베이스 처리를 위해서 Repository가 필수적인데, PDSBoardRepository만이 존재하여 첨부 파일의 이름만 수정해야 하는 상황에서 PDSFile에 대한 Repository가 존재하지 않는다. 이러한 경우에는 @Query를 이용해서 처리하면 편리하다.

   - @Query는 기본적으로 'select' 구문만을 지원하지만 @Modifying을 이용해서 DML(insert, update, delete) 작업을 처리할 수 있다.

    <br />

   ```Java
    public interface PDSBoardRepository extends CrudRepository<PDSBoard, Long> {

        @Modifying
        @Query("update PDSFile f set f.pdsfile = ?2 WHERE f.fno = ?1 ")
        public int updatePDSFile(Long fno, String newFileName );
    }
   ```

   ```Java
    @Transactional
    @Test
    public void testUpdateFileName1(){
        Long fno = 1L;
        String newName = "updatedFile.doc";

        int count = pdsBoardRepository.updatePDSFile(fno, newName);

        // @Log 설정된 이후 사용 가능
        log.info("update count : "+count);
    }
   ```

   - testUpdateFileName1()은 @Query를 이용해서 'update ...'문을 실행하는데, 'delete'나 'update'를 사용하는 경우에는 반드시 @Transactional 처리를 필요로 한다.

     - 로그에 'Rolled back transaction...'이라고 출력되고 있는 것을 볼 수있다.

     - 데이터베이스의 테이블을 보면 정상적으로 업데이트가 되지 않은 것을 확인할 수 있다.

     - 정상적인 처리가 되었음에도 불구하고 데이터베이스의 최종 결과가 변경되지 않는 것은 스프링의 테스트에서 @Transactional이 기본적으로 롤백 처리를 시도하기 때문이다.

     - 롤백 처리를 원하지 않는다면 PDSBoardTests.java의 선언부에 @Commit을 추가해서 자동으로 commit 처리가 되도록 지정할 수 있습니다.

### 추가한 객체를 통한 파일 수정

  - @Query와 @Modifying을 이용하는 것이 편리하지만, 필요한 경우에는 전통적인 접근 방식을 이용해서 첨부 파일의 이름을 수정할 수 있다.

  - 전통적인 방식은 PDSBoardRepository에서 PDSBoard를 얻어온 후에 내용물인 PDSFile을 수정하고, save()를 이용해서 업데이트를 진행하는 방법이다.

  ```Java
    @Transactional
    @Test
    public void testUpdateFileName2(){  
        String newName = "updatedFile2.doc";
        
        // 반드시 번호가 존재하는지 확ㄷ인할 것
        Optional<PDSBoard> result = pdsBoardRepository.findById(2L);
        
        result.ifPresent(pds -> {
            log.info("데이터가 존재하므로 update 시도");
            PDSFile target = new PDSFile();
            target.setFno(2L);
            target.setPdsfile(newName);
            
            int idx = pds.getFiles().indexOf(target);
            if(idx > -1) {
                List<PDSFile> list = pds.getFiles();
                list.remove(idx);
                list.add(target);
                pds.setFiles(list);
            }
            
            pdsBoardRepository.save(pds);
        });
    }
  ```

  - 테스트 코드의 내용은 PDSBoardRepository에서 pid가 '2'인 PDSBoard 객체를 얻는 것으로 시작한다

  - Spring Boot 2.0을 이용하면 findById()의 리턴 타입이 Optional 타입이기 때문에 ifPresent()를 이용해서 데이터가 존재한다면 어떠한 처리를 할 것인지 람다식으로 작성할 수 있다.

    - 추출한 PDSBoard에서 fno값이 2번인 PDSFile 객체를 찾아내고, 첨부 파일의 목록에서 삭제한 후 새로운 데이털르 추가한다.

    - PDSFile 클래스는 fno 값으로 equals()와 hashcode()를 사용한다.

    - setFile()을 이용해서 새롭게 갱신된 첨부 파일의 목록을 설정하고, save()작업을 진행한다.

    ```
    com.example.PDSBoardTests : 데이터가 존재하므로 update 시도

    Hiberante : select files0_.pdsno as pdsno3_2_0_, files0_.fno as fno1_2_0, files0_.fno as fno1_2_1, files0_pdsfile as pdsfile2_2_1 from tbl_pdsfiles files0_ where files0_pdsno=?

    Hibernate : update tbl_pdsfiles set psdsfile=? where fno=?
    ```

    - 실행된 SQL을 보면 tbl_pds 테이블에서 데이터를 조회하는 부분과 tbl_pdsfiles 테이블에서 조회하는 부분이 별개로 실행되고 있는데, 이는 JPA에서 연관관계의 테이블을 조회할 때 지연 로딩(lazy loading)을 하기 때문이다.

<br />

## 4.3.4 첨부 파일 삭제

 - 수정과 마찬가지로 '자료'의 첨부 파일을 삭제하는 작업은  @Query를 이용하거나, 객체를 통해서 접근하는 방식을 사용할 수 있다.

  ```Java
    public interface PDSBoardRepository extends CrudRepository<PDSBoard, Long> {

        @Modifying
        @Query("delete from PDSFile f WHERE f.fno = ?1")
        public int deletePDSFile(Long fno);
    }
  ```

  ```Java
    @Transactional
    @Test
    public void deletePDSFile(){
        // 첨부 파일 번호
        Long fno = 2L;

        int count = pdsBoardRepository.deletePDSFile(fno);
        log.info("DELETE PDSFILE : " + count);
    }
  ```

  - 실행되는 SQL

        Hibernate : delete from tbl_pdsfile where fno=?

        com.example.PDSBoardTests       : DELETE PDSFile : 1


<br />

## 4.3.5 조인 처리

  - 실제화면에서 '특정 자료의 번호와 자료의 제목, 첨부 파일의 수'를 같이 보여주어하는 상황이 있을 수 있다. 이러한 상황에서 @Query를 이용해서 조인을 처리하여 해결할 수 있다.

  - tbl_pds와 tbl_pdsfiles에는 데이터가 충분하지 않으므로, 테스트 코드를 이용해서 여러 개의 데이터를 추가할 필요가 있다.

  ```Java
  @Test
    public void insertDummies(){

        List<PDSBoard> list = new ArrayList<>();

        IntStream.range(1, 100).forEach(i -> {
            PDSBoard pds = new PDSBoard();

            pds.setPname("data"+i);

            PDSFile file1 = new PDSFile();
            file1.setPdsfile("file1.doc");

            PDSFile file2 = new PDSFile();
            file2.setPdsfile("file2.doc");

            pds.setFiles(Arrays.asList(file1, file2));

            log.info("try to save pds");
            list.add(pds);
        });


        pdsBoardRepository.saveAll(list);
    }
  ```

  - insertDummies()는 100개의 자료와 200개의 파일 데이터를 추가한다.

  - 기존의 데이터를 추가하는 기능과 다른 점은 각각의 데이터를 save()하지 않고, List에 데이터들을 보관했다가, 한 번에 저장하는 방식이다.

    - (예시) 자료와 첨부 파일의 수를 자료 번호를 역순으로 출력해라.

    ```Java
    @Query("SELECT p, count(f) FROM PDSBoard p LEFT OUTER JOIN p.files f " +
            "ON p.pid = f WHERE p.pid > 0 GROUP BY p ORDER BY p.pid DESC ")
    public List<Object[]> getSummary();
    ```

    - 단방향의 경우와 조금 다른 점이라면 PDSBoard 객체의 files를 이용해서 Outer Join을 처리하고 있다는 점이다.

    ```Java
    @Test
    public void viewSummary(){

        pdsBoardRepository.getSummary().forEach(arr -> {
            log.info(Arrays.toString(arr));
        });
    }
    ```

    - viewSummery()의 실행 결과는 첨부 파일의 존재 여부에 관계없이 처리되는 것을 볼 수 있다.

    - JPA를 이용해서 객체를 설계할 때에 단방향으로 설정하는 가장 큰 기준에 대해서는 여러 의견들이 존재하다.

       - 어떤 방식이 옳고 그르다고 말하기 전에 필요한 데이터의 가공에 대한 고민이 우선되어야 한다.

       - 단방향의 경우 한쪽의 객체만을 이용한다는 점은 편리하게 다가올 수 있지만, 조인이 필요한 경우에는 좀 더 신중히 고려할 필요가 있다.