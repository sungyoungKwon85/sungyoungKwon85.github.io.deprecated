# QueryDSL 

---

- 20210224
- 김영한님 강의 복습

\

\

\



---



#### Querydsl 설정

```groovy
plugins {
  id 'org.springframework.boot' version ‘2.2.2.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE' //querydsl 추가
  id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
  id 'java'
}
group = 'study'
version = '0.0.1-SNAPSHOT' 
sourceCompatibility = '1.8'
     configurations {
         compileOnly {
             extendsFrom annotationProcessor
         }
}

repositories {
    mavenCentral()
}
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa' implementation 'org.springframework.boot:spring-boot-starter-web'
  //querydsl 추가
  implementation 'com.querydsl:querydsl-jpa'
  compileOnly 'org.projectlombok:lombok'
  runtimeOnly 'com.h2database:h2'
  annotationProcessor 'org.projectlombok:lombok' 	
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
  	exclude group: 'org.junit.vintage', module: 'junit-vintage-engine' 
  }
}
test {
    useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
configurations {
	querydsl.extendsFrom compileClasspath 
}
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}
```

\

\

\



#### Querydsl 환경설정 검증

```java
// 검증용 엔티티 생성
@Entity
@Getter @Setter
public class Hello {
  @Id @GeneratedValue
  private Long id;
}
```

- 검증용 Q 타입 생성
  - Gradle -> Tasks -> build -> clean
  - Gradle -> Tasks -> other -> compileQuerydsl
  - (`./gradlew clean compileQuerydsl`)

- Q 타입 생성 확인
  - build -> generated -> querydsl
    - study.querydsl.entity.QHello.java 파일이 생성되어 있어야 함

- 참고: Q타입은 컴파일 시점에 자동 생성되므로 **버전관리(GIT)에 포함하지 않는 것**이 좋다. 

```java
@SpringBootTest
@Transactional
class QuerydslApplicationTests {
  
  @Autowired
  EntityManager em;
  
  @Test
  void contextLoads() {
    Hello hello = new Hello(); 
    em.persist(hello);
    JPAQueryFactory query = new JPAQueryFactory(em); 
    
    QHello qHello = QHello.hello; //Querydsl Q타입 동작 확인
    
    Hello result = query 
      .selectFrom(qHello)
      .fetchOne(); 		
    
    Assertions
      .assertThat(result)
      .isEqualTo(hello);
    
    //lombok 동작 확인 (hello.getId())
    Assertions
      .assertThat(result.getId())
      .isEqualTo(hello.getId()); 
  }
}
```

\

\

\



---

#### 스프링 부트 설정 - JPA, DB

```yaml
spring:
	datasource:
		url: jdbc:h2:tcp://localhost/~/querydsl username: sa
		password:
			driver-class-name: org.h2.Driver
jpa:
	hibernate:
		ddl-auto: create 
  properties:
		hibernate:
		# show_sql: true
		format_sql: true
logging.level: 
	org.hibernate.SQL: debug
# org.hibernate.type: trace
```

\

\

\



---

#### 예제 도메인 모델

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) @ToString(of = {"id", "username", "age"})
public class Member {
  
  @Id
  @GeneratedValue 
  @Column(name = "member_id") 
  private Long id;
	private String username;
  private int age;
  
	@ManyToOne(fetch = FetchType.LAZY) 
  @JoinColumn(name = "team_id") 
  private Team team;
  
  public Member(String username) {
    this(username, 0);
	}
  
  public Member(String username, int age) {
    this(username, age, null);
	}
  
	public Member(String username, int age, Team team) { 		
    this.username = username;
    this.age = age;
		if (team != null) {
      changeTeam(team);
    }
	}
	public void changeTeam(Team team) { 
    this.team = team; 
    team.getMembers().add(this);
  } 
} 


@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED) @ToString(of = {"id", "name"})
public class Team {

  @Id @GeneratedValue 
  @Column(name = "team_id") 
  private Long id;
  
	private String name;
  
  @OneToMany(mappedBy = "team")
  List<Member> members = new ArrayList<>();
  
