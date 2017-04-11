---
layout: post
title:  "tensorflow-basic"
date:   2017-03-25 16:10:53 +0900
categories: tensorflow
---

# [유투브]  
[유투브]: https://www.youtube.com/watch?v=BQEhUD2XTaATensorflow  
# [유투브 MNIST test source]  
[유투브 MNIST test source]: https://github.com/ahn-kj/javacafe-tensorflow
# [강의 사이트]  
[강의 사이트]: http://hunkim.github.io/ml/

<br><br><br>
# [Tensorflow 기본사용법]   
[Tensorflow 기본사용법]: https://tensorflowkorea.gitbooks.io/tensorflow-kr/content/g3doc/get_started/basic_usage.html  
<br>

- graph  
- Session  
- tensor  
- operation(op)  
- feed/fetch  

연산은 graph로 표현  
graph는 Session내에서 실행  
데이터는 tensor로 표현  
graph에 있는 노드는 작업(op)(작업(operation)의 약자)라고 부른다.  
작업(op)은 0개 혹은 그 이상의 Tensor를 가질 수 있고 연산도 수행하며 0개 혹은 그 이상의 Tensor를 만들어 내기도 한다.  
tensor는 정형화된 다차원 배열  
Session은 graph의 작업을 CPU나 GPU같은 Device에 배정하고 실행을 위한 메서드들을 제공  
메서드들은 작업(op)을 실행해서 tensor를 만들어 낸다.  


<br><br><br>
### graph 만들기  
{% highlight ruby %}
import tensorflow as tf

# 1x2 행렬을 만드는 constant op을 만들어 봅시다.
# 이 op는 default graph에 노드로 들어갈 것입니다.
#
# 생성함수에서 나온 값은 constant op의 결과값입니다.
matrix1 = tf.constant([[3., 3.]])

# 2x1 행렬을 만드는 constant op을 만들어봅시다.
matrix2 = tf.constant([[2.],[2.]])

# 'matrix1'과 'matrix2를 입력값으로 하는 Matmul op(역자 주: 행렬곱 op)을
# 만들어 봅시다.
# 이 op의 결과값인 'product'는 행렬곱의 결과를 의미합니다.
product = tf.matmul(matrix1, matrix2)
{% endhighlight %}


<br><br><br>
### session에서 graph 실행하기  
{% highlight ruby %}
# default graph를 실행시켜 봅시다.
sess = tf.Session()

# 행렬곱 작업(op)을 실행하기 위해 session의 'run()' 메서드를 호출해서 행렬곱
# 작업의 결과값인 'product' 값을 넘겨줍시다. 그 결과값을 원한다는 뜻입니다.
#
# 작업에 필요한 모든 입력값들은 자동적으로 session에서 실행되며 보통은 병렬로
# 처리됩니다.
#
# 'run(product)'가 호출되면 op 3개가 실행됩니다. 2개는 상수고 1개는 행렬곱이죠.
#
# 작업의 결과물은 numpy `ndarray` 오브젝트인 result' 값으로 나옵니다.
result = sess.run(product)
print(result)
# ==> [[ 12.]]

# 실행을 마치면 Session을 닫읍시다.
sess.close()
{% endhighlight %}

연산에 쓰인 시스템 자원을 돌려보내려면 session을 닫아야 한다.  
시스템 자원을 더 쉽게 관리하려면 with 구문을 쓰면 된다.  
{% highlight ruby %}
with tf.Session() as sess:
  result = sess.run([product])
  print(result)
{% endhighlight %}
만약 복수의 GPU를 사용한다면 device 구문을 사용해야 한다.  


<br><br><br>
### Interactive Usage   
{% highlight ruby %}
# 인터렉티브 TensorFlow Session을 시작해봅시다.
import tensorflow as tf
sess = tf.InteractiveSession()

x = tf.Variable([1.0, 2.0])
a = tf.constant([3.0, 3.0])

# 초기화 op의 run() 메서드를 이용해서 'x'를 초기화합시다.
x.initializer.run()

# 'x'에서 'a'를 빼는 작업을 추가하고 실행시켜서 결과를 봅시다.
sub = tf.sub(x, a)
print(sub.eval())
# ==> [-2. -1.]

# 실행을 마치면 Session을 닫읍시다.
sess.close()
{% endhighlight %}


<br><br><br>
### Variable  
{% highlight ruby %}
# 값이 0인 스칼라로 초기화된 변수를 만듭니다.
state = tf.Variable(0, name="counter")

# 'state'에 1을 더하는 작업(op)을 만듭니다.

one = tf.constant(1)
new_value = tf.add(state, one)
update = tf.assign(state, new_value)

# 그래프를 한 번 작동시킨 후에는 'init' 작업(op)을 실행해서 변수를 초기화해야
# 합니다. 먼저 'init' 작업(op)을 추가해 봅시다.
init_op = tf.global_variables_initializer()

# graph와 작업(op)들을 실행시킵니다.
with tf.Session() as sess:
  # 'init' 작업(op)을 실행합니다.
  sess.run(init_op)
  # 'state'의 시작값을 출력합니다.
  print(sess.run(state))
  # 'state'값을 업데이트하고 출력하는 작업(op)을 실행합니다.
  for _ in range(3):
    sess.run(update)
    print(sess.run(state))

# output:

# 0
# 1
# 2
# 3
{% endhighlight %}
그래프를 실행하더라도 변수(variable)의 상태는 유지된다.  
assign() 작업은 add() 작업처럼 graph의 한 부분, 그래서 run()이 graph를 실행시킬 때까지 실제로 작동하지 않는다.  




<br><br><br>
### Fetches  
{% highlight ruby %}
input1 = tf.constant([3.0])
input2 = tf.constant([2.0])
input3 = tf.constant([5.0])
intermed = tf.add(input2, input3)
mul = tf.mul(input1, intermed)

with tf.Session() as sess:
  result = sess.run([mul, intermed])
  print(result)

# output:
# [array([ 21.], dtype=float32), array([ 7.], dtype=float32)]
{% endhighlight %}
 복수의 tensor를 받아올 수도 있다.  




<br><br><br>
### Feeds  
graph의 연산에게 직접 tensor 값을 줄 수 있는 'feed 메커니즘'도 제공  
가장 일반적인 사용방법은 tf.placeholder()를 사용해서 특정 작업(op)을 "feed" 작업으로 지정해 주는 것  

{% highlight ruby %}
input1 = tf.placeholder(tf.float32)
input2 = tf.placeholder(tf.float32)
output = tf.mul(input1, input2)

with tf.Session() as sess:
  print(sess.run([output], feed_dict={input1:[7.], input2:[2.]}))

# output:
# [array([ 14.], dtype=float32)]
{% endhighlight %}





.
<br><br><br>

{% highlight ruby %}
{% endhighlight %}
