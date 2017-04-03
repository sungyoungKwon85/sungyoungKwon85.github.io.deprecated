---
layout: post
title:  "tensorflow-mnist1"
date:   2017-03-25 17:10:53 +0900
categories: tensorflow
---


모델이 이미지를 보고 어떤 숫자인지 예측하는 모델을 훈련시킬 것이다.  
소프트맥스 회귀라고 불리는 아주 간단한 모델로 시작한다.  


### file download  
{% highlight ruby %}
>>> from tensorflow.examples.tutorials.mnist import input_data
>>> mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
{% endhighlight %}

다운로드된 데이터는 55,000개의 학습 데이터(mnist.train), 10,000개의 테스트 데이터(mnist.text), 그리고 5,000개의 검증 데이터(mnist.validation) 이렇게 세 부분이다.    

<br><br><br>
각 이미지는 28x28 픽셀  
이 배열을 펼쳐서 28x28 = 784 개의 벡터로 만들 수 있다.  
일관적으로 처리하기만 한다면, 배열을 어떻게 펼치든지 상관없다.  

<br><br><br>
데이터를 펼친 결과로 mnist.train.images는 [55000, 784]의 형태를 가진 텐서(n차원 배열)가 된다.  
첫 번째 차원은 이미지를 가리키며, 두 번째 차원은 각 이미지의 픽셀을 가르킨다.  
텐서의 모든 성분은 특정 이미지의 특정 픽셀을 특정하는 0과 1사이의 픽셀 강도이다.  

<br><br><br>
MNIST에서 각각에 대응하는 라벨은 0과 9사이의 숫자이다.  
단 하나의 차원에서만 1이고, 나머지 차원에서는 0인 벡터로 바꾼다.  
예를 들어서, 3은 [0,0,0,1,0,0,0,0,0,0]  
결과적으로, mnist.train.labels는 [55000, 10]의 모양을 같은 실수 배열이 된다.  
(수 배열로 취급하는 데에는 이후 소프트맥스 회귀의 결과가 정수형이 아닌 실수형으로 산출되기 때문)  



<br><br><br>
<br><br><br>
### softmax regression  

 이미지는 10가지의 경우의 수 중 하나  
 모델은 9가 쓰여져 있는 이미지를 보고 이 이미지가 80%의 확률로 9라고 추측하지만, 8일 확률도 5% 있다고 계산할 수도 있다.  
 이 상황은 소프트맥스 회귀를 사용하기에 아주 적절한 예이다.  

 소프트맥스 회귀는 두 단계로 이루어진다.  
 우선 입력한 데이터가 각 클래스에 속한다는 증거(evidence)를 수치적으로 계산하고, 그 뒤엔 계산한 값을 확률로 변환한다.  
...


<br><br><br>
<br><br><br>
### regression 구현하기  
파이썬에서 효율적인 수치 연산을 하기 위해 다른 언어로 구현된 보다 효율이 높은 코드를 사용하여 행렬곱 같은 무거운 연산을 수행하는 NumPy등의 라이브러리를 사용한다.  
매 연산마다 파이썬으로 다시 돌아오는 과정에서 여전히 많은 오버헤드가 발생할 수 있다.  
이러한 오버헤드는 GPU에서 연산을 하거나 분산 처리 환경같은, 데이터 전송에 큰 비용이 발생할 수 있는 상황에서 특히 문제가 될 수 있다.  

텐서플로우는 이런 오버헤드를 피하기 위해 한 단계 더 나아간 방식을 활용한다.  
파이썬에서 하나의 무거운 작업을 독립적으로 실행하는 대신, 텐서플로우는 서로 상호작용하는 연산간의 그래프를 유저가 기술하도록 하고, 그 연산 모두가 파이썬 밖에서 동작한다.  


{% highlight ruby %}
import tensorflow as tf

# 텐서플로우에서 연산을 실행할 때 값을 입력할 자리
# 여기서는 784차원의 벡터로 변형된 MNIST 이미지의 데이터를 넣을 것이다.
x = tf.placeholder(tf.float32, [None, 784])

# 모델에는 가중치와 바이어스 역시 필요하다.
# 텐서플로우는 Variable이라고 불리는 좋은 방법을 갖고 있다.
# Variable은 서로 상호작용하는 연산으로 이루어진 텐서플로우 그래프 안에 존재하는, 수정 가능한 텐서이다.  
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))

# 모델을 구현한다.
y = tf.nn.softmax(tf.matmul(x, W) + b)
{% endhighlight %}


<br><br><br>
<br><br><br>
### 학습  
모델을 학습시키기 위해서는 우선 모델이 좋다는 것은 어떤 것인지를 정의해야 한다.  
모델의 손실을 정의하기 위해 자주 사용되는 좋은 함수 중 하나로 "크로스 엔트로피"가 있다.  

{% highlight ruby %}
# 크로스 엔트로피를 구현하기 위해서는 올바른 답을 넣기 위한 새로운 placeholder를 추가하는 것 부터 시작해야 한다.
y_ = tf.placeholder(tf.float32, [None, 10])
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))

# 텐서플로우에게 학습 비율 0.5로 경사 하강법(gradient descent algorithm)을 적용하여 크로스 엔트로피를 최소화하도록 지시
# 텐서플로우는 다른 여러 최적화 알고리즘을 제공하므로 그 중 하나를 적용하는 것은 코드 한 줄만 수정하면 된다.
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)

# 작성한 변수들을 초기화
init = tf.global_variables_initializer()
# module 'tensorflow' has no attribute 'global_variables_initializer'
# initialize_all_variables로 대체 사용했다.

# 세션에서 모델 실행 및 변수 초기화 실행
sess = tf.Session()
sess.run(init)

# 학습 실행
for i in range(1000):
  batch_xs, batch_ys = mnist.train.next_batch(100)
  sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})

{% endhighlight %}



<br><br><br>
<br><br><br>
### 모델 평가하기  

{% highlight ruby %}
# 부울 값으로 이루어진 리스트를 얻게 된다.
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))

accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

print(sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
{% endhighlight %}


<br><br><br>
<br><br><br>
### 모델 평가하기  
결과는 약 92% 정도 나왔다.  
좋은 결과는 아니다. 좋은 모델은 99.7%도 넘을 수 있다.  
다음 튜토리얼을 통해 더 좋은 결과를 얻을 수 있다.  



<br><br><br>

{% highlight ruby %}
{% endhighlight %}
