## 클래스 정의하기
```
class Player {
    fun castFireball(numFireblls: Int = 2) = 
        println("throw firballs (x$numFirballs)") 
}
```
```
fun main() {
    val player = Player()
    player.castFireball()
} 
```
### 가시성(visibility) & 캡슐화
* public
* private
* protected
* internal: 함수/속성이 정의된 클래스가 포함된 모듈에서 가능

자바의 패키지 가시성은 코틀린에 없다. 
#
### 클래스 속성
다음 코드는 에러가 난다. 
```
class Player {
    var name: String
}
```
변수와 달리 클래스 속성은 반드시 초기값이 지정되야 한다. 
#
속성에 대한 getter는 자동 생성 된다.

setter는 경우에 따라 다르다. var로 선언된 경우 자동 생성 된다. 

getter/setter는 overriding해서 새로 정의 할 수 있다. 
```
class Player {
    var name = "kkwon" // setter 추가하려면 val이 아닌 var로 되어야 한다 
        get() = field.capitalize()
        set(value) {
            field = value.trim()
        }

    ...
}
``` 
여기서 field 키워드는 코틀린이 관리하는 backing field를 참조한다. getter/setter에서만 사용된다. 
```
val player = Player()
player.name = "park   "
println(player.name + "TheBrave")
```
속성을 참조하면 자동으로 getter/setter가 호출된다. 
#
만약 속성은 public인데 setter는 숨기고 싶으면 private만 추가해주면 된다. 
```
private set(value) {
```
#
산출 속성(computed property) 이란게 있다.
```
class Dice() {
    val rolledValue 
        get() = (1..6).shuffled().first()
}
```
자신의 독자적인 값을 갖지 않기 때문에 backing field가 생성되지 않는다.
#
지금까지 해온 소스들을 클래스로 옮겨본다. 
```
package com.kkwonsy.nyethack

class Player {

    var name = "madrigal"
        get() = field.capitalize()
        private set(value) {
            field = value.trim()
        }

    var healthPoints = 89
    val isBlessed = true
    private val isImmortal = false

    fun auraColor(): String {
        val auraVisible = isBlessed && healthPoints > 50 || isImmotal
        val auraColor = if (auraVisible) "GREEN" else "NONE"
        return auraColor
    }

    fun formatHealthStatus() =
        when (healthPoints) {
            100 -> "최상"
            in 90..99 -> "약간 찰과상"
            in 75..89 -> if (isBlessed) {
                "경미한 상처, 빠른 치유중"
            } else {
                "경미한 상처"
            }
            in 15..74 -> "많이 다침"
            else -> "최악"
        }

    fun castFireball(numFireballs: Int = 2) = 
        println("A fireball shows up. (x$numFireballs)")
}
```
#

서로 다른 패키지에 있는 같은 이름의 클래스나 함수를 사용하다보면 이름 충돌이 생길 수 있다. 

as 키워드를 사용하면 좋다. 
```
import com.io.extractValue
import com.util.extractValue as extractValueUtil
...

val value1 = extractValue()
val value2 = extractValueUtil()
``` 
#
var 과 val 내부 구현?
```
class Student(var name: String)

...
public final class Student {
    @NotNull
    private String name;

    @NotNull
    public final String getName() ...

    public final void setName(@NotNull String var1) ...

    public Stuent(@NotNull String name) ...
}
```
```
class Student(val name: String)

...
public final class Student {
    @NotNull
    private String name;

    @NotNull
    public final String getName() ...

    public Student(@NotNull String name) ...
}
```
val로 선언하니 setter가 없다. 

산출속성은?
```
Class Student {
    val name: String 
        get() = "Madrigal"
}
...

public final class Student {
    @NotNull
    public final String getName() ...
}
```
getter밖에 없다. 
#
### 경합 상태 방지하기
다음코드는 에러가 난다. 
```
class Weapon(val name: String)
class Player {
    var weapon: Weapon? = Wepon("Ebony Kris")
    
    fun printWeaponName() {
        if (weapon != null) {
            println(weapon.name) // 여기서 에러
        }
    }
}

fun main() {
    Player().printWeaponName()
}
```
분명 null이 아님을 확인한 후 실행을 했으나 에러가 발생한다. 

