 

# JPA 강의 복습하면서 메모했음



---

## 실전! 스프링 데이터 JPA

---



- IntelliJ > Preferences > Build Tools > Gradle > 
  - `Build and run using`: `IntelliJ IDEA` 로 바꾸자
  - `Run tests using`: `IntelliJ IDEA` 로 바꾸자 
  - gradle을 통해서 띄우다 보니 너무 느리다. 



- `starter-data-jpa`를 등록하면 `starter-aop`, `starter-jdbc`, `hibernate-core`등 필요한 디펜던시를 다 땡겨온다. 
- `starter-jdbc` 들어가보면 `dbcp`로 `Hikari`가 되어 있음을 볼 수 있다.기존의 `톰캣 dbcp`랑 설정하는게 좀 다름. 성능이 좋다. 
- `boot 2.2`로 가면서 `junit5`가 디폴트가 됨. (`mockito-junit-jupiter`)



- https://spring.io > projects > springboot > learn >  documentation current version, reference Doc > [Dependency Versions](https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/appendix-dependency-versions.html#dependency-versions)
  - 해당 스프링부트 버전의 디펜던시 버전들을 확인할 수 있다. 
  - 우리가 디펜던시 명시할 때 버전을 생략하면 이 페이지의 버전대로 세팅되는 것. 



- `jpa.hibernate.ddl-auto` 는 조심해서 써야함. 

- `jpa.properties.hibernate.show_sql` 은 주석처리하고 
  `logging.level.org.hibernate.SQL: debug` 로 했음
  콘솔에 남기기보다는 로그로 남기는 게 좋으니까.
- `logging.level.org.hibernate.type: trace` 이 옵션은 파라미터까지 볼 수 있게 하는 거. 

- `@NoArgsConstructor AccessLevel.PROTECTED`: 기본 생성자 막고 싶은데, JPA 스팩상 PROTECTED로 열어두어야 함. (proxy 등을 이용해서 동적으로 클래스들을 생성하려고 하는데 private로 해버리면 생성을 할 수가 없다. )



- 쿼리 파라미터를 보고 싶은데 `logging.level.org.hibernate.type: trace` 얘도 좀 답답함. 
  외부 라이브러리를 가지고 개발 때 사용하면 좋다. 
  `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'`


---

#### Member & Team

- Member >---o Team (N:1 관계)

```java
@Entity
@Getter 
@NoArgsConstructor(access = AccessLevel.PROTECTED) 
@ToString(of = {"id", "username", "age"})
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
}

@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED) 
@ToString(of = {"id", "name"})
public class Team {
    @Id 
    @GeneratedValue 
    @Column(name = "team_id") 
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<>()
}
      
```

- 가급적 `ToString`에는 연관관계 데이터는 넣지 말자
- `ManyToOne` 관계는 `fetch = FetchType.LAZY` 로 셋팅하는게 실무에 유리하다. 

---



- JPA를 사용하면 update는 변경감지(`dirty checking`) 를 이용하는 경우가 많음



```java
@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {}	
```

- 스프링부트를 사용하면 `@SpringBootApplication`에 의해 같은 위치로 지정되므로 위 코드가 필요없다. 
  위치가 달라진다면 위 코드를 명시해줘야 할 것.



```java
 public interface MemberRepository extends JpaRepository<Member, Long> {}
```

- 구현체는 어디있냐?
  Spring Data JPA가 proxy, 리플렉션으로 클래스를 만들고 인젝션 해줌ㅋ (`class com.sun.proxy.$ProxyXXX`)

---

#### 인터페이스 분석

- -- `스프링 데이터` --
  `Repository` 

  ^

  `CrudRepository` 

  ^

  `PagingAndSortingRepository`

  -- `스프링 데이터 JPA` --

  ^
  `JpaRepository`

---

#### [쿼리 메소드 기능](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
     List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

- Spring Data JPA가 마술을 부렸다
- Entity field명이 바뀌면 메서드 이름도 꼭 변경해야 한다(어플리케이션 구동 시점 오류 날 것임 <- 장점이쥬?)
- 실무에서는 메서드 이름이 점점 길어지면서 매우 지저분해져서 `@Query`를 사용하게 됨



#### [Using JPA Named Queries](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.named-queries)

