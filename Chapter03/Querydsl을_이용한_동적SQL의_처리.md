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
 