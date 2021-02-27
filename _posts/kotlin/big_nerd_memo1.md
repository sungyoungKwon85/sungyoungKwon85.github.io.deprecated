# kotlin-practice
빅 너드 랜치의 코틀린 프로그래밍 책을 보며 실습함.
https://github.com/Jpub/BNR_Kotlin
#


### Hello world
```
fun main(args: Array<String>) {
   println("Hello world");
}
```
#


### REPL
파일을 생성하지 않고 코드를 빨리 테스트하는 도구

`IntelliJ -> Tools -> Kotlin -> Kotlin REPL 선택`
#

### JVM에서 실행하기
코틀린 소스 -> (컴파일러) -> 바이트 코드 <- JVM
#

코틀린은 JVM 뿐만 아니라 자바스크립트로도 컴파일 될 수 있으며 특정 플랫폼(Windows, Mac, Linux 등) 에서도 실행되는 Native Binary로도 컴파일 될 수 있다. 
#



## 변수, 상수, 타입
```
fun main() {
    var experiencePoints: Int = 5

    // 틀린 타입으로 지정하면 compile error
//    var errorType: Int = "String" 

    // 연산 가능
    experiencePoints += 5 

    // kotlin types
    var str: String = "String"
    var c:Char = 's'
    var bb:Boolean = false
    var dd:Double = 3.14
    var llist:List<Int> = listOf(1,2,3)
    var sset:Set<Int> = setOf(1,2,3)
    var mmap:Map<Int, Int> = mapOf(0 to 0, 1 to 1)

    val readOnly: String = "can't change it"
    var changeable = "can change it" // 타입 추론, 타입 선언 생략이 가능, 컴파일러가 추론함

//    readOnly = "try to change" // compile error
    changeable = "try to change"
}
```
#


val 변수는 진정한 constant(상수)는 아니다. 다른 값을 반환하는 특별한 경우가 있다.

절대 변경하지 않는 변수는 컴파일 시점 상수 사용을 고려하자
`const val MAX_EXPERIENCE = 5000`
#


```
val player = "kkwon"
val point:Int = 5
```
위 변수 선언을 바이트코드로 살펴보면 아래처럼 타입이 추가되고, Int가 primitive 타입인 int로 변경되어 있음을 볼 수 있다.  
```
String player = "kkwon";
int point = 5
```
기본타입이 성능 및 기타 몇가지 측면에서 장점을 제공하기 때문에 컴파일시 변경해준다. 
코틀린은 내부적으로 참조 타입을 사용해서 객체지향적으로 코딩할 수 있게 해주며
컴파일시점에서 기본타입으로 변경해서 성능도 제공해 준다. 
#


## 조건문과 조건식
```
fun main() {
    val name = "kkwon"
    var point = 100
    val isBlessed = true


    // CASE 1
    if (point == 100) {
        println(name + " good!")
    } else if (point >= 90) {
        if (isBlessed) {
            println(name + " good enough and better soon")
        } else {
            println(name + "good enough anyway")
        }
    } else {
        println(name + " bad")
    }

    // 조건 표현식(conditional expression)
    val status = if (point == 100) {
        "good!"
    } else {
        "bad"
    }
    println(name + " " + status)


    // CASE 2
    val isImmortal = false

    // 논리 연산자 쓰기
    if (isBlessed && point > 90 || isImmortal) {
        println("GREEN")
    } else {
        println("NONE")
    }

    // 조건 표현식
    val auraVisible = isBlessed && point > 50 || isImmortal
    val auraColor = if (auraVisible) "GREEN" else "NONE"
    println(auraColor)


    // When 표현식 + Range
    val healthStatus = when (point) {
        100 -> "Best"
        in 90..99 -> "Almost Best"
        in 75..89 -> if (isBlessed) {
            "A little bad but better soon"
        } else {
            "A little bad"
        }
        in 15..74 -> "Bad"
        else -> "Worst"
    }

    // 문자열 템플릿
    println(
        "(Aura: $auraColor) " +
                "(Blessed: ${if (isBlessed) "YES" else "NO"})"
    )
    println("$name $healthStatus")
}
```
#


