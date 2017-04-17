---
layout: post
title:  "gensim-word2vec"
date:   2017-04-12 11:10:53 +0900
categories: machine-learning
---

## [Mining English and Korean text with Python]  
[Mining English and Korean text with Python]: https://www.lucypark.kr/courses/2015-ba/text-mining.html#  
한글 : https://www.lucypark.kr/courses/2015-dm/text-mining.html  

**use Python 3**  

<br>

# 용어 정리  
- Corpus	말뭉치	A set of documents  
- Morphemes	형태소	Smallest meaningful unit in a language  
- POS	품사	Part-of-speech (ex: Nouns)  
- Clustering	군집화	An unsupervised learning task where XX is given  

<br><br>

# 프로세스  
- Load text
- Tokenize text (ex: stemming, morph analyzing)
- Tag tokens (ex: POS, NER)
- Token(Feature) selection and/or filter/rank tokens (ex: stopword removal, TF-IDF)
- ...and so on (ex: calculate word/document similarities, cluster documents)

<br><br>

# Python Packages for Text Mining and NLP

http://konlpy.org/ko/v0.4.3/install/  

{% highlight ruby %}
# 1. install NLTK(자연어처리용 python lib)
# 2. install KoNLPy(한글처리)
# 3. install Gensim(text mining lib, word2vec지원함)
# 4. install Twitter API(twitter api 쉽게 사용하기 위함)
pip install nltk
pip install konlpy
pip3 install konlpy # python3
pip install -U gensim
pip install twython
{% endhighlight %}



<br><br>

# Text exploration  
{% highlight ruby %}
# 1. Read document
from konlpy.corpus import kobill    # Docs from pokr.kr/bill
files_ko = kobill.fileids()         # Get file ids
doc_ko = kobill.open('1809890.txt').read()



# 2. Tokenize
from konlpy.tag import Twitter; t = Twitter()
tokens_ko = t.morphs(doc_ko)

#여기서 no module named 'jpype' 라는 오류가 발생.
# http://jpype.readthedocs.io/en/latest/install.html 를 참고하여 아래의 명령어를 사용

$ conda install -c conda-forge jpype1

# t = Twitter() 부분에서 아래와 같은 에러가 발생.... 뭘까?
# TypeError: Package kr.lucypark.tkt.TktInterface is not Callable




# 3. Load tokens with nltk.Text()
# nltk.Text() is a convenient way to explore a current document.
import nltk
ko = nltk.Text(tokens_ko, name='테스트명입니다.')

# 3-1. Get Tokens info
print(len(ko.tokens))       # returns number of tokens (document length)
print(len(set(ko.tokens)))  # returns number of unique tokens
ko.vocab()     

# 3-2. chart
ko.plot(50)

# 3-3. with count
ko.count('초등학교')

# 3-4. distribute chart
ko.dispersion_plot(['육아휴직', '초등학교', '공무원'])

# 3-5. concordance (색인)
ko.concordance('초등학교')

# 3-6. find similar words
ko.similar('자녀')
ko.similar('육아휴직')
{% endhighlight %}
**20170412**  
- No module named 'jpype' 라는 에러가 출력되어 아래처럼 추가 설치를 했음.  
{% highlight ruby %}
$ pip3 install JPype1-py3
{% endhighlight %}

- The plot function requires matplotlib to be installed. 에러가 출력되어 아래처럼 추가 설치 했음.  
{% highlight ruby %}
pip install -U matplotlib
{% endhighlight %}

- 아래와 같은 에러메시지 출력되어 [Problem Cause In mac os image rendering back end of matplotlib] 내용 참고하여 해결!  
Python is not installed as a framework. The Mac OS X backend will not be able to function correctly if Python is not installed as a framework. See the Python documentation for more information on installing Python as a framework on Mac OS X. Please either reinstall Python as a framework, or try one of the other backends. If you are using (Ana)Conda please install python.app and replace the use of 'python' with 'pythonw'. See 'Working with Matplotlib on OSX' in the Matplotlib FAQ for more information.  
{% highlight ruby %}
$ cd ~/.matplotlib
$ vi matplotlibrc
backend: TkAgg # 입력
{% endhighlight %}
[Problem Cause In mac os image rendering back end of matplotlib]: http://stackoverflow.com/questions/21784641/installation-issue-with-matplotlib-python

<br><br>


# Tagging and chunking
{% highlight ruby %}
# 1. POS tagging
from konlpy.tag import Twitter; t = Twitter()
tags_ko = t.pos("작고 노란 강아지가 페르시안 고양이에게 짖었다")