```java
@Entity
@NamedQuery(name = "User.findByEmailAddress",
  query = "select u from User u where u.emailAddress = ?1")
public class User {
}

public interface UserRepository extends JpaRepository<User, Long> {
  List<User> findByLastname(String lastname);
  User findByEmailAddress(String emailAddress);
}
```

- 실무에서 잘 안씀. `@Query`를 사용해서 repository에 직접 정의하는게 나음



#### [`@Query`](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.named-queries)

```java
public interface UserRepository extends JpaRepository<User, Long> {
  @Query("select u from User u where u.emailAddress = ?1")
  User findByEmailAddress(String emailAddress);
}
```

- `JPQL` 문법으로 작성하게 된다. 
- 애플리케이션 실행 시점에 문법 오류를 발견할 수 있음(good!)
- native query로 작성하고자 하면 아래와 같이 `nativeQuery`를 이용하면 된다. 

```java
public interface UserRepository extends JpaRepository<User, Long> {
  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
}
```

- **동적 쿼리**는 `QueryDSL` 써야 한다



---

#### @Query, 값, DTO 조회하기

- 단순한 값 조회

```java
 @Query("select m.username from Member m")
 List<String> findUsernameList();
// JPA 값 타입( @Embedded )도 이 방식으로 조회할 수 있다.
```



- **DTO**로 직접 조회
  - `new` 명령어를 사용해야 한다
  - 생성자가 동일해야 한다

```java
@Query("select new study.datajpa.repository.MemberDto(m.id, m.username, t.name)" + 
       "from m join m.team t"
List<MemberDto> findMemberDto();
```

- 요런것도 `QueryDSL` 쓰면 편해진다. 



#### 파라미터 바인딩

```java
select m from Member m where m.username = ?0 //위치 기반 
select m from Member m where m.username = :name //이름 기반
```

```java
import org.springframework.data.repository.query.Param
public interface MemberRepository extends JpaRepository<Member, Long> {
@Query("select m from Member m where m.username = :name")
    Member findMembers(@Param("name") String username);
}
```

- 가독성/유지보수를 위해 **이름 기반 바인딩을 권장**한다.



