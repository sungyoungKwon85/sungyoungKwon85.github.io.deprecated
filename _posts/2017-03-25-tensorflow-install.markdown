---
layout: post
title:  "tensorflow-install"
date:   2017-03-25 15:10:53 +0900
categories: machine-learning
---

머신 러닝 관련된 내용들을 학습한다.  



<br><br><br>

# [Tensorflow 다운로드 및 설치 kr]  
[Tensorflow 다운로드 및 설치 kr]: https://tensorflowkorea.gitbooks.io/tensorflow-kr/content/g3doc/get_started/os_setup.html  
# [Tensorflow 다운로드 및 설치 en]  
[Tensorflow 다운로드 및 설치 en]: https://www.tensorflow.org/install/install_mac  

<br><br><br>
### 가상환경 사용하기  
가상환경은 virtualevn 말고도 anaconda, docker 등을 선택할 수 있다.  
**We recommend the virtualenv installation.**  
**within Anaconda, we recommend installing TensorFlow with the pip install command, not with the conda install command.**  
**We strongly recommend pip or pip3 version 8.1 or higher**  

{% highlight ruby %}
$ mkdir hellotensorflow
$ cd hellotensorflow
$ python3 -m venv myvenv
~/hellotensorflow$ source myvenv/bin/activate
{% endhighlight %}

<br><br><br>
### tensorflow 설치  
GPU는 mac에서 테스트할 수 없기 때문에 skip한다.  

{% highlight ruby %}
# Mac OS X, CPU 전용, Python 3.4 or 3.5:
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/tensorflow-0.9.0-py3-none-any.whl

$ sudo pip3 install --upgrade $TF_BINARY_URL
{% endhighlight %}


<br><br><br>
### tensorflow 설치 테스트    
{% highlight ruby %}
$ python
...
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
  Hello, TensorFlow!
a = tf.constant(10)
b = tf.constant(32)
print(sess.run(a + b))
  42
{% endhighlight %}

**20170411**  
no module named tensorflow 라는 문구가 뜨면서 import가 안되는 경우가 있다.  
{% highlight ruby %}
$ pip install --upgrate tensorflow # 시도
{% endhighlight %}
0.9.0 설치했고 현재는 1.0.1 버전이 최신이다.  
여전히 동작하지 않는다.  
아래와 같이 버전을 확인해보니 anaconda와 연결이 되어 있다.  
현재 아나콘다를 사용하는 프로젝트가 없으므로 [아나콘다 삭제] 를 시도.  
{% highlight ruby %}
$ python3 --version
Python 3.6.0 :: Anaconda 4.3.1 (x86_64)

$ conda install anaconda-clean
$ anaconda-clean
$ rm -rf ~/anaconda3
$ sudo vi .bash_profile
# anaconda 경로를 주석처리 해줘야 한다.
bash: /Users/1100459/anaconda/bin/python3: No such file or directory
# 뭐지?
$ virtualenv
$ python3 --version
anaconda/bin/virtualenv: No such file or directory

$ python --version
# mac에는 기본적으로 2.7 버전의 python이 설치되어 있다.

$ source .bash_profile
# 이거 해줘야 갱신됨 -.-;;

# 이제 tensorflow import 된다~~~
# 아래는 공식홈 튜토리얼
$ sudo easy_install pip
$ sudo pip install --upgrade virtualenv
$ virtualenv --system-site-packages targetDirectory
$ source ~/tensorflow/bin/activate
pip3 install --upgrade tensorflow     # for Python 3.n
{% endhighlight %}
[아나콘다 삭제]: https://docs.continuum.io/anaconda/install#os-x-uninstall



<br><br><br>
### tensorflow 데모 모델 실행하기  
데모 모델을 포함, 모든 텐서플로우의 패키지는 파이썬 라이브러리로 설치되어 있다.  
아래 명령으로 라이브러리의 정확한 디렉토리를 찾을 수 있다.  
{% highlight ruby %}
python3 -c 'import os; import inspect; import tensorflow; print(os.path.dirname(inspect.getfile(tensorflow)))'
{% endhighlight %}

MNIST 데이터셋을 이용한 손글씨 숫자를 분류하는 간단한 데모 모델은 models/image/mnist/convolutional.py 에 있다.  
(GPU가 없어 매우 느림...)
{% highlight ruby %}
# 파이썬 검색 범위에서 프로그램을 찾기 위해서 'python -m' 명령을 이용한다.:
# 데모 모델을 실행한다.
$ python3 -m tensorflow.models.image.mnist.convolutional
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
...etc...

# 파이썬 인터프리터에 모델 프로그램의 파일 경로를 전달할 수 있다.
# (텐서플로우가 설치된 파이썬 버전을 사용해야 한다.
# 예를 들어, 파이썬 3의 경우는 .../python3.X/... 가 된다.).
$ python /usr/local/lib/python3.5/dist-packages/tensorflow/models/image/mnist/convolutional.py
...
{% endhighlight %}


<br><br><br>
<br><br><br>
# [Tensorflow 기본사용법]  
[Tensorflow 기본사용법]: https://tensorflowkorea.gitbooks.io/tensorflow-kr/content/g3doc/get_started/basic_usage.html  
- 연산은 graph로 표현  
- graph는 Session내에서 실행  
- 데이터는 tensor(정형화된 다차원 배열)로 표현  
- Variable은 그 상태를 유지  
- operation(graph에 있는 노드)에서 데이터를 입출력할 때 feed와 fetch를 사용할 수 있음  
- operation은 0개 이상의 Tensor를 가질 수 있음  

<br><br><br>

<br><br><br>

<br><br><br>

# [Conda Install and settings link 1]  
[Conda Install and settings link 1]: http://creativeworks.tistory.com/entry/%EC%95%84%EB%82%98%EC%BD%98%EB%8B%A4Anaconda%EC%97%90-TensorFlow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-on-Mac-Installing-TensorFlow-at-Anaconda-on-MAC-OS-X
# [Conda Install and settings link 2]  
[Conda Install and settings link 2]: http://egloos.zum.com/mataeoh/v/7052271

<br><br><br>

# [Conda Download]
[Conda Download]: https://conda.io/docs/download.html

<br><br><br>


# Conda 가상환경 생성 및 tensorflow 설치  
{% highlight ruby %}
#python 3.5
$ conda create -n tensorflow python=3.5

$ source activate tensorflow

# Mac OS X, CPU 전용, Python 3.4 or 3.5:
(tensorflow)$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.0.1-py3-none-any.whl

# Python 3
(tensorflow)$ pip3 install --upgrade $TF_BINARY_URL
{% endhighlight %}

# 가상환경 정보 보기
$ conda info --envs

<br><br><br>

.
{% highlight ruby %}
{% endhighlight %}
