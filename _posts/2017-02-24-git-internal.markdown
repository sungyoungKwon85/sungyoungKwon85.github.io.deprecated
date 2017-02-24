---
layout: post
title:  "Git-internal"
date:   2017-02-24 15:10:53 +0900
categories: Git
---

### GIT ITNTERNAL

SNAPSHOT으로 동작하므로 빠르다?  
DELTA?  
* PACKAGE로 관리, DELTA를 관리하는 것을 자동으로  
* 가장 최근 것을 SNOPSHOT으로, 나머지는 DELTA  
참고 : http://stackoverflow.com/questions/8198105/how-does-git-store-files  
<br><br>

DELTA?  
![DELTA](https://i.stack.imgur.com/O98qj.png)  
<br><br>

모든 파일을 해시로 저장한다.(SHA-1)  
<br><br>

`$ git add README`  
* index  

`$ git is-files --staged` #index에 등록된 파일 확인  

`$ git tree`  

`$ git commit -m "add README.md file"` # 이 때 SNAPSHOP이 된다.  
<br><br>

[commit FLOW]  
![commit FLOW](http://codingdomain.com/git/partial-commits/git-staging-area.png)  
<br><br><br>

참고  
http://blog.naver.com/PostView.nhn?blogId=choigohot&logNo=40192765409  


`$ git add README test.rb LICENSE`  
`$ git commit -m 'initial commit of my project'`  

- 각 파일에 대한 Blob 세 개, 파일/디렉토리 구조가 들어있는 트리 개체 하나, 메타데이터와 루트 트리를 가리키는 포인터가 담긴 커밋 개체 하나가 생성된다.
![구조](http://postfiles1.naver.net/20130710_160/choigohot_1373441976273C1dt8_PNG/18333fig0301-tn.png?type=w2)  


<br><br><br>
- 다시 파일을 수정하고 커밋하면 이전 커밋이 무엇인지도 저장한다.   
![구조](http://cfile22.uf.tistory.com/image/011EF04E50F6B6B4226A8A)  

<br><br><br>
<br><br><br>


# 브랜치  
브랜치는 커밋 사이를 가볍게 이동할 수 있는 포인터 같은 것!  
SVN에서 브랜치를 따면 copy&paste개념이다.  
GIT은 기본적으로 master 브랜치를 만든다.  
자동으로 마지막 커밋을 가리키게 한다.  
![구조](http://postfiles14.naver.net/20130710_13/choigohot_1373442015488DTQhi_PNG/18333fig0303-tn.png?type=w2)

<br><br><br>
`$ git branch testing`  
새로 만든 브랜치도 지금 작업하고 있던 마지막 커밋을 가리킨다.  
![구조](http://postfiles10.naver.net/20130710_249/choigohot_1373442042258IJJK4_PNG/18333fig0304-tn.png?type=w2)


<br><br><br>
작업중인 브랜치가 무엇인지 어떻게 알까?  
--> **HEAD**  
Git은 아직 master를 가리키고 있다.  
브랜치를 생성했다고 가리키는게 바뀌진 않는다.  
그럼 어떻게 옮길까?  

`$ git checkout testing`  
![구조](http://postfiles13.naver.net/20130710_124/choigohot_1373442080939ukoT1_PNG/18333fig0306-tn.png?type=w2)


<br><br><br>

다시 커밋하면?  
![구조](http://postfiles2.naver.net/20130710_33/choigohot_1373442107996YvjqX_PNG/18333fig0307-tn.png?type=w2)  

<br><br><br>

$ git checkout master # 이렇게 하면 어떻게 될까?  
![구조](http://postfiles6.naver.net/20130710_133/choigohot_1373442181005E4HLs_PNG/18333fig0309-tn.png?type=w2)  




<br><br><br>
<br><br><br>

# Merge  
참고 :  
https://git-scm.com/book/ko/v1/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-Merge%EC%9D%98-%EA%B8%B0%EC%B4%88  

<br><br><br>





































.
