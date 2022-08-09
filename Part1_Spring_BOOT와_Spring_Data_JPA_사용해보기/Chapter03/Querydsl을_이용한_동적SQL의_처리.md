# 3.3 Querydsl을 이용한 동적 SQL의 처리

  - 쿼리를 처리하다 보면 다양한 상황에 맞게 쿼리를 생성하는 경우가 많다. 대표적인 케이스가 다양한 검색 조건에 대해서 쿼리를 실행해야 하는 경우라고 할 수 있다.

     - 쿼리 메소드나 @Query를 이용하는 경우에 개발 속도는 좋지만 고정적인 쿼리만을 생산합니다.

     - 동적인 상황에 대한 처리를 위해서 Querydsl을 이용한다.

     - Querydsl의 dsl은 'Domain Specific Language'의 약자로 특정 도메인 객체를 조회한다는 의미이다. Hibernate의 경우 Criteria라는 것을 주로 이용해왔지만, Spring Data JPA를 이용하는 경우 Querydsl이 편리하다.

  - Querydsl를 이용하기 위한 과정

     1. pom.xml의 라이브러리와 Maven 설정의 변경 및 실행

     2. Predicate의 개발

     3. Repository를 통한 실행

### 3.3.1 Querydsl을 위한 준비

  - Querydsl은 Java를 이용해서 쿼리 조건을 처리할 때 사용하는 라이브러리라고 표현할 수 있다.

    - 개발자는 SQL을 직접 처리하지 않고, Querydsl을 이용해서 필요한 조건을 처리하는 Java 코드를 생성하고, Repository를 통해서 이를 처리한다.

    - Querydsl을 이용하는 경우 엔티티 클래스는 Querydsl에서 사용하는 '쿼리 도메인 클래스'를 생성해서 처리해야 하는데 이를 위해서 pox.xml과 Maven에 추가적인 변경 작업이 필요하다.


### Querydsl의 라이브러리

  - querydsl은 두 종류의 라이브러리가 있으므로 주의해야 한다.

  ```xml
  	<dependency>
			<groupId>com.querydsl</groupId>
			<artifactId>querydsl-apt</artifactId>
			<version>4.1.4</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.querydsl</groupId>
			<artifactId>querydsl-jpa</artifactId>
			<version>4.1.4</version>
		</dependency>
  ```

  - Querydsl은 JPA를 처리하기 위해서 엔티티 클래스를 생성하는 방식을 이용한다.

    - 이를 'Qdomain'이라고 하는데, 동적 쿼리를 생성할 때 사용하기 때문에 반드시 이를 실행할 수 있는 방법이 있어야 한다.

    - Qdomain 클래스를 생성하는 작업을 위한 코드 생성기가 필요하다.
 
   ```xml
   <plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
			<!-- ADDED FOR Querydsl -->
			<plugin>
				<groupId>com.mysema.maven</groupId>
				<artifactId>apt-maven-plugin</artifactId>
				<version>1.1.3</version>
				<executions>
					<execution>
						<goals>
							<goal>process</goal>
						</goals>
						<configuration>
							<outputDirectory>target/generated-sources/java</outputDirectory>
							<processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
   ```

   - 정상적으로 설정이 되었다면 프로젝트 내에 tart이라는 폴더가 생성된다.

## 3.3.2 Predicate 준비

 - Predicate는 '이 조건이 맞다'고 판단하는 근거를 함수로 제공하는 것이다.

 - 함수형 패러다임을 가진 언어들에서 자주 사용되는데, Java 8 버전에도 포함되어 있다.

 - Repository에서 Predicate를 파라미터로 전달하기 위해서는 QueryDslPredicateExecutor 인터페이스를 Repository에 추가해 주어야만 한다.

 - QeuryDslPredicateExecutor 인터페이스의 메소드
   ```
    - long count(Predicate) : 데이터의 전체 개수

    - boolean exists(Predicate) : 데이터의 존재 여부

    - Iterable<T> findAll(Predicate) : 조건에 맞는 모든 데이터

    - Page<T> findAll(Predicate) : 조건에 맞는 모든 데이터

    - Iterable<T> findAll(Predicate, Sort) : 조건에 맞는 모든 데이터와 정렬;

    - T findOne(Predicate) : 조건에 맞는 하나의 데이터 
   ```

### Repository 변경

  - Repository 인터페이스에는 QueryDslPredicateExecutor 인터페이스를 상속받도록 추가해 주어야 한다.

  ```Java
    public interface BoardRepository extends CrudRepository<Board, Long>, QuerydslPredicateExecutor<Board> {
    }
  ```

## 3.3.3 Predicate 생성 및 테스트

  - Predicate는 '단언하다, 확신하다'는 의미이다.

     - boolean으로 리턴되는 결과 데이터를 만들어야 한다.

     - 주로 BooleanBuilder를 이용해서 생성하는데 사용하는 방법은 테스트 코드를 생성해서 알아보아야 한다.

     ```Java
     // Predicate 테스트
	@Test
	public void testPredicate(){
		String type = "t";
		String keyword = "17";

		BooleanBuilder builder = new BooleanBuilder();

		QBoard board = QBoard.board;

		if(type.equals("t")){
			builder.and(board.title.like("%" + keyword + "%"));
		}

		// bno > 0
		builder.and(board.bno.gt(0L));

		PageRequest pageable = PageRequest.of(0, 10);

		Page<Board> result = boardRepository.findAll(builder, pageable);

		System.out.println("PAGE SIZE : " + result.getSize());
		System.out.println("TOTAL PAGES : " + result.getTotalPages());
		System.out.println("TOTAL COUNT : " + result.getTotalElements());
		System.out.println("NEXT : " + result.nextPageable());

		List<Board> list = result.getContent();

		list.forEach(b -> System.out.println(b));

	}
     ```

     - BooleanBuilder를 생성하여 동적 쿼리에 필요한 조건을 and() 등을 이용해 추가한다.

     - QBoard는 Board의 속성을 이용해서 다양한 SQL에 필요한 구문을 처리할 수 있는 기능이 추가된 형태이므로, like()나 get() 등을 이용해서 원하는 SQL을 구성하는데 도움을 준다.

     - 리턴 타입을 Page 타입으로 설정했기 때문에 데이터를 추출하는 SQL과 개수를 파악하는 SQL이 실행되고, 이때 필요한 조건들이 지정되게 된다.

     - Predicate는 필요한 곳에서 생성하는 방식을 이용하기도 하지만, 별도의 클래스 등으로 만들어서 사용할 수도 있다.