if 문에서 null 확인을 하면 코틀린 컴파일러는 null이 올 수 없는 Weapon 을 임시 변환해준다. (=smart casting)

근데 왜 에러가 날까?

경합상태(race condition)을 발생할 가능성이 있음을 코틀린 컴파일러가 알고 있기 때문이다. 
쉬운 말로 null 체크 후에 **weapon = null** 코드가 들어올 수 있다는 것을 컴파일러가 체크해 준다.

다음 코드로 변경해주면 잘 돌아간다.  
```
fun printWeaponName() {
    weapon?.also {
        println(it.name)
    }
}
```
it은 코드로 변경할 수 없기 때문에 컴파일러가 에러를 주지 않는다.
#

## 초기화 
### 생성자
위 글에서는 속성들을 바로 초기화 했었으나 인스턴스 생성시 값을 변경할 방법이 없다. 

primary constructor가 필요하다. 
```
class Player {
    var name = "madrigal"
        get() = field.capitalize()
        private set(value) {
            field = value.trim()
        }

    var healthPoints = 89
    val isBlessed = true
    private val isImmortal = false
...



class Player(_name: String,
             var healthPoints: Int = 100,
             val isBlessed: Boolean,
             private val isImmortal: Boolean) {
    var name = _name
        get() = field.capitalize()
        private set(value) {
            field = value.trim()
        }
```
보조생성자도 만들 수 있다. 
```
    constructor(name: String) : this(name,
        isBlessed = true,
        isImmortal = false) {
        if (name.toLowerCase() == "kar") healthPoints = 40
    }
``` 
기본인자가 많아지면 혼란스러우니 지명인자(named argument)를 사용할 수 있다. 
```
val player = Player(name = "kwon",
               healthPoints = 100,
               isBleassd = true,
               isImmortal = false)
```
### initializer block
속성 값의 검사 등을 위해 초기화 블록을 사용할 수 있다.

인스턴스가 생성될 때 마다 자동으로 호출된다.  
```
    init {
        require(healthPoints > 0, { "healthPoints는 0보다 커야 합니다." })
        require(name.isNotBlank(), { "플레이어는 이름이 있어야 합니다." })
    }
```  
#
속성을 하나 추가하자. 

`val hometown: String`

이렇게만 하면 초기화를 안해서 에러가 발생한다. 

`val hometown = ""`

에러는 없지만 좋은 해결책은 아니다. 함수를 통해 초기값을 지정해보자. 
```
var name = _name
        get() = "${field.capitalize()} of $hometown"

val hometown = selectHometown()

private fun selectHometown() = File("data/towns.txt")
        .readText()
        .split("\r\n") // 맥 OS나 리눅스에서는 .split("\n")
        .shuffled()
        .first()
``` 
name에서 hometown을 이용해 getter를 정의하게 했다.

초기화가 보장은 되므로 컴파일 에러가 생기지 않는다.  
#
다른 객체를 참조하는 속성의 경우 선언하는 시점에서 초기화 될 수 없는 경우가 있다. 

생성자가 호출되는 방법이나 시점을 우리가 제어 할 수 없는 경우, 예를 들면 외부 프레임워크에서 초기화 되는 경우가 그렇다.  

안드로이드에서 Activity는 각종 컴퓨넌트 클래스들의 인스턴스를 참조하는 속성을 갖는다. 

애플리케이션이 실행되면 Activity 클래스의 인스턴스가 자동 생성되고 onCreate 함수가 자동 호출된다. 

이때 모든 속성을 초기화 되어야 한다. 

생성 시점에 모든 속성이 초기화 될 수 없는데 이때 지연 초기화를 해야 한다. 
```
class Wheel {
    lateinit var alignment: String
    ...
}
```
초기화를 안해서 컴파일 에러가 나지 않으므로 조심해서 사용해야 한다. 

초기화 전에 사용되어버리면 UninitializedPropertyAccessException이 발생된다. 
#
초기화 지연에는 다른 방법이 있다. 
lazy initialization이다. 

```
val hometown by lazy { selectHometown() }
```
hometown은 파일에서 읽은 데이터로 초기화 되므로 필요한 시점에서 초기화 되게 해주는게 효울적이다. 

다른코드에서 최초 사용시에 lazy 함스의 람다가 실행되어 초기화 된다. 