- **컬렉션 파라미터** 바인딩

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```



#### [반환 타입](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types)

- 스프링 데이터 JPA는 **유연한 반환 타입** 지원

  ```java
  List<Member> findByUsername(String name); //컬렉션
  Member findByUsername(String name); //단건
  Optional<Member> findByUsername(String name); //단건 Optional
  ```

  - 컬렉션
    - 결과 없음: 빈 컬렉션 반환 
  - 단건 조회
    - 결과 없음: `null` 반환
    - 결과가 2건 이상: `javax.persistence.NonUniqueResultException` 예외 발생





#### 순수 JPA 페이징과 정렬

- JPA

```java
public List<Member> findByPage(int age, int offset, int limit) {
    return em.createQuery("select m from Member m where m.age = :age order by
        m.username desc") 
        .setParameter("age", age) 
        .setFirstResult(offset)          
        .setMaxResults(limit) 
        .getResultList();
}

public long totalCount(int age) {
    return em.createQuery("select count(m) from Member m where m.age = :age",
        Long.class)
        .setParameter("age", age) 
        .getSingleResult();
}
```



- Spring Data JPA

```java
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 
Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함
List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함
List<Member> findByUsername(String name, Sort sort);
```

- `Page`는 1부터 시작이 아니라 0부터 시작

- `Page`로 반환하면 **totalCount**, **totalElement**가 같이 나옴

  - **totalCount는 join 등의 상황에 맞물려 무거워질 것, 아래처럼 분리하면 좋다**

  - ```java
    @Query(value = "select m from Member m",
        countQuery = "select count(m.username) from Member m")
    Page<Member> findMemberAllCountBy(Pageable pageable);
    ```

- `Slice`는 앞뒤가 있는지 정도만 봄

- `Sort`도 복잡해지면 성능이 안나올 수가 있다. 이런 경우 우의 totalCount 처럼 분리해서 사용하면 될 것

- Page<Member> 같은건 바로 api 결과로 하면 안된다. DTO로 변환해야 한다. (`map`)

  ```java
  Page<Member> page = memberRepository.findByAge(10, pageRequest);
  Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
  ```

  





---

#### 벌크

- JPA

```java
public int bulkAgePlus(int age) { 
  int resultCount = em.createQuery(
    "update Member m set m.age = m.age + 1" + "where m.age >= :age")
    .setParameter("age", age)
    .executeUpdate(); 
  return resultCount;
}
```



- Spring Data JPA
  - 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화 해야 한다. `@Modifying(clearAutomatically = true)` (이 옵션의 기본값은 false )
  - 이 옵션 없이 회원을 findById로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아서 문제가 될 수 있다. 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자. 
  - **(벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 엔티티 상태가 달라질 수 있다.)**
    - 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크 연산을 먼저 실행한다.
    - 부득이하게 영속성 컨텍스트에 엔티티가 있으면 벌크 연산 직후 영속성 컨텍스트를 초기화 한다.

```java
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age") int bulkAgePlus(@Param("age") int age);
```

---

#### `@EntityGraph`

- member team은 **지연로딩 관계**이다. 따라서 다음과 같이 team의 데이터를 조회할 때 마다 쿼리가 실

  행된다. **(N+1 문제 발생)**

- **연관된 엔티티를 한번에 조회하려면** `페치 조인`**이 필요하다.**

```java
@Query("select m from Member m left join fetch m.team") List<Member> findMemberFetchJoin();
```





- **엔티티 그래프 기능을 사용 하면 JPQL 없이 페치 조인을 사용할 수 있다. (JPQL + 엔티티 그래프도 가능)**
  - 사실상 `페치 조인(FETCH JOIN)`의 간편 버전
  - **LEFT OUTER JOIN** 사용

```java
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"}) 
List<Member> findAll();

//JPQL + 엔티티 그래프 
@EntityGraph(attributePaths = {"team"}) 
@Query("select m from Member m") 
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(@Param("username") String username)
```



- **NamedEntityGraph** **사용 방법**

```java
@NamedEntityGraph(name = "Member.all", attributeNodes = @NamedAttributeNode("team"))
@Entity
public class Member {}

// 

@EntityGraph("Member.all") 
@Query("select m from Member m")
List<Member> findMemberEntityGraph();
```

---



#### JPA Hint & Lock

- JPA Hint: JPA 쿼리 힌트(SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트)

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);


@QueryHints(value = { @QueryHint(name = "org.hibernate.readOnly", value = "true")},
Page<Member> findByUsername(String name, Pagable pageable);
```

- dirty checking 기능을 위해서는 아무리 최적화를 했다해도 비용이 발생한다. 
  100% 조회용으로 쓰고 싶다면 흰트를 이용하자. (진~~짜 트래픽 넘치는 쿼리에만 사용해도 무방할 정도임)





- Lock
  - JPA가 제공하는 락은 **JPA 책 16.1 트랜잭션과 락 절**을 참고

```java
@Lock(LockModeType.PESSIMISTIC_WRITE) 
List<Member> findByUsername(String name);
```





---

#### 확장 기능

- 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면?
- 실무에서는 주로 `QueryDSL`이나 `SpringJdbcTemplate`을 함께 사용할 때 사용자 정의 리포지토 리 기능 **자주 사용**

```java
  public interface MemberRepositoryCustom {
      List<Member> findMemberCustom();
  }
```

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
      private final EntityManager em;
      @Override
      public List<Member> findMemberCustom() {
          return em.createQuery("select m from Member m") 
              .getResultList();
      } 
}

```

```java
public interface MemberRepository
           extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```

- 규칙: 리포지토리 인터페이스 이름 + Impl' 
- 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록



- Impl 대신 다른 이름으로 변경하고 싶으면? 

  ```java
  @EnableJpaRepositories(basePackages = "study.datajpa.repository",
                             repositoryImplementationPostfix = "Impl")
  ```

  - xml도 가능

  ```xml
   <repositories base-package="study.datajpa.repository"
    repository-impl-postfix="Impl" />
  ```

  

#### 사용자 정의 리포지토리 구현 최신 방식

- 스프링 데이터 2.x 부터는 사용자 정의 구현 클래스에 리포지토리 인터페이스 이름 + Impl 을 적용하는 대

  신에 **사용자 정의 인터페이스 명 + Impl 방식**도 지원한다.

  기존 방식보다 이 방식이 사용자 정의 인터페이스 이름과 구현 클래스 이름이 비슷하므로 더 직관적이다. 

  **새롭게 변경된 이 방식을 사용하는 것을 더 권장한다.**

```java
@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom {
        private final EntityManager em;
        @Override
        public List<Member> findMemberCustom() {
            return em.createQuery("select m from Member m") 
              .getResultList();
        } 
}
```





#### Auditing

- 엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶으면? 
  - 등록일
  - 수정일 
  - 등록자 
  - 수정자



- 순수 JPA 사용

```java
package study.datajpa.entity;
@MappedSuperclass
@Getter
public class JpaBaseEntity {
  