## 함수
코틀린의 기본 가시성제한자(visibility modifier)는 public이다.
#

```
private fun foramtStatus(point: Int = 2, isBleassed: Boolean): String =
    when(points)
        100 -> "Best"
        else -> "Worst"
```
매개변수에 기본값을 지정할 수도 있다.

게다가 return 생략해도 String이 리턴된다

또한 단인 표현식으로 중괄호를 안쓸 수도 있다

여기선 생략 안했지만 리턴타입(String)도 생략해도된다

리턴타입과 리턴문도 없고 단일 표현식을 쓰면 Unit 함수라고 부른다.(리턴안함) 
#


`printStatus("None", true, "kkwon", status)`

이렇게 호출하는 함수를 아래와 같이 매개변수 이름과 함께 순서를 바꿔 호출도 가능하다. 코드가 명확해진다.  

```
printStatus(status = status,
            isBlessed = true,
            name = "kwon",
            auraColor = "NONE")
```   
#

### Nothing 타입
앞서의 Unit 타입처럼 반환하지 않는데 차이점이 있다.
 
제어가 복귀되지 않는다.

```
// 내장함수 중 TODO라는게 있음
@kotlin.internal.InlineOnly
public inline fun TODO(): Nothing = throw NotImplementedError()

fun shouldReturnAString(): String {
    TODO("....")
}
``` 
이렇게 TODO를 써놓으면 Nothing이 반환되는데 이는 즉 제어가 복귀되지 않아 그냥 넘어가버리고 다음 코드가 실행되게 된다. 
(컴파일 에러 안남)
#

### 백틱 함수 이름
```
fun `**~prolly not a good idea!~**` () {
    ...
}
```
특수 문자를 사용하면 이렇게 문장처럼 쓸 수 있다. 

자바와 코틀린이 예약어가 다르기 때문에 (예를 들면 is()... 코틀린에서는 예약어지만 자바에서는 쓸 수 있음)
이름 충돌을 막기 위해 사용할 수 있다.

```
fun doSomthing() {
    `is`() // 자바의 is 메서드를 코틀린에서 호출
}
```
여튼 테스트 코드를 더 쉽게 나타내기에 좋다. 뭐.. 굳이 안써도됨 
#

## 익명 함수와 함수 타입 
```
// 코틀린 표준 라이브러리의 함수 중 하나
    val numLetters = "Mississippi".count()
    println(numLetters)

    // 특정 문자 개수를 알고 싶다면?
    val numLetters2 = "Mississippi".count { letter -> letter == 's'} // 괄호는 생략 가능
//        "Mississippi".count ({ letter -> letter == 's'})
    println(numLetters2)
```
#

### 직접 익명함수 만들어보기
```
fun main() {
    // 익명함수 정의하고 호출하기
    println({
        val currentYear = 2020
        "Welcome to Seoul! in $currentYear"
    }()) // ()를 사용해서 호출한거임

    println("################################################")

    // 익명함수도 타입을 가진다 = 함수타입
    var greetingFunction: () -> String = {
        val currentYear = 2020
        "Welcome to Seoul! in $currentYear"
    }
    // 리턴 키워드가 없어도 String이 반환된다.
    // 암시적 반환이라고 하는데 익명함수에서는 return 키워드 사용이 금지되어 있음(어떤 곳으로 복귀하는지 컴파일러가 알 수 없음)
//    var greetingFunction2: () -> String = {
//        val currentYear = 2020
//        return "Welcom to Seoul! in $currentYear" // 에러남
//    }

    println(greetingFunction())


    println("################################################")

    // 인자 사용하기
    val greetingFunction3: (String) -> String = {name ->
        val currentYear = 2020
        "Welcome to Seoul, $name! in $currentYear"
    }

    // it 키워드 사용하기 (다만 데이터가 무엇인지 알기 어려울 수 있으니 중첩된 익명함수에서는 매개변수 이름을 쓰자)
    val greetingFunction4: (String) -> String = {
        val currentYear = 2020
        "Welcome to Seoul, $it! in $currentYear"
    }
    println(greetingFunction3("kkwon"))
    println(greetingFunction4("kkwon"))

    // 다중 인자받기
    val greetingFunction5: (String, Int) -> String = { name, year ->
        "Welcome to Seoul, $name! in $year"
    }
    println(greetingFunction5("kkwon", 2020))

    // 타입추론을 이용한 생략
    val greetingFunction6 = { name: String, year: Int ->
        "Welcome to Seoul, $name! in $year"
    }
    println(greetingFunction6("kkwon", 2020))

    println("################################################")

    // 함수를 인자로 받는 함수
    runSimulation("kwon", greetingFunction6)

    // 매개변수인 함수를 구현하면서 전달해도됨
    runSimulation("kwon") {
        name, year ->
        "Welcome to Seoul, $name! in $year"
    }

    println("################################################")


}


// 함수를 인자로 받는 함수
fun runSimulation(name: String, greetingFunction: (String, Int) -> String) {
    val random = (1990..2020).shuffled().last()
    println(greetingFunction(name, random))
}
```
#

