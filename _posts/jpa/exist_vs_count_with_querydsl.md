# exist query

`jpa` 에서 제공하는  `existsById` 메서드를 실행해보자.  

```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
  ...
    boolean existsById(ID id);
}
```

<br><br>

아래와 같이 `count` 쿼리가 날라간다.   

```
select
	count(*) as col_0_0_ 
from
	user user0_ 
where
	user0_.user_id=?
```

[어떤 블로그](https://jojoldu.tistory.com/516) 에서 자세히 설명이 되어 있는데, `count` 쿼리는 데이터가 많을수록 성능상 효율적이지 못하다.  따라서 `limit 1` 을 이용하여 개선하는 것이 좋다.  

<br><br>

나는 아래처럼 `querydsl` 을 이용해서 `exist` 를 구현했다.    

`fecthFirst()` 가 `limit` 을 사용하도록 바뀐다.   

```kotlin
fun existByUserIdAndTermId(userId: Long, termId: Long): Boolean {
        return queryFactory
  					.selectFrom(userTermAgree)
            .where(
                userTermAgree.user.userId.eq(userId)
                    .and(userTermAgree.term.termId.eq(termId))
            )
            .select(userTermAgree.id)
            .fetchFirst() != null
    }
```

```
select
	usert0_.id as col_0_0_ 
from
	user_term_agree usert0_ 
where
	usert0_.user_id=? 
	and usert0_.term_id=? limit ?
```