또한 한번만 실행 된다. 이후에는 캐시에 저장된 결과가 사용된다. 
#

## 상속
```
open class Room(val name: String) {
    protected open val dangerLevel = 5

    fun description() = "Room: $name\r\n" + // 맥 OS나 리눅스에서는 "Room: $name\n"
            "위험 수준: $dangerLevel"

    open fun load() = "아무도 여기에 오지 않았습니다..."
}

class TownSquare : Room("Town Square") {
    override val dangerLevel = super.dangerLevel - 3
    private var bellSound = "댕댕"

    final override fun load() = "당신의 참여를 주민들이 다 함께 환영합니다!\r\n${ringBell()}"

    private fun ringBell() = "당신의 도착을 종탑에서 알립니다. $bellSound"
}
```
Room 클래스가 서브 클래스를 가질 수 있으려면 `open` 키워드가 필요하다. 

또한 overriding 할 수 있으려면 함수에도 `open` 키워드가 필요하다. 

코틀린에서는 속성도 overriding 할 수 있고 `open` 키워드가 필요하다.

만약 TownSqure의 서브 클래스는 생성할 수 있게 하고 load 함수는 overriding 못하게 하려면?
```
open class TownSquare : Room("Town Square") {
    ...
    final override fun load() ...
}
```
`final` 키워드를 사용할 수 있다.
#
타입 검사는 `is` 연산자를 통해 하면 된다. 
```
var room = Room("Foyer")
room is Room // true
room is TownSquare // false

var townSquare = TownSquare()
townSquare is Room // true
townSquare is TownSquare // true
```
#
코틀린에서 최상위 슈퍼 클래스는 `Any` 이다
```
fun sth(any: Any) {
    val isSth = if (any is Player) {
        any.isBlessed // smart casting!!
    } else {
        false
    }
}
```
#
Any 클래스는 equals, hashCode, toString을 정의하고 있다. 

Any? null 가능 타입이 있다. 

모든 null 가능 타입은 Any? 의 서브 타입이다. 

모든 null 불가능 타입은 Any 의 서브 타입이다.

Number 는 Number?의 서브 타입이다. 

Any 는 Any? 의 서브 타입이다. 
#

## 객체
nested class, data class, enum class 등을 알아본다. 
#
singleton 객체가 필요할 떄가 있는데 `object` 키워드를 사용하면 된다. 

방법은 3가지가 있다. 

* 객체 선언 (object declaration)
* 객체 표현식 (object expression)
* 동반 객체(companion object)
#
### 객체 선언
```
object Game {
    init {
        println("방문을 환영합니다.")   
    }

    fun play() {
        while (true) {
            // TODO
        }
    }
}

fun main() {
    Game.play()
}
```
#    
### 객체 표현식
대개의 경우는 class 키워드를 사용해서 이름이 있는 클래스를 별도의 파일에 정의한다. 

기존 클래스의 서브 클래스를 이름 없이 정의한 후 바로 인스턴스를 생성해서 사용하면 편리할 때가 있다. (anonymous class)
```
val abandonedTownSquare = object : TownSquare() {
    override fun load() = "No one here..."
}
```
TownSquare의 서브 클래스를 객체 표현식을 통해 익명 클래스로 만든 것이다.

이 인스턴스는 위의 코드가 실행될 때 생성과 초기화 되며 val에 저장했으므로 싱글톤 객체가 된다. 
물론 abandonedTownSquare 변수가 존재하는 동안만이다.
#
### 동반 객체
클래스 내부에 정의해서 사용한다. 

`object` 키워드 앞에 `companion` 키워드를 추가하면 된다. 

```
class PremadeWorldMap {
    ...
    companion object {
        private const val MAPS_FILEPATH = "nyethack.maps"

        fun load() = File(MAPS_FILEPATH).readBytes()
    }
}
``` 
PremadeWorldMap은 동반 객체를 가지고 있다. 