람다를 정의하면 JVM에서 객체로 생성이 됨

또한 JVM은 람다를 사용하는 모든 변수의 메모리 할당을 수행하므로 메모리가 많이 사용된다

inline을 사용하면 최적화를 할 수 있다

```
inline fun runSimulation(name: String, greetingFunction: (String, Int) -> String) {
    val random = (1990..2020).shuffled().last()
    println(greetingFunction(name, random))
} 
```
inline 키워드를 추가하면 컴파일러가 바이트코드 생성시 람다 코드를 함수 안으로 넣어버린다. 

람다를 인자로 받는 재귀함수는 코드가 무수히 많아질 수 있으니 컴파일러가 인라인 처리 하지 않고 루프 형태로 변경해버린다. 
#

함수 참조를 이용해 호출도 가능하다
```
    ...
    // 함수 참조 사용하기
    runSimulation2(
        "kwon",
        ::printDays, // 요기서 지정함
        greetingFunction6
    )
}

// 함수 참조 사용하기
inline fun runSimulation2(name: String,
                          dayPrinter: (Int) -> Unit,
                          greetingFunction: (String, Int) -> String) {
    val random = (1990..2020).shuffled().last()
    dayPrinter(random)
    println(greetingFunction(name, random))
}

// 함수 참조 사용하기
fun printDays(year: Int) {
    val dayPerYear = 365
    println("All days: ${dayPerYear * year}")
}
```
#

반환 타입으로 함수 타입을 사용할 수도 있다. 
```
    ...
    // 반환타입으로 함수 사용하기
    runSimulation3()

}

// 반환타입으로 함수 사용하기
fun runSimulation3() {
   val greetingFunction = configureGreetingFunction()
    println(greetingFunction("kwon"))
}

// 반환타입으로 함수 사용하기
fun configureGreetingFunction(): (String) -> String {
    val type = "Lunar"
    var month = 4
    return { name: String ->
        val year = 2020
        month+= 1
        "Mr.$name, Date: $year $month($type)"
    }
}
```
configureGreetingFunction에 선언된 지역변수인 type 및 month가 반환하는 람다에서 사용되었다. 

코틀린의 람다는 클로저이다. 

다른 함수에 포함된 함수에서 자신을 포함하는 함수의 매개변수와 변수를 사용할 수 있는 것을 클로저라고 한다. 

예제에서 month는 var로 선언했는데 반환하는 함수에서는 별도의 객체로 저장되어 사용된다.
예를 들어보자. 
```
// 클로저 변수 스코프 확인
fun runSimulation4() {
    val greetingFunction = configureGreetingFunction()
    println(greetingFunction("kwon"))
    println(greetingFunction("kwon"))
}

...

Mr.kwon, Date: 2020 5(Lunar)
Mr.kwon, Date: 2020 6(Lunar)
``` 
configureGreetingFunction에서 month는 4이지만 반환한 별도의 month는 따로 만들어져서 점점 증가했다. 
#

## null 안전과 예외 
자바는 구분하지 않으므로 컴파일 시점에 알려주지 못한다. 
코틀린에서는 null을 가질 수 있다는 것을 명시해줘야 한다.
```
var sth = "kwon"
sth = null // error!!
```
#
물음표를 지정하면 null이 가능하다는 의미이다.

