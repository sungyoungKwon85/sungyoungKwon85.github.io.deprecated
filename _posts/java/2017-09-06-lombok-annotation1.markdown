---
layout: post
title:  "Lombok Annotation"
date:   2017-09-06 13:10:53 +0900
categories: lombok
---

# Lombok's annotations  


팀에서 Lombok annotation 사용 정책에 있는 리스트
- @Cleanup
- @Getter/@Setter
- @ToString
- @NoArgsConstructor
- @Builder
- @SneakyThrows
- @Synchronized
- @Getter(laze=true)
- @Log
  
<br><br><br>    
      
{% highlight ruby %}
@AllArgsConstructor
public static class Order {
    private long cancelPrice;
    private long orderPrice;
    //
}
  
// 취소금액 5,000원, 주문금액 10,000원
Order order = new Order(5000L, 10000L);
{% endhighlight %}       
아래의 예제와 같이 @Builder 붙여서 사용  
@Builder는 가급적이면 클래스에 붙이지 않고, 생성자 또는 static 객체 메서드에 사용         
{% highlight ruby %}
public static class Order {
    private long cancelPrice;
    private long orderPrice;
  
    @Builder
    private Order(long cancelPrice, long orderPrice) {
        this.cancelPrice = cancelPrice;
        this.orderPrice = orderPrice;
    }
}
 
// 필드 순서를 변경해도 문제 없음.
Order order = Order.builder().cancelPrice(5000L).orderPrice(10000L).build();
System.out.println(order);
{% endhighlight %}

<br><br><br>

Immutable 클래스에서만 @EqualsAndHashCode를 사용    
(@EqualsAndHashCode는 equals(Object) 메소드와 hashCode() 메소드를 오버라이드한다.)    

<br><br><br>

@Data는 @EqualsAndHashCode, @RequiredArgsConstructor를 포함하고 있기 때문에 사용 금지  
@Data 대신 아래와 같이 사용  

{% highlight ruby %}
@Getter
@Setter
@ToString
public class Order {
...
    // 생성자와 필요한 경우에만 equals, hashCode 직접 작성
}
{% endhighlight %}

<br><br><br>

@Value도 @AllArgConstructor를 포함하기 때문에 사용 금지  
@Value 대신 아래와 같이 사용  

{% highlight ruby %}
@Getter
@ToString
public class Order {
   // private final 로 여러 필드 생성
   // 생성자와 필요한 경우에만 equals, hashCode 직접 작성
}
{% endhighlight %}

<br><br><br>

@Val은 문법파괴적이고 IDE 지원이 불명확하여 사용금지  

<br><br><br>

@NonNull 사용 금지  
대신 아래와 같이 guava 등의 유틸리티를 활용해서 예외를 발생시킴  

{% highlight ruby %}
// null일 경우 "userName must not be null." 메시지로 예외 발생
Preconditions.checkNotNull(userName, "userName must not be null.")
{% endhighlight %}

<br><br><br>

@SneakyThrows  
메소드 선언부에 사용되는 throws 키워드 대신 사용하는 어노테이션으로 예외 클래2017-09-06-lombok-annotation.markdown스를 파라미터로 입력받는다.  

<br><br><br>

@Cleanup  
try-with-resource 구문과 비슷한 효과를 가진다. 구문이 종료될 때 AutoCloseable 인터페이스의 close()가 호출되는 try-with-resource와 달리 Scope가 종료될 때 close()가 호출된다.  

<br><br><br>

@Synchronized  
메소드에 사용되는 어노테이션으로 기본적으로 지원되는 synchronized 키워드보다 더 세세한 설정이 가능한 어노테이션이다.   
synchronized 키워드는 static 혹은 instance 단위로 락을 걸지만 @Synchronized 어노테이션은 파라미터로 입력받는 Object 단위로 락을 건다.   
파라미터로 아무 것도 입력하지 않으면 어노테이션이 사용된 메소드 단위로 락을 건다.  



<br><br><br>


참조 : http://partnerjun.tistory.com/53









{% highlight ruby %}
{% endhighlight %}
<br><br><br>