PremadeWorldMap의 인스턴스 생성 없이 load 함수를 호출할 수 있다. 
```
PremadeWorldMap.load()
```
#
### 중첩 클래스
클래스 내부에 정의되는 모든 클래스가 이름이 없진 않다. nested 클래스에도 class 키워드를 사용해 이름을 넣을 수 있다. 
```
object Game {
...
    fun play() {
        while (true) {
            ...
            print("> 명령을 입력하세요: ")
            println(GameInput(readLine()).processCommand())
        }
    }

    private class GameInput(arg: String?) {
        private val input = arg ?: ""
        val command = input.split(" ")[0]
        val argument = input.split(" ").getOrElse(1, { "" })
    
        fun processCommand() = when (command.toLowerCase()) {
            "move" -> move(argument)
            else -> commandNotFound()
        }
    
        private fun commandNotFound() = "적합하지 않은 명령입니다!"
    }
}
```
외곽 클래스의 인스턴스가 생성되어야 사용할 수 있다. 
#
### 데이터 클래스
데이터를 저장하기 위해 특별히 설계된 클래스이다. 
강력한 데이터 처리 기능을 갖고 있다. 

`data` 키워드를 class 앞에 추가하면 된다. (data class는 슈퍼 클래스가 될 수 없다) 
```
data class Coordinate(val x: Int, val y: Int) {
    val isInBounds = x >= 0 && y >= 0
}
```
앞서 모든 클래스는 Any 클래스로부터 상속 받는다고 했다. 그리고 Any는 `toString, equals, hashCode` 함수가 있다. 

data class를 정의하면 코틀린 컴파일러가 이 함수들을 자동으로 생성해준다. `copy` 함수도 해준다.

toString은 이쁘게 출력하게 해주고, equals는 속성값을 비교하도록 구현해주며, copy는 속성과 값만 인자로 전달하여 복사하게 해준다. 
#
### enum 클래스
```
enum class Direction(private val coordinate: Coordinate) {
    NORTH(Coordinate(0, -1)),
    EAST(Coordinate(1, 0)),
    SOUTH(Coordinate(0, 1)),
    WEST(Coordinate(-1, 0)); // 제일 끝에 세미콜론을 추가한다
    fun updateCoordinate(playerCoordinate: Coordinate) =
        coordinate + playerCoordinate
}
```
#
### 연산자 오버로딩

`a+b`를 코틀린 컴파일러는 `a.plus(b)`로 실행하도록 바이트 코드를 생성한다. 
다른것들도 마찬가지다. 

Int가 아닌 다른 타입, 예를 들면 Coordinate를 + 연산자를 이용해보도록 하자. 
```
data class Coordinate(val x: Int, val y: Int) {
    val isInBounds = x >= 0 && y >= 0

    operator fun plus(other: Coordinate) = Coordinate(x + other.x, y + other.y)
}
```
`operator` 키워드를 사용하면 된다. 
* plus // +
* plusAssign // +=
* equals // ==
* compareTo // >
* get // []
* rangeTo // ..
* contains // in
#
### ADT (Algebraic data type, 데이적 데이터 타입)
지정된 타입과 연관될 수 있는 서브 타입들의 폐집합을 나타낸다. enum도 ADT의 간단한 형태다. 

```
enum class StudentStatus {
    NOT_ENROLLED,
    ACTIVE,
    GRADUATED
}

class Student(var status: StudentStatus)

fun main() {
    val student = Student(StudentStatus.NOT_ENROLLED)
}

fun studentMessage(status: StudentStatus): String {
    return when (status) {
        StudentStatus.NOT_ENROLLED -> "Please choose a course"
    }
}
``` 
위에서 NOT_ENROLLED만 처리하게 되면 컴파일 에러가 난다. 나머지를 처리안했기 때문이다. 

각 항목 나름의 더 복잡한 처리가 필요한 경우는 각 항목을 클래스로 정의해서 `sealed` 클래스로 묶어서 사용할 수 있다.
```
enum class StudentStatus {
    NOT_ENROLLED,
    ACTIVE,
    GRADUATED;
    var courseId: String?= null // ACTIVE에서만 사용되는 속성...
}
``` 
```
sealed class StudentStatus {
    object NotEnrolled : StudentStatus()
    class Active(val courseId: String) : StudentStatus()
    object Graduated : StudentStatus()
}
...
val active = StudentStatus.Active("Kotlin")
```
NotEnrolled랑 Graduated는 하나만 필요하므로 object로 선언했고, Active는 학생에 따라 여러 과정이 있을 수 있어 클래스로 선언했다. 
#
## 인터스페이스와 추상 클래스
### 인터페이
```
interface Fightable {
    var healthPoints: Int
    val diceCount: Int
    val diceSides: Int
    val damageRoll: Int
        get() = (0 until diceCount).map {
            Random().nextInt(diceSides) + 1
        }.sum()
    fun attack(opponent: Fightable): Int
}
```
속성은 초기화 코드가 없고, 함수는 몸체가 없다. 
기본 구현 getter도 추가했다. 
```
class Player(_name: String,
         override var healthPoints: Int = 100,
         val isBlessed: Boolean,
         private val isImmortal: Boolean) : Fightable {
         ...
    override val diceCount = 3
    
    override val diceSides = 6

    override fun attack(opponent: Fightable): Int {
        val damageDealt = if (isBlessed) {
            damageRoll * 2
        } else {
            damageRoll
        }
        opponent.healthPoints -= damageDealt
        return damageDealt
    }
}
```
Fightable을 구현하고 healthPoints는 기본 생성자에서 override 받았다. 