`public fun readLine(): String?`
```
var sth = readLine().capitalize()
println(sth)
```
이대로 실행하면 컴파일 에러가 난다. 
readLine()은 null을 반환할 수가 있기 때문에 컴파일이 미리 막은 것이다.

처리하는 방법은 여러가지가 있다. 
```
// 1-1
var sth = readLine()?.capitalize()

// 1-2
var sth2 = readLine()?.let {
    if (it.isNotBlank()) {
        it.capitalize()
    } else {
        "something"
    }
}
```
let은 안전 호출 연산자 라고 표현한다.
추가 작업을 수행할 수 있다. 
#
`var sth = readLine()!!.capitalize()`

!!은 뭐가 오든 뒤에 것을 실행하라는 non-null 단언 연산자이다. 
하지만 여기서는 NPE가 발생한다.

이 단언 연산자는 컴파일러가 null 발생을 미리 알 수 없는 상황에서 쓰인다. 
!!를 쓰기 전에 이미 null 검사를 한 경우가 그 예다.
#
전통적인 if문을 쓰는 방법은 생략한다.
#
null 복합 연산자를 사용할 수도 있다.
null인 경우 지정해주는 것이다. 
```
var sth = readLine()
val nco: String = sth ?: "something"
```
#
let과 함께 null 복합 연산자를 써보겠다. : 이후에 할당하는 게 아니니 헷갈리지 말자. 
```
var sth = readLine()
sth?.let {
    sth = it.capitalize()
} ?: println("sth is null")
```
#
### 예외
코틀린도 예외를 가지고 있다.

흔히 발생하는 예외 중 IllegalStateException이 있다. 문자열을 같이 전달하여 출력할 수 있어 유용하다. 

허나 이름만 봐서는 확실히 알기 어렵다. 
여튼 사용해보자.
```
fun main() {
    var sth: Int? = null
    val is conidtion = (1..3).shuffled().last() == 3
    if (condition) {
        sth = 2
    }

    checkSth(sth)
    sth = sth!!.plus(1)
}

fun checkSth(sth: Int?) {   // 파라미터에도 null 가능을 명시해야 함
    sth ?: throw IllegalStateException("NPE...")
}
```
여러번 돌리다보면 Exception이 발생할 것이다. 
#
커스텀 예외를 정의해보자
```
...
class UnSatisfiedException() : IllegalStateException("NPE...")
```
여기서 : 은 상속 또는 구현을 뜻한다. 
이걸 위에서 사용하면 되겠다. 
#
전제조건 함수를 사용할 수도 있다. 
```
fuc checkSth(sth: Int?) {
    checkNotNull(sth, {"NPE..."})
}
```
null이면 IllegalStateException이 발생할 것이다. 

이 외에도 require, requireNotNull, error, assert가 있다. 
#

코틀린에서는 모든 예외가 unchecked 예외다. 즉 예외가 생길 수 있는 모든 코드를 try/catch로 처리하도록 강요하지 않는다는 뜻이다. 

자바의 처리는 불편함이 많다. 
처리를 전혀하지않고 넘어가는 경우도 꽤 많은데 이럴 경우 원인을 찾기가 매우 어려워 진다. 

경험적으로 checked 예외는 문제를 해결하기보다는 더 많은 문제(코드 중복, 이해하기 어려운 에러복구 로직, 기록없는 예외 무시 등)를 야기하므로 현대 언어에서는 unchecked 예외를 지원하는 것 같다. 
#

