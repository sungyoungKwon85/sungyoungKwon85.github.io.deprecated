---
layout: post
title:  "clean-code"
date:   2017-05-22 10:10:53 +0900
categories: clean-code
---



### 좋은 습관  
- 의미전달이 명확하게 코드를 작성하자.  
- Clean했던 코드가 경험이 쌓여 발전하면서 더이상 Clean 하지 못할 수 있다  
- 일관된 개발  
- 최소한의 설계는 필요하다 --> UML, class diagram, sequence diagram 정도  
- 약자를 쓸거면 계속 쓰기
- 처리부분과 in/out부분의 구분
- 사용한 interface를 먼저 선언하고 구현은 이후에
- 의미있는 변수명사용  
- packeage, public 등을 고려하자(자동으로 작성되다보니 신경쓰지 않고 넘어갈때가 있음)
- 언어마다 클린코드는 다를 수 있음
- 직관적으로 작성하자
- `혼자 작성하다 보면 자기 스타일에 빠진다, 혼자서 코딩하지 말자`
- javascript, 코드의 분리를 하자.
  - 유의점 : addEventListener를 안쓰고 onClick을 javascipt에 구분시켜도 결국은 dom에 코드가 노출됨.  
- 생성자를 이용해 필듣값을 적용시킨다면 set을 안써도 될 수 있음을 고려.(자동을 get/set을 작성하는 것을 피하자)
- java 공식 api문서를 보면 옛날 코드에는 주석이 많음. 최근 코드를 참고하는게 좋다.
- 최근 코드를 참고해보면 `UnCheckedException을 활용`한다.
- `코드 리펙토링을 할 때는 exception을 활용`하자. JVM 내부적으로 exception을 처리하기 때문에 성능이 훨씬 좋아진다.
- 좋은 주석? UnCheckedException에 대한 내용을 써주자. 나머지는 interface에 명료하게 나타나있으면 쓸 필요가 없다.
- Stack Trace는 아래에서 위로 올라가면서 보자. Thread > Framework > My Source > Framework 이런 순으로 출력된다.  


<br><br><br>


### OOP?  
- 재사용에 따른 유지보수의 효율성을 증대 --> 다형성  
- 인터페이스의 활용  
- Designe Pattern의 활용  
경우의 수를 고려하여 등장한 GoF, MVC등의 패턴..  
GoF의 경우 80~90년대의 경험이 쌓여 등장하게 되었다.  


<br><br><br>


### 그래서..
- `Clean code 책`을 예전에 사고 아직 읽지 않고 있었는데, 잘 봐야 하겠다.
- 좋은 내용들이나 `실제 코드로 연습하면서 익히자`. Text만으로는 습관시킬수가 없다.
- `UML 활용`


<br><br><br>

{% highlight ruby %}
{% endhighlight %}
`
