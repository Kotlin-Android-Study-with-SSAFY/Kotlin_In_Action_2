# object 키워드: 클래스 선언과 인스턴스 생성

> Kotlin in action (p.181 ~ p.194)

object 키워드 사용 상황

- 객체 선언은 싱클턴을 정의하는 방법 중 하나이다.
- 동반 객체는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다. 동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.

<aside>
💡

**동반 객체**

코들린에서 클래스 내부에 정의되는 정적 멤버를 포함하는 객체이다. (자바에서는 static 키워드를 사용해서 클래스에 속한 정적 메소드 및 필드를 만든다.)

</aside>

| 비교 항목 | 자바 static | 코틀린 companion object |
| --- | --- | --- |
| 접근 방식 | 클래스명.정적멤버 | 클래스명.동반객체멤버 |
| 객체 생성 여부 | 없음(메모리에 고정됨) | 실제 객체(싱글톤)로 존재 |
| 상속 가능 여부 | 불가능 | 가능(interface 구현 가능) |
| 초기화 방식 | 클래스 로드 시 초기화 | 프로그램 실행 중 동적으로 초기화 가능(lateinit)  |
- 객체 식은 자바의 무명 내부 클래스 대신 쓰인다.

```kotlin
interface Printer{
	fun print();
}

fun main(){
	val printer=object:Printer{//객체 식
		override fun print(){
			println("Hello")
		}
	}
	
	Printer.print()
}
```

**객체 선언: 싱글턴을 쉽게 만들기**

코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다. 

객체 선언= 클래스 선언 + 그 클래스에 속한 단일 인스턴스의 선언 

```kotlin
object Payroll{ //객체 선언 
	val allEmployees= arrayListOf<Person> ()
	
	fun calculateSalary(){
		for (person in allEmployees){
			...
		}
	}
}
```

객체 선언 안에 프로퍼티, 메소드, 초기화 블록 등이 들어갈 수 있다.

하지만 생성자는 객체 선언에 쓸 수 없다. → 일반 클래스 인스턴스와 달리 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다. 따라서 객체 선언에는 생성자 정의가 필요 없다. 

```kotlin
Payroll.allEmployees.add(Person(...))
Payroll.calculateSalary()
```

<객체 선언을 사용해 Comparator 구현>

```kotlin
object CaseInsenesitiveFileComparator: Comparator<File>{
	override fun compare(file1: File, file2: File):Int{
		return file1.path.compareTo(file2.path, ignoreCase=true)//대소문자 무시 
	}
}

>>> println(CaseInsensitiveFileComparator.compare(File("/User"),File("user")))
```

Comparator 구현은 두 객체를 인자로 받아 그중 어느 객체가 더 큰지 알려주는 정수를 반환한다. Comparator 안에는 데이터를 저장할 필요가 없다. → 어떤 클래스에 속한 객체를 비교할 떄 사용하는 Comparator는 보통 클래스마다 하나씩만 있으면 되기 때문에 Comparator 인스턴스를 방식으로는 객체 선언이 가장 좋은 방식이다. 

> **Quiz!
대규모 소프트웨어 시스템에서는 객체 선언이 항상 적합하지 않다. 의존 관계가 별로 많지 않은 소규모 소프트웨어에서는 싱글턴이나 객체 선언이 유용하지만, 시스템을 구현하는 다양한 구성요소와 상호작용하는 대규모 컨포넌트에는 싱글턴이 적합하지 않다. 그 이유가 무엇일까요?**
> 

**코틀린 객체를 자바에서 사용하기** 

코틀린 객체 선언은 유일한 인스턴스에 대한 정적 필드가 있는 자바 클래스로 컴파일된다. 이때 인스턴스 필드의 이름은 항상 INSTANCE이다. 

자바 코드에서 코틀린 싱글턴 객체를 사용하려면 정적인 INSTANCE 필드를 통하면 된다. 

```kotlin
CaseInsensitiveFileComparator.INSTANCE.compare(file1, file2);
```

**동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소**

❗️ 코틀린 클래스 안에는 정적인 멤버가 없다. 코틀린 언어는 자바 static 키워드를 지원하지 않는다. 그 대신 코틀린에서는 패키지 수준의 최상위 함수와 객체 선언을 활용한다. 

