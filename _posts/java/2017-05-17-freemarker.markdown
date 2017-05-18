---
layout: post
title:  "free-marker"
date:   2017-05-17 11:10:53 +0900
categories: spring
---

# [Freemarker 안전하게 사용하기]  
Free Marker는 Java 진영에서 대중적인 View Template Engine이다.  

JSTL, Velocity의 경우는 과거  
Thymelear, Handlbar.java는 아직 널리 퍼지진 않았다.(지금은 많이 쓸 것 같은데..?)  

보통 View를 구성할 때는 공통 사항에 대해서는 따로 관리를 한다.  
아래와 같은 느낌이랄까  
GNB,  
Body[],  
Footer,  

<br>

작성하는 페이지에 GNB와 Footer를 적용하기 위해 html코드에 #include를 사용해서 넣는게 보통일 것 같지만, 이 방식에는 문제점이 있다.  
GNB든 Footer든 다른 View파일이든 한 파일만 문제가 발생해도 전체 페이지가 나오지 않는단다.  

따라서 해당 링크에서는 자바 코드에서 미리 처리하자는 팁을 제시하고 있다.  

try/catch문을 통해 template 진행 중 실패하면 빈 문자열로 대체하여 내려줄 수 있도록 한다.  



<br><br><br>



# [기본 문법]  
**if && ||**  
{% highlight ruby %}
<#if obj?? && obj?size == 0>
...
<#/if>
{% endhighlight %}


<br><br><br>


# [Spring + Freemarker]  

- Spring MVC 4.3.2.RELEASE
- Freemarker 2.3.25-incubating

{% highlight ruby %}
<dependency>
  <groupid>org.freemarker</groupid>
  <artifactid>freemarker</artifactid>
  <version>2.3.25-incubating</version>
</dependency>
{% endhighlight %}

링크에서는 freemarker 설정을 위해 xml설정 방식을 보여주고 있다.  
자세한건 링크를 참고 하자.  


<br><br><br>


[Freemarker 안전하게 사용하기]: http://jojoldu.tistory.com/30
[기본 문법]: http://cpdev.tistory.com/44
[Spring + Freemarker]: http://andang72.blogspot.kr/2016/09/spring-part-2-freemarker.html
<br><br><br>

{% highlight ruby %}
{% endhighlight %}
