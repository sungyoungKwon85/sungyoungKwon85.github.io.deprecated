---
layout: post
title:  "external-api-call-not-working"
date:   2017-04-26 11:10:53 +0900
categories: linux
---

# external-api-call-not-working  


{% highlight ruby %}

$ nslookup
> weather.~~.com
...
Address: 52.78.~
> exit
$ ssh aaa@weather.~.com -p443

$ netstat -ntl | grep 52.78
{% endhighlight %}

확인해보면 호출 status를 알 수 있다.  
SYN_SENT라면 response가 없다는 것.  

A 서버  
B 서버  

A 서버에서 B서버로 call  
B 서버에서 response를 내려줄 때,  

B 서버에서는 A서버의 ip를 등록해두고 방화벽을 open해둔다.  
다만 A서버에서 B서버로 ip등록을 위해 AWS의 L4 domain을 알려줬고,  
B 서버는 그 domain을 통해 ip를 등록했다.  

하지만, AWS는 종종 ip가 변경이 이루어져 통신이 원할치 못했다.  
domain base로 등록되었더라면 좋았을 텐데..  

AWS의 EC2는 종종 ip가 변경된다는 것을 알아두자!  







<br><br><br>

{% highlight ruby %}
{% endhighlight %}