  @Column(updatable = false)
  private LocalDateTime createdDate;
  private LocalDateTime updatedDate;
  
  @PrePersist
  public void prePersist() {
    LocalDateTime now = LocalDateTime.now(); createdDate = now;
    updatedDate = now;
  }
  
  @PreUpdate
  public void preUpdate() {
    updatedDate = LocalDateTime.now(); 
  }
}

public class Member extends JpaBaseEntity {}
```



- 스프링 데이터 JPA 사용

```java
package jpabook.jpashop.domain;

@EntityListeners(AuditingEntityListener.class) 
@MappedSuperclass
public class BaseEntity {
  
  @CreatedDate
  @Column(updatable = false)
  private LocalDateTime createdDate;
  
  @LastModifiedDate
  private LocalDateTime lastModifiedDate;
  
  @CreatedBy
  @Column(updatable = false)
  private String createdBy;
  
  @LastModifiedBy
  private String lastModifiedBy;
}

```



- 등록자, 수정자를 처리해주는 `AuditorAware` 스프링 빈 등록
  - 실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받음

```java
@Bean
public AuditorAware<String> auditorProvider() {
	return () -> Optional.of(UUID.randomUUID().toString()); 
}
```





- 참고: 실무에서 대부분의 엔티티는 등록시간, 수정시간이 필요하지만, 등록자, 수정자는 없을 수도 있다. 그

  래서 다음과 같이 Base 타입을 분리하고, 원하는 타입을 선택해서 상속한다.

  ```java
  public class BaseTimeEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
  }
  
  public class BaseEntity extends BaseTimeEntity {
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    private String lastModifiedBy;
  }
  ```

  - 참고로 저장시점에 저장데이터만 입력하고 싶으면 `@EnableJpaAuditing(modifyOnCreate = false)` 옵션을 사용하면 된다.





---



#### Web 확장 - 도메인 클래스 컨버터

- 도메인 클래스 컨버터 사용 전

  ```java
  @RestController
  @RequiredArgsConstructor
  public class MemberController {
    
    private final MemberRepository memberRepository;
    
    @GetMapping("/members/{id}")
    public String findMember(@PathVariable("id") Long id) {
      Member member = memberRepository.findById(id).get();
      return member.getUsername(); }
  }
  ```

- **도메인 클래스 컨버터 사용 후**

  ```java
  @RestController
  @RequiredArgsConstructor
  public class MemberController {
    
    private final MemberRepository memberRepository;
    
    @GetMapping("/members/{id}")
    public String findMember(@PathVariable("id") Member member) {
      return member.getUsername(); }
  }
  ```

- HTTP 요청은 회원 id를 받지만 도메인 클래스 **컨버터가 중간에 동작**해서 회원 엔티티 객체를 반환

- 간단한 경우에만 사용하기 좋음
- **조회용으로만 사용해야함**. 컨버터도 repository를 이용하는데 트랜잭션이 없음. 
- 그닦 잘 안쓰임 ㅋ





---

#### Web 확장 - 페이징과 정렬

```java
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
  Page<Member> page = memberRepository.findAll(pageable);
  return page;
}
```

- 예) /members?**page**=0&**size**=3&**sort**=id,desc&**sort**=username,desc



- 글로벌 설정: 스프링 부트

```
spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/ spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
```

- 개별 설정

  ```java
  @RequestMapping(value = "/members_page", method = RequestMethod.GET) public String list(@PageableDefault(size = 12, sort = “username”,
      direction = Sort.Direction.DESC) Pageable pageable) {
    ...
  }
  ```



- **접두사**
  - 페이징 정보가 둘 이상이면 접두사로 구분
  - 예제: /members?member_page=0&order_page=1

```java
public String list(
	@Qualifier("member") Pageable memberPageable, 
  @Qualifier("order") Pageable orderPageable, 
  ...
```



- Page 내용을 **DTO로 변환하기**

```java
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
  return memberRepository.findAll(pageable).map(MemberDto::new); 
}
```





- Page를 1부터 시작하기?
  - Pageable, Page를 파리미터와 응답 값으로 사용히자 않고, **직접 클래스를 만들어서 처리**
  - `spring.data.web.pageable.one-indexed-parameters`는 한계가 있음 





---



- `@Transactional(readOnly = true)`
  - 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음
    자세한 내용은 JPA 책 15.4.2 읽기 전용 쿼리의 성능 최적화 참고



- **매우 중요**!!! 
  - **save() 메서드**
    - 새로운 엔티티면 저장( `persist` ) 
    - 새로운 엔티티가 아니면 병합( `merge` )

- 실무 하다보면 문제가 생기는 부분.

  - id를 `@GeneratedValue`를 무슨 이유에서인지 안했다고 하자. 

    ```java
    @Entity
    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public class Item {
      @Id
      private Long id;
    }
    ```

    ```java
    @Test
    public void save() {
      Item item = new Item("A"); 
      itemRepository.save(item);
    }
    ```

  - `save()` 메서드를 디버깅 해보자

    ```java
    @Transactional
    @Override
    public <S extends T> S save(S entity) {
      if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
      } else {
        return em.merge(entity)
      }
    }
    ```

  - **isNew에서 false가 되어 버림 --> `merge()`가 동작해버림**
    **즉, select해서 있는지 보고 없어서 insert를 함. 비효율적.**

  - **@Generated**를 안쓸거라면 Persistable<T> 인터페이스를 사용하자.  
    그리고 `CreatedDate`를 가지고 `isNew` 여부를 판단하면 좋다. 

    ```java
    package org.springframework.data.domain;
    public interface Persistable<ID> {
      ID getId();
      boolean isNew();
    }
    ```

    ```java
    @Entity 
    @EntityListeners(AuditingEntityListener.class) @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public class Item implements Persistable<String> {
      
      @Id
      private String id;
      
      @CreatedDate
      private LocalDateTime createdDate;
      
      public Item(String id) { 
        this.id = id;
      }
      
      @Override
      public String getId() {
      	return id; 
      }
      
      @Override
      public boolean isNew() {
      	return createdDate == null;
      }
    }
    ```

    

  - `merge()`를 하는 상황은 거의 `detached` 되는 상황인데 **실무**에서는 `dirty checking`이 대부분임. 

---

- Specifications: 
  - **실무에서는 JPA Criteria를 거의 안쓴다! 대신에 QueryDSL을 사용하자.**
- Query By Example: 
  - **실무에서 사용하기에는 매칭 조건이 너무 단순하고 LEFT 조인이 안됨**
  - **실무에서는 QueryDSL을 사용하자**



#### Projections

- 회원의 이름만 딱 조회하고 싶어!

```java
public interface UsernameOnly {
  String getUsername();
}

