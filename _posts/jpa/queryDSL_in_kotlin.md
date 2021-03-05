# QueryDSL in Kotlin

#### build.gradle.kts

```kotlin
plugins {
  ...
  kotlin("plugin.jpa") version "1.4.21"
  kotlin("kapt") version "1.4.10"
}
sourceSets["main"].withConvention(org.jetbrains.kotlin.gradle.plugin.KotlinSourceSet::class) {
    kotlin.srcDir("$buildDir/generated/source/kapt/main")
}

dependencies {
  ...
  implementation("com.querydsl:querydsl-jpa:4.4.0")
  kapt("com.querydsl:querydsl-apt:4.4.0:jpa")
}
```

- kapt
  - Kotlin Annotation Processing
  - 코틀린에서 Annotation Processing을 지원하기 위해서는 kapt가 필요하다.

<br><br><br>

---

#### QueryDslConfiguration

```kotlin
@Configuration
class QueryDslConfiguration(
    @PersistenceContext
    val entityManager: EntityManager
) {
    @Bean
    fun jpaQueryFactory() = JPAQueryFactory(entityManager)
}
```

<br><br><br>

---

#### Usage

```kotlin
@Repository
class UserTermAgreeQueryRepository @Autowired constructor(
    private val queryFactory: JPAQueryFactory
) : QuerydslRepositorySupport(UserTermAgree::class.java) {

    fun existByUserIdAndTerInfoId(userId: Long, termId: Long): Boolean {
        return queryFactory
      		  .selectFrom(userTermAgree)
            .where(
                userTermAgree.user.userId.eq(userId)
                    .and(userTermAgree.term.termId.eq(termId))
            )
            .select(userTermAgree.id)
            .fetchFirst() != null
    }
}
```



<br><br><br>

---

#### Dependency management

```kotlin
plugins {
    id("org.springframework.boot") version "2.4.2"
  	...
}

dependencies {
  	...
    implementation("com.querydsl:querydsl-jpa:4.2.2")
    kapt("com.querydsl:querydsl-apt:4.2.2:jpa")
}
```

위와 같이 `SpringBoot 2.4.2` 및 `querydsl 4.2.2` 를 사용할 경우 `UnsupportedOperationException` 문제가 발생한다.   

```
java.lang.UnsupportedOperationException
	at java.base/java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
	at com.querydsl.jpa.JPQLSerializer.visitConstant(JPQLSerializer.java:327)
	at com.querydsl.core.support.SerializerBase.visit(SerializerBase.java:221)
```

Exception만 보고는 어떤 문제인지 쉽사리 알아차리기 어려운데, 이는 `Dependency version` 에 문제가 있어서 발생하는 것이다.   

[Spring Dependency versions](https://docs.spring.io/spring-boot/docs/2.4.2-SNAPSHOT/reference/html/appendix-dependency-versions.html#dependency-versions) 문서를 참고해보면  `SpringBoot 2.4.2` 가 지원하는 `querydsl` 의 버전은 `4.4.0` 임을 알 수 있다.   