	public Team(String name) { 
    this.name = name;
  } 
}
```

\

\

\



---



#### 기본 문법

- `Querydsl` vs JPQL

  ```java
  @PersistenceContext
  EntityManager em;
  
  JPAQueryFactory queryFactory;
  
  @BeforeEach
  public void before() {
    queryFactory = new JPAQueryFactory(em);
  	//...
  }
  
  @Test
  public void startJPQL() { 
    //member1을 찾아라.
    String qlString =
      "select m from Member m " +
      "where m.username = :username";
    
    Member findMember = em.createQuery(qlString, Member.class) 
      .setParameter("username", "member1") 
      .getSingleResult();
    
    assertThat(findMember.getUsername())
        .isEqualTo("member1"); 
  }
  
  @Test
  public void startQuerydsl() {
    //member1을 찾아라.
    QMember m = new QMember("m");
    
    Member findMember = queryFactory 
      .select(m)
      .from(m)
      .where(m.username.eq("member1"))//파라미터 바인딩 처리
      .fetchOne(); 
    
    assertThat(findMember.getUsername())
      .isEqualTo("member1");
  }
  ```

  - JPQL: 문자(실행 시점 오류), **Querydsl: 코드(컴파일 시점 오류)** 굳!!
  - `JPAQueryFactory`를 필드로 제공하면 동시성 문제는 어떻게 될까? 
    동시성 문제는 `JPAQueryFactory`를 생성할 때 제공하는 `EntityManager`(em)에 달려있다. 
    **스프링 프레임워크는 여러 쓰레드에서 동시에 같은 `EntityManager`에 접근해도, 트랜잭션 마다 별도의 영속성 컨텍스트를 제공**하기 때문에, 동시성 문제는 걱 정하지 않아도 된다.

\

\

\



---

#### 기본 Q-Type 활용

- Q클래스 인스턴스를 사용하는 2가지 방법

  ```java
  QMember qMember = new QMember("m"); //별칭 직접 지정 
  QMember qMember = QMember.member; //기본 인스턴스 사용
  ```

- **기본 인스턴스를 static import와 함께 사용** (권장)

  ```java
  import static study.querydsl.entity.QMember.*;
  
  @Test
  public void startQuerydsl3() {
    //member1을 찾아라.
    Member findMember = queryFactory
      .select(member)
      .from(member) 
      .where(member.username.eq("member1"))
      .fetchOne(); 
     
    assertThat(findMember.getUsername())
      .isEqualTo("member1");
  }
  ```

- 결국 JPQL로 변경될 건데 볼수가 없으면 아래 내용 추가

  ```
  spring.jpa.properties.hibernate.use_sql_comments: true
  ```

\

\

\

---

- 기본 검색 쿼리

```java

@Test
public void search() {
  Member findMember = queryFactory 
    .selectFrom(member)
    .where(member.username.eq("member1") 
           .and(member.age.eq(10)))
    .fetchOne(); 
  //  .and(),.or()를 메서드체인으로 연결할 수 있다.
  
  assertThat(findMember.getUsername())
    .isEqualTo("member1");
}

// AND 조건을 파라미터로 처리
// where() 에 파라미터로 검색조건을 추가하면 AND 조건이 추가됨
// 이경우 null 값은무시 -> 메서드추출을 활용해서 동적쿼리를 깔끔하게 만들수 있음
@Test
public void searchAndParam() {
  List<Member> result1 = 
    queryFactory 
    .selectFrom(member)
    .where(member.username.eq("member1"), 
           member.age.eq(10))
    .fetch(); 
  
  assertThat(result1.size()).isEqualTo(1);
}

```

\

\

\

- 결과 조회

  ```java
  //List
  List<Member> fetch = queryFactory 
    .selectFrom(member)
    .fetch();
  
  //단 건
  Member findMember1 = queryFactory
    .selectFrom(member) 
    .fetchOne();
  
  //처음 한 건 조회
  Member findMember2 = queryFactory
    .selectFrom(member) 
    .fetchFirst();
  
  //페이징에서 사용
  QueryResults<Member> results = queryFactory
    .selectFrom(member) 
    .fetchResults();
  
  //count 쿼리로 변경
  long count = queryFactory
    .selectFrom(member) 
    .fetchCount();
  ```

\

\

\

- 정렬

  ```java
  List<Member> result = queryFactory 
    .selectFrom(member)
    .where(member.age.eq(100))
    .orderBy(member.age.desc(), 
             member.username.asc().nullsLast()) 
    .fetch();
  ```

  - `desc()` , `asc()` : 일반 정렬
  - `nullsLast()` , `nullsFirst()` : null 데이터 순서 부여

\

\

\

- 페이징

  ```java
  // 조회 건수 제한
  @Test
  public void paging1() {
    List<Member> result = queryFactory 
      .selectFrom(member)
      .orderBy(member.username.desc()) 
      .offset(1) //0부터 시작(zero index) 
      .limit(2) //최대 2건 조회
      .fetch();
    
    assertThat(result.size()).isEqualTo(2); 
  }
  
  
  // 전체 조회 수가 필요하면?
  // 주의: count 쿼리가 실행되니 성능상 주의!
  @Test
  public void paging2() {
    QueryResults<Member> queryResults = queryFactory 
      .selectFrom(member)
      .orderBy(member.username.desc()) 
      .offset(1)
      .limit(2)
      .fetchResults();
    
    assertThat(queryResults.getTotal()).isEqualTo(4); 
    assertThat(queryResults.getLimit()).isEqualTo(2); 
    assertThat(queryResults.getOffset()).isEqualTo(1); 
    assertThat(queryResults.getResults().size())
      .isEqualTo(2);
  }
  ```

  - 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, **count 쿼리는 조인이 필요 없는 경우도 있다**. 그런데 이렇게 자동화된 count 쿼리는 원본 쿼리와 같이 **모두 조인을 해버리기 때문에 성능이 안나올 수 있다.** **count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.**

\

\

\

---

