참고로 인터페이스는 `open` 키워드가 필요없다.

인터페이스는 how가 아닌 what을 구현하는데 초점을 맞춘다.   

인터페이스를 구현하는 클래스에서 how를 다룬다. 
#
### 추상 클래스
인터페이스와 비슷하게 헤더만 정의된 추상 함수와 속성을 갖는다.

하지만 몸체가 있는 일반 함수도 가질 수 있다. (하지만 인스턴스 생성은 못함, 서브 클래스에서 상속 받아 구현하도록 하는것이 목적이므로)
```
abstract class Monster(val name: String,
                       val description: String,
                       override var healthPoints: Int) : Fightable {
    override fun attack(opponent: Fightable): Int {
        val damageDealt = damageRoll
        opponent.healthPoints -= damageDealt
        return damageDealt
    }
}

class Goblin(name: String = "Goblin",
             description: String = "추하게 생긴 고블린",
             healthPoints: Int = 30) : Monster(name, description, healthPoints) {
    override val diceCount = 2
    override val diceSides = 8
}
```
Fightable 인터페이스를 구현하길 바라는 추상 클래스 Monster와 그 서브 클래스인 Goblin이다. 

추상클래스에서는 인터페이스의 것들을 모두 구현할 필요가 없다. 서브 클래스가 못다한 구현을 해줘야 한다. 
#
인터페이스와 추상 클래스의 차이는 무엇인가?

인터페이스는 생성자를 지정할 수 없다. 

일반 클래스들은 하나의 추상 클래스만 슈퍼 클래스로 가진다. 하지만 인터페이스는 여러 개를 구현할 수 있다. 

즉, *개념적으로 상속 관계가 없으면서 공통적인 속성이나 행동을 갖는다면 인터페이스를*.

*상속 관계가 있으면서 인스턴스 생성이 필요 없는 슈퍼 클래스가 필요하면 추상 클래스가* 적합하다.

단 인스턴스 생성이 필요한 슈퍼 클래스라면 추상 클래스가 아닌 일반 클래스를 쓰면 된다. 
#
## 제네릭
```
class LootBox<T : Loot>(vararg item: T) {
    var open = false
    private var loot: Array<out T> = item

    operator fun get(index: Int): T? = loot[index].takeIf { open }

    fun fetch(item: Int): T? {
        return loot[item].takeIf { open }
    }

    fun <R> fetch(item: Int, lootModFunction: (T) -> R): R? {
        return lootModFunction(loot[item]).takeIf { open }
    }
}

open class Loot(val value: Int)

class Fedora(val name: String, value: Int) : Loot(value)

class Coin(value: Int) : Loot(value)

fun main(args: Array<String>) {
    val lootBoxOne: LootBox<Fedora> = LootBox(Fedora("평범한 중절모", 15), // 제일 끝에 쉼표 추가
                                        Fedora("눈부신 자주색 중절모", 25))
    val lootBoxTwo: LootBox<Coin> = LootBox(Coin(15))

    lootBoxOne.open = true

    lootBoxOne.fetch(1)?.run {
        println("$name 를 LootBox에서 꺼냈습니다!")
    }

    val coin = lootBoxOne.fetch(0) {
        Coin(it.value * 3)
    }
    coin?.let { println(it.value) }

    val fedora = lootBoxOne[1]
    fedora?.let { println(it.name) }
}
```
`val coint = lootBoxOne.fetch` 함수를 잘 살펴보자. 