## 문자열
```
const val TAVERN_NAME = "Taernyl's Folly"
fun main() {
    placeOrder("shandy,Dragon's Breath,5.91")
}

fun placeOrder(s: String) {
    val indexOfApostrophe = TAVERN_NAME.indexOf('\'')
    val tavernMaster = TAVERN_NAME.substring(0 until indexOfApostrophe)
    println("Order to $tavernMaster")

    val data = s.split(',')
    val type = data[0]
    val name = data[1]
    val price = data[2]
    val message = "Purchase $name ($type) with $price"
    println(message)

    // 해체선언(destructuring declaration) 사용하기
    val (type2, name2, price2) = s.split(',')
    val message2 = "Purchase $name2 ($type2) with $price2"
    println(message2)
    
    val phrase = "와, $name 진짜 좋구나!"
    println("ohoh ${toDragonSpeak(phrase)}")
}
    
private fun toDragonSpeak(phrase: String) =
    phrase.replace(Regex("[aeiou]")) {
        when (it.value) {
            "a" -> "4"
            "e" -> "3"
            "i" -> "1"
            "o" -> "0"
            "u" -> "|_|"
            else -> it.value
        }
    }

```
#
```
private fun placeOrder(s: String) {
    ...
    varl phrase = if (name == "Dragon's Breath") {
        "${toDragonSpeak("와, $name 진짜 좋구나!")}"
    } else {
        "Tahnks, $name"
    }
}

```
여기서 동등 비교 연산자인 ==를 사용했다.
문자열에서 이 연산자를 사용하면 문자열의 각 문자를 같은 순서로 하나씩 비교한다. 

또 다른 방법으로는 참조 동등(referential equality) 비교가 있다. 
참조를 똑같이 갖는지 검사한다. 이떄는 === 를 사용한다. 

**자바에서는 == 연산자가 참조를 비교하므로 코틀린과 다르다.
자바에서 문자열 값을 비교할때는 equals 메서드를 사용한다.**
#

하나씩 다루기
```
"Dragon's Breath".forEach {
    println("$it\n")
}
```
#

## 숫자
문잦열을 숫자 타입으로 반환하기
```
toFloat
toDouble
toDoubleOrNull
toIntOrNull
toLong
toBigDecimal
```
아래처럼 활용하면 된다. 

`val gold: Int = "5.91".toIntOrNull() ?: 0`
#
2자리까지만 나타내기

`println("${%.2f.format(doubleVariable)}")`
#
Double을 Int로

`val intVal = doubleVal.toInt()` -> 소수점 이하값 절삭

`val intVal = (doubleVal % 1 * 100).roundToInt()` -> 소수점 남기기 
#

## 표준 함수
코틀린 라이브러리에 있는 표준 함수는 람다를 인자로 받아 동작한다.
apply, let, run, with, also, takeIf를 알아본다. 

### apply
```
var menuFile = File("menu-file.txt")
menuFile.setReadable(true)
menuFile.setWritable(true)
menuFile.setExecutable(false)

val menuFile = File("menu-file.txt").apply {
    setReadable(true)
    setWritable(true)
    setExecutable(false)
}
```
수신자 객체 반환
### let
```
// ex1
val squared = listOf(1,2,3).first().let {
    it * it
}

// ex2
fun formatGreeting(name: String?) : String {
    return name?.let {
        "$it, welcome"
    } ?: "nobody welcome"
}
```
결과를 반환
### run
```
// ex1
var menuFile = File("menu-file.txt")
val servesDragonsBreath = menuFile.run {
    readText().contains("Dragon's Breath")
}

// ex2, 함수 참조를 실행할 떄 사용 
fun nameIsLong(name: String) = name.lenth >= 20
"Madrigal".run(::nameIsLong) // return false
"Polarcubis, Supreme Master of NyetHack".run(::nameIsLong)" // return true

// ex3
fun playerCreateMessage(nameTooLong: Boolean): String {
    return if (nameTooLong) {
        "too long.."
    } else {
        "Welcome!"
    }   
}

"Polarcubis, Supreme Master of NyetHack"
    .run(::nameIsLong)
    .run(::playerCreateMessage)
    .run(::println)

``` 
결과를 반환
### with
```
val nameTooLong = with("Polarcubis, Supreme Master of NyetHack") {
    length >= 20
}
```
run이 나음
### also
```
var fileContents: List<String>
File("file.txt")
    .also {
        print(it.name)
    }.also {
        fileContents = it.readLines()
    }
```
let과 비슷한데 수신자 객체를 반환한다. 
### takeIf
```
val fileContents = File("myfile.txt")
    .takeIf { it.canRead() && it.canWrite() }
    ?. readText()
```
람다에 제공된 조건식(predicate)을 실행 한 후 true면 수신자 객체, false면 null을 반환한다. 