public interface MemberRepository ... {
	List<UsernameOnly> findProjectionsByUsername(String username);
}


...
@Test
public void projections() throws Exception {
  	//when
    List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1");
    //then
    Assertions.assertThat(result.size()).isEqualTo(1);
}
```

- 메서드 이름은 자유, 반환 타입으로 인지



- 인터페이스 기반 Open Proejctions

  - 다음과 같이 스프링의 SpEL 문법도 지원

  ```java
  public interface UsernameOnly {
  	@Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
     String getUsername();
  }
  ```

- 클래스 기반 Projection

  - 다음과 같이 인터페이스가 아닌 구체적인 DTO 형식도 가능

  ```java
  public class UsernameOnlyDto {
    
    private final String username;
  	public UsernameOnlyDto(String username) { 
      this.username = username;
  	}
        
    public String getUsername() {
      return username;
    } 
  }
  ```

- 동적 Projections

  ```java
   <T> List<T> findProjectionsByUsername(String username, Class<T> type);
  ```



- 실무의 복잡한 쿼리를 해결하기에는 한계가 있다.
- 실무에서는 단순할 때만 사용하고 조금만 복잡해지면 QueryDSL을 사용하자



---

#### 네이티브 쿼리

- 가급적 네이티브 쿼리는 사용하지 않는게 좋음, 정말 어쩔 수 없을 때 사용

  - JPQL처럼 애플리케이션 로딩 시점에 문법 확인 불가
  - 동적 쿼리 불가
  - Sort 파라미터를 통한 정렬이 정상 동작하지 않을 수 있음(믿지 말고 직접 처리)

- 최근에 나온 궁극의 방법 `스프링 데이터 Projections` 활용

  







