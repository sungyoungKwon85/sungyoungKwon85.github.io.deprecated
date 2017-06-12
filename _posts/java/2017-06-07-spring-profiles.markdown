---
layout: post
title:  "spring-profiles"
date:   2017-06-07 10:10:53 +0900
categories: spring
---

## [Profiles history]   
## [Spring profiles]  


[Profiles history]: https://www.slideshare.net/sbcoba/2015-47137155
[Spring profiles]: http://jdm.kr/blog/81



{% highlight ruby %}
# application-context.xml
<context:property-placeholder location="classpath:/config/config-#{systemProperties['myenv']}.properties" />  

# catalina.properties
myenv=local



{% endhighlight %}