# 2. Noun phrase chunking
parser_ko = nltk.RegexpParser("NP: {<Adjective>*<Noun>*}")
chunks_ko = parser_ko.parse(tags_ko)
chunks_ko.draw()
{% endhighlight %}

<br><br>

아래 이미지처럼 NP(명사절)단위로 문장 구조를 보여준다.  
![저장구조](https://www.lucypark.kr/courses/2015-ba/images/tree_ko.png)

<br><br>

# Preprocessing  
{% highlight ruby %}
# 1. Load documents
from konlpy.corpus import kobill
docs_ko = [kobill.open(i).read() for i in kobill.fileids()]

# 2. Tokenize
from konlpy.tag import Twitter; t = Twitter()
pos = lambda d: ['/'.join(p) for p in t.pos(d, stem=True, norm=True)]
texts_ko = [pos(doc) for doc in docs_ko]
print(texts_ko[0])

# 3. Encode tokens to integers
from gensim import corpora
dictionary_ko = corpora.Dictionary(texts_ko)
dictionary_ko.save('ko.dict')  # save dictionary to file for future use

# 4. Calculate TF-IDF
from gensim import models
tf_ko = [dictionary_ko.doc2bow(text) for text in texts_ko]
tfidf_model_ko = models.TfidfModel(tf_ko)
tfidf_ko = tfidf_model_ko[tf_ko]
corpora.MmCorpus.serialize('ko.mm', tfidf_ko) # save corpus to file for future use

# print first 10 elements of first document's tf-idf vector
print(tfidf_ko.corpus[0][:10])
# print top 10 elements of first document's tf-idf vector
print(sorted(tfidf_ko.corpus[0], key=lambda x: x[1], reverse=True)[:10])
# print token of most frequent element
print(dictionary_ko.get(414))
{% endhighlight %}

<br><br>

# Train topic models  
{% highlight ruby %}
# LSI
ntopics, nwords = 3, 5
lsi_ko = models.lsimodel.LsiModel(tfidf_ko, id2word=dictionary_ko, num_topics=ntopics)
print(lsi_ko.print_topics(num_topics=ntopics, num_words=nwords))

# LDA
import numpy as np; np.random.seed(42)  # optional
lda_ko = models.ldamodel.LdaModel(tfidf_ko, id2word=dictionary_ko, num_topics=ntopics)
print(lda_ko.print_topics(num_topics=ntopics, num_words=nwords))

# HDP
import numpy as np; np.random.seed(42)  # optional
hdp_ko = models.hdpmodel.HdpModel(tfidf_ko, id2word=dictionary_ko)
print(hdp_ko.print_topics(topics=ntopics, topn=nwords))
{% endhighlight %}

<br><br>

# Scoring documents  
{% highlight ruby %}
bow = tfidf_model_ko[dictionary_ko.doc2bow(texts_ko[0])]
sorted(lsi_ko[bow], key=lambda x: x[1], reverse=True)
sorted(lda_ko[bow], key=lambda x: x[1], reverse=True)
sorted(hdp_ko[bow], key=lambda x: x[1], reverse=True)

bow = tfidf_model_ko[dictionary_ko.doc2bow(texts_ko[1])]
sorted(lsi_ko[bow], key=lambda x: x[1], reverse=True)
sorted(lda_ko[bow], key=lambda x: x[1], reverse=True)
sorted(hdp_ko[bow], key=lambda x: x[1], reverse=True)
{% endhighlight %}


<br><br>

# Word embedding(word2vec)  
{% highlight ruby %}
# 1. Load documents
from konlpy.corpus import kobill
docs_ko = [kobill.open(i).read() for i in kobill.fileids()]

# 2. Tokenize
from konlpy.tag import Twitter; t = Twitter()
pos = lambda d: ['/'.join(p) for p in t.pos(d)]
texts_ko = [pos(doc) for doc in docs_ko]

# 3. Train
from gensim.models import word2vec
wv_model_ko = word2vec.Word2Vec(texts_ko)
wv_model_ko.init_sims(replace=True)
wv_model_ko.save('ko_word2vec.model')

# 4. test
wv_model_ko.most_similar(pos('정부'))
wv_model_ko.most_similar(pos('초등학교'))
{% endhighlight %}


<br><br><br><br><br><br>

## [word2vec 모델]  
[word2vec 모델]: https://tensorflowkorea.gitbooks.io/tensorflow-kr/content/g3doc/tutorials/word2vec/  



<br><br><br>
# 왜 Word Embeddings 를 배워야하지?  
이미지, 오디오 처리 시스템들을 데이터에 대하여 개별의 가공되지 않은, 이를테면 픽셀=강도 값의 벡터, 파워스펙트럴밀도계수로 기록된 데이터셋 등을 포함하고 있다.  
반면에 **자연어 처리 시스템** 들은 일반적으로 discrete atomic symbols. 즉 'cat' = 'Id537' 등으로 표현함으로써 개별 symbol들 간의 관계에는 **의미없는 정보를 제공** 한다.  
이것은 성공적으로 학습시키기 위해서 더 많은 데이터가 필요함을 의미한다.

**벡터** 표현을 사용하면 여러 장애들을 해결 할 수 있다.
**word2vec는 특히 가공되지 않은 text로부터 학습한 단어 Embeddings에 대해 계산적으로 효율적인 예측 모델이다.**  
두 가지 종류로 나타난다.  
- Continuous Bag-of-Word  
- Skip-Gram  

알고리즘에 대해서는 링크를 참조할 것.  

<br><br><br>

# Graph 구성하기  
embedding 매트릭스를 정의해보자.(시작하기 위한 랜덤 매트릭스)  

{% highlight ruby %}
# 유닛 큐브의 값이 균일하도록 초기화
embeddings = tf.Variable(
    tf.random_uniform([vocabulary_size, embedding_size], -1.0, 1.0))

# 어휘(vocabulary) 의 각 단어에 대한 가중치(weights)와 편향(biases) 을 정의
nce_weights = tf.Variable(
    tf.truncated_normal([vocabulary_size, embedding_size],
                        stddev=1.0 / math.sqrt(embedding_size)))
  nce_biases = tf.Variable(tf.zeros([vocabulary_size]))

# Placeholders for inputs
train_inputs = tf.placeholder(tf.int32, shape=[batch_size])
train_labels = tf.placeholder(tf.int32, shape=[batch_size, 1])

# 집단(batch) 안의 각 원본 단어들에 대한 벡터를 살펴본다
# 각 단어에 대한 embeddings 을 만든다
embed = tf.nn.embedding_lookup(embeddings, train_inputs)

# noise-contrastive 학습 목적함수를 이용한 타겟 단어 예측을 시도
# 매번 음수 라벨링 된 셈플을 이용한 NCE loos 계산
loss = tf.reduce_mean(
  tf.nn.nce_loss(nce_weights, nce_biases, embed, train_labels,
                 num_sampled, vocabulary_size))

# gradients 를 계산하고 파라미터들을 업데이트 하는 등에 필요한 노드들을 추가
# SGD optimizer 를 사용
optimizer = tf.train.GradientDescentOptimizer(learning_rate=1.0).minimize(loss)

# 모델을 학습
for inputs, labels in generate_batch(...):
  feed_dict = {training_inputs: inputs, training_labels: labels}
  _, cur_loss = session.run([optimizer, loss], feed_dict=feed_dict)
{% endhighlight %}
,_
학습이 완료된 후 t-SNE 를 사용하여 학습한 embeddings 를 시각화 할 수 있다.  
예상한 것 처럼 **비슷한 단어들은 결국 서로 가까운 집단화(clustering)** 된다.  

<br><br><br>
<br><br><br>
## [한국어 Word2Vec]  
[한국어 Word2Vec]: http://blog.theeluwin.kr/post/146591096133/%ED%95%9C%EA%B5%AD%EC%96%B4-word2vec  

**파이썬 gensim** 을 사용해서 **한국어 Word2Vec을 구현** 한다.  
Word2Vec 알고리즘을 내장하고 있는 gensim 라이브러리를 사용해서 빠르게 한국어 Word2Vec을 구현하는 것만 소개한다.  
좋은 성능을 내고 있는 한국어 Word2Vec 사이트로는 [elnn이 만든 Korean Word2Vec]이 있으니 참고할 것.  

gensim을 쓰기에 앞서 우선 **corpus** 를 준비해야한다.  
그리고 같은 단어라도 품사가 다를 경우 다르게 인식해줘야하기 때문에 적당한 형태소 분석기를 사용해서 **POS 태깅** 을 해주는게 좋다.  
- [형태 분석기 차이]

해당 링크의 작성자의 경우 너무 세세하게 나눠주는 타 형태소 분석기보다 [Twitter의 형태소 분석기]가 적합하다고 판단했다.  


[Twitter의 형태소 분석기]: https://github.com/twitter/twitter-korean-text
[형태 분석기 차이]: http://konlpy.org/en/v0.4.4/morph/#comparison-between-pos-tagging-classes  
[elnn이 만든 Korean Word2Vec]: http://w.elnn.kr/search/


<br><br><br>
<br><br><br>




















<br><br><br>

{% highlight ruby %}
{% endhighlight %}
