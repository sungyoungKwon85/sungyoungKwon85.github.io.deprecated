---
layout: post
title:  "python-django"
date:   2017-02-25 17:10:53 +0900
categories: python-django
---

참고 :  
https://tutorial.djangogirls.org/ko/django/  

<br><br><br>

# python3 설치 for OS X    
https://www.python.org/downloads/release/python-343/  

<br><br><br>

# 버전 확인  
`$ python3 -- version`  

<br><br><br>

# Django란?  
파이썬으로 만들어진 무료 오픈소스 웹 어플리케이션 프레임워크(web application framework).  
웹 서버에 요청이 오면 장고로 전달된다.  
장고의 **urlresolver** 가 어떤 요청이 들어왔는지 확인한다.  
일치하는 게 있으면 장고는 그 요청을 관련된 함수(view)에 넘겨준다.  
대부분의 일들은 **view** 함수에서 모두 처리한다.
view에서 result를 생성하면 장고가 web browser에 보내주는 역할을 한다.  

<br><br><br>

# 가상환경 사용하기  
{% highlight ruby %}
$ mkdir djangogirls
$ cd djangogirls
$ python3 -m venv myvenv
{% endhighlight %}
- myvenv 는 가상환경의 이름  
- 소문자, 공백없어야 한다  

`~/djangogirls$ source myvenv/bin/activate`  

<br><br><br>

# Django 설치  

`$ pip install django==1.8`  

<br><br><br>
<br><br><br>

## 프로젝트 시작하기  

`$ django-admin startproject mysite .`  

디렉토리 구조는 아래와 같다.  
{% highlight ruby %}
djangogirls
├───manage.py
└───mysite
        settings.py
        urls.py
        wsgi.py
        __init__.py
{% endhighlight %}

<br><br><br>

# 설정 변경  
settings.py  
{% highlight ruby %}
TIME_ZONE = 'Asia/Seoul'  
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
{% endhighlight %}

<br><br><br>

# 데이터베이스 설정하기  
sqlite3는 settings.py에 기본 설치가 되어 있다.  
{% highlight ruby %}
  DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }
{% endhighlight %}

데이터배이스 생성하려면 아래와 같이 실행한다.    
`$ python manage.py migrate`

서버를 띄워보자  
`$ python manage.py runserver`

`http://127.0.0.1:8000/`

<br><br><br>
<br><br><br>

# 장고 모델  

`$ python manage.py startapp blog`  

{% highlight ruby %}
djangogirls
├── mysite
|       __init__.py
|       settings.py
|       urls.py
|       wsgi.py
├── manage.py
└── blog
  ├── migrations
  |       __init__.py
  ├── __init__.py
  ├── admin.py
  ├── models.py
  ├── tests.py
  └── views.py
{% endhighlight %}

mysite/settings.py  
{% highlight ruby %}
INSTALLED_APPS = (
     'django.contrib.admin',
     'django.contrib.auth',
     'django.contrib.contenttypes',
     'django.contrib.sessions',
     'django.contrib.messages',
     'django.contrib.staticfiles',
     'blog',
 )
{% endhighlight %}


<br><br><br>

# 블로그 글 모델 만들기  

모든 Model 객체는 아래의 파일에 선언하여 모델을 만든다.  
blog/models.py  
{% highlight ruby %}
from django.db import models
    from django.utils import timezone


    class Post(models.Model): # 모델을 정의하는 코드
        author = models.ForeignKey('auth.User')
        title = models.CharField(max_length=200)
        text = models.TextField()
        created_date = models.DateTimeField(
                default=timezone.now)
        published_date = models.DateTimeField(
                blank=True, null=True)

        def publish(self):  # publish라는 메서드를 생성
            self.published_date = timezone.now()
            self.save()

        def __str__(self):
            return self.title
{% endhighlight %}

<br><br><br>

# 데이터베이스 모델 만들기  

`$ python manage.py makemigrations blog`  
`$ python manage.py migrate blog`  

<br><br><br>

# Django 관리자  

blog/admin.py  
{% highlight ruby %}
from django.contrib import admin
from .models import Post

admin.site.register(Post)
{% endhighlight %}

`$ python manage.py runserver`  

![admin](https://tutorial.djangogirls.org/ko/django_admin/images/login_page2.png)  


로그인을 하려면 슈퍼유저를 생성해야 한다.  
`$ python manage.py createsuperuser`  

kkwonsy  
ssyang0@naver.com  


<br><br><br>

## 배포하기  

아래의 Pythonanywhere는 배포 과정이 간단한 무료 서버를 제공을 해준다.  
https://pythonanywhere.com/  

Github에 코드를 저장하고 PythonAnywhere가 이를 사용하도록 한다.  

# Git 셋팅  

- .gitignore  
{% highlight ruby %}
\*.pyc
__pycache__
myvenv
db.sqlite3
.DS_Store
{% endhighlight %}

- github > new repository  
  - my-first-blog  
  - https://github.com/sungyoungKwon85/my-first-blog.git  


{% highlight ruby %}
$ git config --global user.name "sungyoungKwon85"  
$ git config --global user.email "ssyang0@naver.com"  
$ git add --all
$ git commit -m "blah"  
$ git remote add origin https://github.com/sungyoungKwon85/my-first-blog.git
$ git push -f origin master
{% endhighlight %}



























.
