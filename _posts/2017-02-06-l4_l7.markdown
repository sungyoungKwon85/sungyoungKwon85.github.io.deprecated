BeautifulSoup과 Scrapy가 인기있는 크롤링 라이브러리  
Scrapy가 기능이 더 다양하다.  



//brew : 맥에서 패키지를 관리해줌 apt와 비슷함
https://brew.sh/
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

//brew 이용해 wget 설치
$ brew install wget

//brew 버전확인
$ brew -v

// pip : python으로 만들어진 패키지를 관리하는 녀석
$ sudo easy_install pip

------------
동영상 강의
https://www.youtube.com/watch?v=CwkhdqfyvrM&index=3&list=PLWUxS6i2fXtip8sHElwRUubwWfLowlFA4


virtualenv에 대한 설명
http://hackersstudy.tistory.com/43


// six어쩌구저쩌구 인스톨 실패나서 ignore키워드 추가해줬음
$ sudo pip install --ignore-installed six virtualenv virtualenvwrapper


// mac을 위한 alias설정법
http://blog.pigno.se/post/130756784138/mac-%EC%9C%A0%EC%A0%80%EB%A5%BC-%EC%9C%84%ED%95%9C-alias-%EC%84%A4%EC%A0%95

// 설정
$ vi bash_profile
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh


// 갱신
$ source ~/.bash_profile
virtualenvwrapper......



// 사용할 가상환경 이름
$ mkvirtualenv kkwonsy_python


// 가상환경 종료
$ deactivate

// 가상환경 들어가기
$ workon kkwonsy_python

// 가상환경 경로
$ cd ~/.virtualenvs/


// scrapy 설치
$ pip install scrapy
