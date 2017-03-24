---
layout: post
title:  "python-scrapy"
date:   2017-02-23 00:10:53 +0900
categories: python-scrapy
---

BeautifulSoup과 Scrapy가 인기있는 크롤링 라이브러리  
Scrapy가 기능이 더 다양하다.  

<br><br>
<br><br>
<br><br>

//brew : 맥에서 패키지를 관리해줌 apt와 비슷함  
https://brew.sh/  
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"  

<br><br>

//brew 이용해 wget 설치  
$ brew install wget  

<br><br>

//brew 버전확인  
$ brew -v  

<br><br>

// pip : python으로 만들어진 패키지를 관리하는 녀석  
$ sudo easy_install pip  

<br><br>
<br><br>
<br><br>


동영상 강의  
https://www.youtube.com/watch?v=CwkhdqfyvrM&index=3&list=PLWUxS6i2fXtip8sHElwRUubwWfLowlFA4  

<br><br>

virtualenv에 대한 설명  
http://hackersstudy.tistory.com/43  

<br><br>

// six어쩌구저쩌구 인스톨 실패나서 ignore키워드 추가해줬음  
$ sudo pip install --ignore-installed six virtualenv virtualenvwrapper  

<br><br>

// mac을 위한 alias설정법  
http://blog.pigno.se/post/130756784138/mac-%EC%9C%A0%EC%A0%80%EB%A5%BC-%EC%9C%84%ED%95%9C-alias-%EC%84%A4%EC%A0%95  

<br><br>

// 설정  
$ vi bash_profile  
export WORKON_HOME=$HOME/.virtualenvs  
source /usr/local/bin/virtualenvwrapper.sh  

<br><br>

// 갱신  
$ source ~/.bash_profile  
virtualenvwrapper......  

<br><br>

// 사용할 가상환경 이름  
$ mkvirtualenv kkwonsy_python  

<br><br>

// 가상환경 종료  
$ deactivate  

<br><br>

// 가상환경 들어가기  
$ workon kkwonsy_python  

<br><br>

// 가상환경 경로  
$ cd ~/.virtualenvs/  

<br><br>

// scrapy 설치  
$ pip install scrapy  

<br><br>
<br><br>
<br><br>
<br><br>

// 튜토리얼 프로젝트 생성  
$ workon kkwonsy_python  
$ scrapy startproject tutorial  
// 디렉터리가 생성된 것을 볼 수 있다  

<br><br>

// Pycharm 설치  
https://www.jetbrains.com/pycharm/  

<br><br>

Pycharm 실행  
* mac은 디렉터리 찾을 때 command + shift + G > ~/.virtualenvs  
Setting > Project Interpreter > ~/.virtualenvs/bin/python2.7 선택 > OK  
Project Open > 디렉터리 선택

<br><br>

튜토리얼   
https://doc.scrapy.org/en/latest/intro/tutorial.html  

<br><br>

// 명령어 정리  
// 실행  
$ scrapy crawl [spider name]  

<br><br>

// scrapy 쉘에 들어가볼 수 있다  
$ scrapy shell "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/"  

<br><br>

// json으로 추출  
scrapy crawl -o result.json  


// robots.txt  
ppomppu.co.kr/robots.txt 확인하면 크롤링 허용 정보를 알 수 있다.  



<br><br><br>

# scrapy-redis  
https://scrapy-redis.readthedocs.io/en/stable/readme.html  





























.