lootModFunction 이라는 함수를 매개변수의 인자로 받는다. 

그리고 lootModFunction은 T 타입의 인자를 받아서 R 타입으로 반환해준다. 

`(T) -> R` 이것을 함수 타입(function type) 이라고 한다.

lootBoxOne은 Fedora 타입의 인스턴스를 참조하지만 반환 타입이 Coin이 된다.

#
표준화된 명칭
* E: Entity
* K: Key
* V: Value
* N: Number
* T: Type
* R: Return
#
`operator fun get(...` 을 보자. 

인덱스 연산자인 []를 사용하고자 oprator 를 구현했다. 
#
LootBox 인스턴스에 여러개의 Loot 인스턴스를 저장하려면?

`vararg` 키워드를 사용하면 된다. 

`out` 키워드는 T 타입을 포함해서 서브 타입도 인자가 될 수 있다는 것을 뜻한다. 
#
```
class Barrel<T>(var item: T)

fun main() {
    var fedoraBarrel: Barrel<Fedora> = Barrel(Fedora("중절모", 15))
    var lootBarrel: Barrel<Loot> = Barrel(Coin(15))

    lootBarrel = fedoraBarrel
}
```
이 경우는 에러가 난다. 

Loot 타입이므로 그 서브인 Fedora가 될 것 같은데 안된다. 

```
var loot: Loot = Fedora("중절모", 15)
```
이건 가능하다. 

Law 타입으로 했기 때문이다. 

lootBarrel의 타입은 Barrel<Loot>이고 fedoraBarrel 타입은 Barrel<Fedora> 이다. 

따라서 Law 타입이 아니기 때문에 둘은 서로 다른 타입으로 간주된다. 

즉, 제네릭 클래스의 인스턴스는 원시 타입과 제네릭 타입이 결합된 것이 자신의 타입이 되므로 슈퍼-서브 관계가 있다 하더라도 컴파일러가 인식하지 못한다. 

따라서 이 문제를 해결하기 위한 키워드가 `in/out` 이다.

```
class Barrel<out T>(val item: T)
```
요렇게 변경하면 컴파일러가 에러로 잡아주지 않는다. 
```
fun main() {
    var fedoraBarrel: Barrel<Fedora> = Barrel(Fedora("중절모", 15))
    var lootBarrel: Barrel<Loot> = Barrel(Coin(15))

    lootBarrel = fedoraBarrel // OK!
    val myFedora: Fedora = lootBarrel.item // ERROR!
}
```
아래줄은 에러가 난다. 왜 그럴까? Barrel 클래스의 타입 매개 변수에 `out`을 지정했기 때문이다.

이 키워드를 지정하면 컴파일러가 슈퍼-서브 관계를 고려해준다. (타입이 Fedora로 바뀐게 아님)

`val` 이므로 읽기만 가능해 타입이 변경되지 않으므로 안전하기 때문이다.  

`in` 키워드는 `out`과 정반대이다. 
#
```
val list = listOf(1,2)
if (list is List<String>) {
    println("List<String>!!)
}
```
이 코드는 에러가 난다. JVM 으로 생성될 때는 제네릭 타입은 소거되므로 정확한 검사를 할 수 없기 때문이다. 

이런 문제를 해결해주기 위해 `inline`과 `reified` 키워드를 사용한다.
```
inline fun <reified T> myFunc(backupLoot: () -> T): T {
    ...
}
``` 
#
## 확장
```
val String.numVowels
    get() = count { "aeiouy".contains(it) }

fun String.addEnthusiasm(amount: Int = 1) = this + "!".repeat(amount)

fun <T> T.easyPrint() : T {
    println(this)
    return this
}

infix fun String?.printWithDefault(default: String) = print(this ?: default)

fun main(args: Array<String>) {
    "마드리갈이 그 건물에서 나왔습니다".easyPrint().addEnthusiasm().easyPrint()
    42.easyPrint()
    "How many vowels?".numVowels.easyPrint()

    val nullableString: String? = null
    nullableString printWithDefault "기본 문자열"
}
```
#
## Functional programming
#
## With Java
@nullable

@NotNull

@JvmOverloads

@JvmField

@Throws

등을 잘 파악하고 있어야 상호 운용을 할 수 있다  




















 





 