BUT! 최상위 함수는 private로 표시된 클래스 비공개 멤버에 접근할 수 없다.

![스크린샷 2025-02-26 오후 9.21.44.png](attachment:305b6d4a-4e82-4a3b-a957-314c22f37c2b:스크린샷_2025-02-26_오후_9.21.44.png)

그래서 클래스의 인스턴스와 관계 없이 호출해야 하지만, 클래스 내부 정보에 접근해야 하는 함수가 필요할 때는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다. → 대표적인 예로 팩토리 메소드 (객체를 생성하는 메서드. 메서드를 통해 인스턴스 생성)

```kotlin
class Animal private constructor(val name: String){//생성자를 private로 숨김
	companion object{
		fun createDog():Animal{ //팩토리 메소드
			return Animal("Dog")
		}
		fun createCat():Animal{//팩토리 메소드 
			return Animal("Cat")
		}
	}
}

fun main(){
	val dog=Animal.createDog()
	val cat=Animal.createCat()
	
	println(dog.name)
	println(cat.name)
}
```

클래스 안에 정의된 객체 중 하나에 companion이라는 특별한 표시를 붙이면 그 클래스의 동반 객체로 만들 수 있다. 동반 객체의 프로퍼티나 메소드에 접근하려면 그 동반 객체가 정의된 클래스 이름을 사용한다. 

→ 동반 객체는 private 생성자를 호출하기 좋은 위치이다. 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있기 때문!!! 따라서 동반 객체는 바깥쪽 클래스의 private 생성자도 호출할 수 있다. 

아래와 같은 부 생성자가 여럿 있는 클래스를 정의하는 것보다 ….

```kotlin
class User{
	val nickname:String
	constructor(email:String){
		nickname=email.substringBefire('@')
	}
	
	constructor(facebookAccountId:Int){
		nickname=getFacebookName(facebookAccountId)
	}
}
```

클래스의 인스턴스를 생성하는 팩토리 메소드가 더 유용하다. 

```kotlin
class User private constructor(val nickname:String){//주 생성자를 비공개로 만든다.

//생성자를 통해 User 인스턴스를 만들 수 없고 팩토리 메소드를 통해서 접근 
	companion object{ //동반 객체를 선언한다. 
		fun newSubscribingUser(email:String)=User(email.substringBefore('@')) //팩토리 메소드
		fun newFaceBookUser(accountId:Int)= User(getFaceBookName(accountId)) //팩토리 메소드 
	}
}

//클래스 이름을 사용해 그 클래스에 속한 동반 객체의 메소드를 호출할 수 있다. 
>>> val subscribingUser= User.newSubscribingUser("jiwon@gmail.com")
>>> val facebookUser= User.newFacebookUser(4)
>>> println(subscribingUser.nickname)
```

- 목적에 따라 팩토리 메소드 이름을 정할 수 있다.
- 팩토리 메소드는 그 팩토리 메소드가 선언된 클래스의 하위 클래스 객체를 반환할 수도 있다.
- 생성할 필요가 없는 객체를 생성하지 않을 수 있다. (캐싱)

**동반 객체를 일반 객체처럼 사용**

동반 객체는 클래스 안에 정의된 일반 객체이다. 따라서 동반 객체에 이름을 붙이거나, 동반 객체가 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다. 

```kotlin
class Person(val name:String){
	companion object Loader{//동반 객체에 이름을 붙인다. 
		fun fromJSON(jspnText:String):Person=...
	}
}

>>> person=Person.Loader.fromJSON("{name: 'Dmitry'}")
>>> person.name
Dmitry

>>> person2=Person.fromJSON("{name:'Brent'}")
>>> person2.name
Brent
```

동반 객체에 특별히 이름을 지정하지 않으면 동반 객체의 이름은 자동으로 Companion이 된다. 

**동반 객체에서 인터페이스 구현**

인터페이스를 구현하는 동반 객체를 참조할 때 객체를 둘러싼 클래스의 이름을 바로 사용할 수 있다. 

시스템에 Person을 포함한 다양한 타입의 객체가 있다고 가정하자! 이 시스템에서는 모든 객체를 역직렬화를 통해 만들어야 하기 때문에 모든 타입의 객체를 생성하는 일반적인 방법이 필요하다. → JSOM을 역직렬화 하는 JSONFactory 인터페이스가 존재한다.

