---
layout: post
title:  "python-django-database"
date:   2017-02-27 17:10:53 +0900
categories: python-django
---

참고 :  
http://tuwlab.com/26425  


<br><br><br>

### Database 연동하기  

Django에서는 **ORM 추상화 계층** 을 통해 실제 Table의 field 형식에 구애받지 않고 다양한 형식의 Field를 사용할 수 있다.

<br><br><br>

# 모델 정의  

앞서 JSONField를 설치해야 한다.  
`pip install jsonfield`  

<br>

myapp/models.py  
{% highlight ruby %}
from django.db import models
from jsonfield import JSONField

class Book(models.Model):
    isbn = models.BigIntegerField(primary_key=True)
    title = models.CharField(max_length=128)
    memo = JSONField(default={}, dump_kwargs={'ensure_ascii': False})

    def __str__(self):
        return self.title
{% endhighlight %}

모델 필드에 대한 상세한 정보는 아래 주소를 참고할 것  
https://docs.djangoproject.com/en/1.8/ref/models/fields/  


<br><br><br>

# Admin에 모듈 등록  

{% highlight ruby %}
myapp/admin.py

from myapp.models import Book
admin.site.register(Book)
{% endhighlight %}  

<br><br><br>

# Migration을 통한 반영  

`$ ./manage.py makemigrations myapp`  
myapp/migrations디렉토리 안에 Migration파일(xxxx_initial.py)이 생성된다.  

`$ ./manage.py migrate`  
DB에 Table이 정상적으로 생성된 것  


<br><br><br>

# View와 DB 연동  
{% highlight ruby %}
myapp/views.py

from myapp.models import Book

...

def InsertBook(request, isbn, title, memo):
    Book(isbn=isbn, title=title, memo={'content': memo}).save()
    return render(request, 'myapp/mypage.html',
                  { 'welcome_text': 'Insert: ' + title })
{% endhighlight %}


{% highlight ruby %}
sitebase/urls.py

urlpatterns = [
    ...
    url(r'^insert/(?P<isbn>.+);(?P<title>.+);(?P<memo>.*)',
        MyAppView.InsertBook),
]

**
{% endhighlight %}


`http://localhost:8000/insert/1234;공업수학1;Welcome_to_andromeda`  


<br><br><br>

# DB로부터 data 읽어와 View 만들기  

{% highlight ruby %}
myapp/views.py

def DisplayBook(request, isbn):
    result = Book.objects.filter(isbn=isbn)[0]
    bookInfo = "ISBN: {0}; TITLE: {1}; MEMO: {2}".format(result.isbn,
                                                        result.title,
                                                        result.memo['content'])
    return render(request, 'myapp/mypage.html',
                  { 'welcome_text': bookInfo })


sitebase/urls.py
urlpatterns = [
    ...
    url(r'^show/(?P<isbn>.+)', MyAppView.DisplayBook),
]              
{% endhighlight %}

`http://localhost:8000/show/1234`  


<br><br><br>
<br><br><br>

# 기존 테이블을 models.py 코드화 시키기  

`$ python manage.py inspectdb > ***.py`  











{% highlight ruby %}

{% endhighlight %}


.