그 외 takeUnless도 있다. (takeIf와 반대임)
#

## List와 Set
`val patronList: List<String> = listOf("El","Mo","So")`

`val patronList = listOf("El","Mo","So")` -> 타입추론 가능 

`val patronList: List<String> = listOf("El","Mo","So", 1)` -> 에러
#
사용예제
```
list.first()
list.last()
list[1]
if (list.contains("El")) ...
if (list.containsAll(listOf("EL", "Mo"))) ...
```
list[1] 등은 인덱스가 넘을 수 있으므로 아래처럼 하는게 좋다.
```
list.getOrElse(4) {"default"}
list.getOrNull(4) ?: "default"
```
#
### List 요소 변경하기
?가 없으면 읽기 전용이다. 따라서 mutableListOf를 사용한다. 

```
val list = mutableListOf("El", "Mo", "So")
fun main() {
    list.remove("El")
    list.add("Al")
}
```
새로운 요소는 List 제일 끝에 추가되는데 특정 위치에 넣을 수도 있다.
 
`list.add(0, "Al")`

toList와 toMutableList를 사용해서 상호 변경할 수 있다. 
```
val list = mutableListOf("A", "B", "C")
val readOnlyList = list.toList()
```
#
add 등을 변경자 함수 (mutator function) 라고한다. 
* add
* addAll
* +=
  * ex) list += listOf("D","E")
* -=
* clear
* removeIf
#
List는 iteration기능이 있다.

Set, Map, IntRange 등도 가능하다.   
```
for (element in list) {
    ...
}

list.forEach { e ->
    ...
}

// 인덱스 쓰고 싶으면
list.forEachIndexed { index, e -> 
    ...
}
```
### Set
```
val planets = setOf("Mercury", "Venus", "Earth")

val planets2 = mutableSetOf("Mercury", "Venus", "Earth")
```
#
toSet() 및 toList() 를 활용하면 컬렉션 변환을 쉽게 할 수 있다.

`val list = listOf("A","B","C","C","D").toSet().toList()`

List에는 distinct() 함수가 있는데 얘가 내부적으로 toSet()과 toList()를 호출한 것이다. 
#
### 배열타입
코틀린에서는 기본타입이 아닌 Array를 지원한다. 
물론 앞에서 설명했듯이 바이트코드로 생성될 때는 기본 타입으로 컴파일 된다.

`val ages: IntArray = intArrayOf(1,2,3,4,5)`

코틀린 변환 함수들을 사용하면 자바의 기본 배열 타입으로 변환할 수 있다. 

```
val ages: List<Int> = listOf(1,2,3,4,5)
ages.toIntArray()
``` 
* IntArray
* DoubleArray
* BooleanArray
...
#
```
val x = listOf(mutableListOf(1,2,3))
val y = listOf(mutableListOf(1,2,3))
x[0].add(4)
x == y // flase
```
변경불가한 list의 요소로 변경가능한 list를 가지만 위와 같이 변경이 된다. 
```
var list: List<Int> = listOf(1,2,3)
(list as MutableList) list[2] = 1000
```
as 키워드를 사용하면 또한 변경 가능하다. 

불변성을 강요하지는 않는다. 우리가 적절히 사용할 수 있을 것. 
#
## Map
mapOf, mutableMapOf

```
val mapEx = mapOf("A" to 10.5, "B" to 8.0)

val mapEx2 = mapOf(Pair("A", 10.5), Pair("B", 8.0))

mapEx += "C" to 6.0

val aVal = mapExp["A"]

val dVal = mapExp.getOrElse("D") { 1.1 }

val eVal = mapExp.getOrDefault("E", 0.0)

val mutableMap = mutableMapOf<String, Double>()
mapEx.forEach {
    mutableMap[it] = 6.0
}

mutableMap["A"] = 11.0 - 5.5

mapEx.forEach { k, v ->
    println("$k, value: ${"%.2f".format(v)}") 
}
```    













  







 


  