```kotlin
interface JSONFactory<T>{
	fun fromJSON(jsonText: String): T
}

class Person(val name: String){
	companion object: JSONFactory<Person>{
		override fun fromJSON(jsonText: String): Pserson=... //동반 객체가 인터페이스를 구현한다. 
	}
}
```

이제 JSON으로부터 각 원소를 다시 만들어내는 추상 팩토리가 있다면 Person 객체를 그 팩토리에 넘길 수 있다. 

```kotlin
fun loadFromJSON<T> (factory: JSONFactory<T>:T{...}

loadFromJSON(Person) //동반 객체의 인스턴스를 함수에 넘긴다. 
```

❗️동반 객체가 구현한 JSONFactory의 인스턴스를 넘길 때 Person 클래스의 이름을 사용했다는 점 주의

> **QUIZ!**
클래스의 동반 객체는 일반 객체와 비슷한 방식으로, 클래스에 정의된 인스턴스를 가리키는 정적 필드로 컴파일된다. 동반 객체에 이름을 붙이지 않았다면 자바 쪽에서는 어떤 이름으로 그 참조에 접근할 수 있을까?
> 

**동반 객체 확장**

클래스에 동반 객체가 있다며 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다. 

예를 들어 c라는 클래스 안에 동반 객체가 있고 그 동반 객체 안에 func을 정의하면 외부에서는 func()를 c.func()로 호출할 수 있다. 

```kotlin
//비지니스 로직 모듈
class Person(val firstName: String, val lastName:String){
	companion object{} //비어있는 동반 객체를 선언한다. 
}

//클라이언트/서버 통신 모듈
fun Person.Companion.fromJSON(json: String): Person{...} //확장함수 

val p=Person.fromJSON(json)
```

마치 동반 객체 안에서 fromJSON 함수를 정의한 것처럼 fromJSOn을 호출할 수 있다. 하지만 실제로 fromJSON은 클래스 밖에서 정의한 확장 함수이다.

⭐️ 동반 객체에 대한 확장 함수를 작성할 수 있으려면 원래 클래스에 동반 객체를 꼭 선언해야 한다. 빈 객체라도….

**객체 식: 무명 내부 클래스를 다른 방식으로 작성**

object 키워드를 싱글턴과 같은 객체를 정의하고 그 객체에 이름을 붙일 때만 사용하지 않는다. → **무명 객체**를 정의할 때도 object 키워드를 쓴다. 

<aside>
💡

무명 객체: 자바의 무명 내부 클래스를 대신한다. 

</aside>

```kotlin
window.addMouseListener(
	object: MouseAdapter(){ //MouseAdapter를 확장하는 무명 객체를 선언 
		override fun mouseClicked(e: MouseEvent){...} //MouseAdapter 메소드를 오버라이드
	}
	override fun mouseEntered(e: MouseEvent){...} //MouseAdapter 메소드를 오버라이드
)
```

객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 클래스나 인스턴스에 이름을 붙이지 않는다. → 이런 경우 보통 함수를 호출하면서 인자로 무명 객체를 넘기기 때문에 클래스와 인스턴스 모두 이름이 필요하지 않다. 

하지만 객체에 이름을 붙여야 한다면?! 변수에 무명 객체를 대입하면 된다. 

```kotlin
val listener= object: MouseAdapter(){
	override fun mouseClicked(e: MouseEvent){...}
	override fun mouseEntered(e: MouseEvent){...}
}
```

<aside>
💡

객체 선언과 달리 무명 객체는 싱글턴이 아니다. 객체 식이 쓰일 때마다 새로운 인스턴스가 생성된다. 

</aside>

자바의 무명 클래스와 같이 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있다. 

BUT! 자바와 달리 final이 아닌 변수도 객체 식 안에서 사용할 수 있다. → 따라서 객체 식 안에서 그 변수의 값을 변경할 수 있다. 

```kotlin
fun countClicks(window: Window){
	val clickCount=0; //로컬 변수 정의 
	
	window.addMouseListener(object: MouseAdapter(){
		override fun mouseClicked(e: MouseEvent){
			clickCount++ //로컬 변수의 값 변경 
		}
	}
}
```
