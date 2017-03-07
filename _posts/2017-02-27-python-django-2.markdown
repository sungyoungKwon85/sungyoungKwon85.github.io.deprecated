---
layout: post
title:  "python-django-view"
date:   2017-02-27 17:10:53 +0900
categories: python-django
---

참고 :  
http://tuwlab.com/26314  

<br><br><br>

### URL Config, Template, View  

MVC가 아닌 MTV방식, 즉 View가 Controller+View 역할을 한다고 볼 수 있다.  

<br><br><br>

# URL Router  
URL Router는 요청이 들어온 **주소** 를 검사하여 적합한 **View메소드** 와 연결시켜주는 동작을 수행한다.

주소 체계와 관련된 귀찮은 작업을 파일 하나에서 체계적으로 정의하고 관리할 수 있다.  


<br><br><br>


# Template 파일 작성  

Template는 HTML응답을 만들어내기 위한 틀.  
렌더링할 때 필요한 파라미터를 치환할 Maker가 존재한다.  

`$ mkdir -p myapp/templates/myapp`  

myapp/templates/myapp/mypage.html  
{% highlight ruby %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Django Tutorial</title>
</head>
<body>
{{ welcome_text }}
</body>
</html>
{% endhighlight %}

문법 참조 : https://docs.djangoproject.com/en/1.8/topics/templates/  


<br><br><br>

# View 메서드 정의  

myapp/views.py  
{% highlight ruby %}
from django.shortcuts import render

def DisplayMyPage(request):
    return render(request, 'myapp/mypage.html', { 'welcome_text': 'Hello World!' })
{% endhighlight %}

<br><br><br>

# URL Router에 View 메서드 등록  

urls.py  
{% highlight ruby %}
...
from myapp import views as MyAppView
...
urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^mypage/', MyAppView.DisplayMyPage),
]
{% endhighlight %}

`$ ./manage.py runserver`  

`http://localhost:8000/mypage`   


<br><br><br>

# URL Parameter를 View로 전달  

views.py  
{% highlight ruby %}
...
def DisplayMyPageWithParameter(request, my_parameter):
    welcomeText = my_parameter
    return render(request, 'myapp/mypage.html', { 'welcome_text': welcomeText })
{% endhighlight %}

<br>

urls.py  
{% highlight ruby %}
urlpatterns = [
    ...
    url(r'^mypage_param/(?P<my_parameter>.+)', MyAppView.DisplayMyPageWithParameter),
]

{% endhighlight %}

`http://localhost:8000/mypage_param/ThisIsParameter!`   

<br><br><br>

# HTTP Method, Cookie  

클라이언트에서 Form을 통해 전송한 파라미터나 Browser Cookie 정보는 모두 View메서드에 파라미터로 전달되는 request객체에 포함되어 있다.

{% highlight ruby %}
uid = request.GET.get('uid')  
uid = request.POST.get('uid')
uid = request.REQUEST['uid']
lang = request.COOKIES.get('lang')
{% endhighlight %}

<br>

응답(response)는 HttpResponse객체의 set_cookie()를 활용한다.  

myapp/views.py  
{% highlight ruby %}
def DisplayMyPage(request):
    ...
    response = render(request, 'myapp/mypage.html', context)
    response.set_cookie('lang', 'ko_kr')
    return response
{% endhighlight %}



















{% highlight ruby %}

{% endhighlight %}


.
