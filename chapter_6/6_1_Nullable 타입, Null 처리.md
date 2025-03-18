# Nullable타입, Null 처리

> Kotlin in action (p.244 ~ p.260)

## 0. 널 가능성

널 가능성(Nullability)는 NullPointerException 오류(NPE)를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성이다.
자바를 사용하다보면 "AN error has occurred: java.lang.NullPointerException" 메세지를 본 적 있을 것이다.

코틀린에서는 이런 오류를 줄이기 위해 null 확인을 런타임이 아닌 컴파일 시점으로 옮긴다.
컴파일 시점에 미리 감지하기 위해서 널이 될 수 있는지 여부를 타입 시스템에 추가한다.

## 1. 널이 될 수 있는 타입

코틀린에서는 널이 될 수 있는 타입을 지원한다. 기본적으로는 null을 허용하지 않는다.

`널이 될 수 있는 변수`에 메소드를 호출할 수 없도록 해서 NullPointerException 오류를 미연에 방지한다.

이해를 돕기 위해 아래 코드를 보자.

```java
/* 자바 */
int strLen(String s){
    return s.length();
}

strLen(null);

```

자바에서는 위 함수에 null을 넘기면 NullPointerException 발생한다.

```kotlin
/* 코틀린 */
fun strLen(s: String) = s.length

strLen(null) // ERROR: Null can not be a value of a non-null type String
```

하지만 코틀린에서는 null을 넘기면, 컴파일 시 오류가 난다.
s의 타입은 String이기에 항상 String 인스턴스여야 한다는 뜻이다.

만약 null을 받을 수 있게 하려면 타입 이름뒤에 물음표(?)를 명시해야 한다.
어떤 타입이든 타입 이름 뒤에 물음표를 붙이면 프로퍼티에 null 참조를 저장할 수 있다는 뜻이다.

```kotlin
fun strLenSafe(s: String?) =  ...
```

![image](https://github.com/user-attachments/assets/29239f0e-934d-4fb9-baed-0c895b1d4d0f)

---

널이 될 수 있는 타입의 변수는 수행할 수 있는 연산이 제한된다.
아래 처럼 `변수.메소드()` 처럼 메소드를 직접 호출할 수 없다.

```kotlin
fun strLenSafe(s: String?) = s.length()

// ERROR: only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type kotlin.String?

```

널이 될 수 **없는** 타입에도 대입할 수 없다.
널이 될 수 없는 타입의 파라미터에도 넘길 수 없다.

```kotlin
val x: String? = null
var y: String = x
// ERROR: Type mismatch: inferred type is String? but String was expected

strLen(x)
// ERROR: Type mismatch: inferred type is String? but String was expected
```

그렇다면 널이 될 수 있는 타입에 메서드를 사용하려면 어떻게 해야할까?
-> null을 비교하면 된다. null과 비교하고 나면 컴파일러가 null이 아님을 기억한다.

Q. 아래 println에 무엇이 출력될까요?

```kotlin
fun strLenSafe(s: String?): Int =
    if (s != null) s.length else 0 // null 검사를 추가하면 코드가 컴파일 된다.

val x: String? = null
println(strLenSafe(x)) 
println(strLenSafe("abc")) 
```

<br/>

${\textsf{\color{gray}답}}$
${\textsf{\color{gray}0}}$
${\textsf{\color{gray}3}}$


만약 널을 다루기 위해서 if문은 다 써야한다면 코드가 번잡해질 것이다.
다행히 코틀린에서는 널이 될 수 있는 값을 다룰 때 도움이 되는 도구들을 제공한다.

그전에 널 가능성과 변수 타입의 의미에 대해 알아보자.

## 2. 타입의 의미

왜 변수에 타입을 지정해야 하는 걸까?
위키피디아 글에서는 "타입은 분류(classification)로 ... 타입은 어떤 값들이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다." 라고 되어있다.

double타입을 살펴보자. double타입에 속한 값이라면 어떤 값이든 관계없이 일반 수학 연산 함수를 적용할 수 있다.
따라서, double타입 변수가 있다 -> 컴파일러가 그 변수의 연산을 통과 -> 그 연산이 성공적으로 실행되리란 사실을 확신할 수 있다.
double은 null이 안된다.

double과 달리 자바의 String타입 변수에는 String과 null 두가지 종류가 들어갈 수 있다.

❗**하지만 이 두 종류는 완전히 다르다.**
자바에서 instanceof 연산자도 null이 String이 아니라고 답한다. null인 경우 사용할 수 있는 연산자가 많지 않다.

그럼에도 null이 함께 들어 갈 수 있다는 점이 오류 가능성을 높인다. 널 여부를 추가로 검사하기 전에는 연산을 수행할 수 있는지 알 수 없다.
실행 시점에 오류가 생기는 경우가 많다.

---

물론 자바에도 NullPointerException 문제를 해결하는 데 도움을 주는 도구가 있다.
`@Nullable` `@NotNull` 애노테이션 를 사용하면 값이 널이 될 수 있는지 여부를 표시한다.
하지만 모든 곳에 애노테이션을 추가하는 일이 쉽지 않고, 모든 NPE를 해결할 수는 없었다.

---

실행 시점에 널이 될 수 있는 타입이나 널이 될 수 없는 타입의 객체는 같다. 널이 될 수 없는 타입을 감싼 래퍼 타입이 아니다.

## 3. 안전한 호출 연산자: ?.

코틀린에서 널에서 안전한 호출 연산자를 제공한다.

`?.` 는 null 검사와 메소드 호출을 한 번의 연산으로 수행한다.

`s?.toUpperCase()` 는 `if(s != null) s.toUpperCase() else null`과 같다.
호출하려는 값이 null 이 아니라면 일반 메소드 호출처럼 작동한다.
null이라면 이 호출을 무시되고 결과값은 `null`이다.

![image](https://github.com/user-attachments/assets/8493b6bd-c8a7-43aa-9059-941f749d6809)

```kotlin
fun printAllCaps(s: String?){
    val allCaps: String? = s?.toUpperCase() // allCaps는 null일 수 있다. 결과값이 null일 수 있기 때문
    println(allCaps)
}

printAllCaps("abc") // ABC
printAllCaps(null) // null
```

조금 복잡한 코드를 보자.
managerName 함수는 이름이 null일 경우 null을 반환한다.

Q. 아래 println에 무엇이 출력될까요?

```kotlin
class Employee(val name: String, val manager: Emplyee?)
fun managerName(employee: Employee): String? = emplyee.manager?.name


val ceo = Employee("Da Boss", null)
val developer = Employee("Bob Smitch", ceo)
println(managerName(developer))
println(managerName(ceo))

```

${\textsf{\color{gray}답}}$

${\textsf{\color{gray}null}}$

${\textsf{\color{gray}Da Boss}}$


조금더 복잡한 코드를 보자.
아래처럼 `?.` 를 연쇄 사용할 수도 있다.

아래는 회사가 null인 경우 null 대신 "Unknown" 문자열을 반환하는 함수를 구현했다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String{
    val country = this.company?.address?.country // 안전한 호울 연산자를 연쇄 사용
    return if (country ! = null) country else "Unknown"
}
```

## 4. 엘비스 연산자: ?:

?. 는 null일 때 실행을 안하는 연산자였다면, null일 때 다른 값을 반환하고 싶을 수도 있다.

앨비스 연산자를 사용하면 null을 체크하는 if문을 아예 없앨 수 있다.
이를 도와주는 연산자가 앨비스 연산자 `?:` 이고, null coalescing이라고도 한다.
앨비스 연잔자를 시계방향으로 90 돌리면 앨비스 프레슬리(Elvis Presley) 헤어 스타일이 보여서 엘비르 연산자라고 불린다...

![image](https://github.com/user-attachments/assets/50de86f2-ae89-45a3-b33e-c13d685a305c)

`?:` 는 좌항이 널인지 검사한후, **널이 아니면** 좌항 결과를 반환하고, **널**이면 우항을 결과로 한다.

```kotlin
fun foo(s: String?){
    val t: String = s ?: "" // s가 널이면 빈 문자열이 결과
}
```

![image](https://github.com/user-attachments/assets/3b549b57-40c0-4c43-a251-d2f7bb14d3dd)

간단한 예시를 보자. s가 null이면 길이를 반환, null 이면 0을 반환한다.
반환값이 Int? 일 필요가 없다.

Q. 아래 println에 무엇이 출력될까요?

```kotlin
fun strLenSafe(s: String?): Int = s?.length ?: 0
println(strLenSafe("abc"))
println(strLenSafe(null))
```

${\textsf{\color{gray}답}}$

${\textsf{\color{gray}3}}$

${\textsf{\color{gray}0}}$


`?:`를 사용하면 `?.` 에서 보았던 예시를 더 짧게 쓸 수 있다.

```kotlin
fun Person.countryName(): String{
    val country = this.company?.address?.country // 안전한 호울 연산자를 연쇄 사용
    return if (country ! = null) country else "Unknown"
}
```

```kotlin
fun Person.countryName() = company?.address?.country ?: "Unknown"
```

앨비스 연산자를 이용한 복잡한 예시를 보자. 주소가 없으면 예외를 발생시킨다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun printSippingLabel(person: Person){
    val address = person.company?.address ?: throw IllegalArgumentException("No address") // 주소가 없으면 예외 발생

    with(address){ // 수신 객체 지정 람다 with
        println(streetAddress)
        println("$zipCode $city, $ccountry")
    }
}

val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
val jetbrains = Company("JetBrains", address)
val person = Person("Dmitry", jetbrains)

// null 이 아니기 때문에 주소가 잘 출력됐다.
printShippingLabel(person) // Elsetr. 47
                           // 80687 Munich, Germany

// null 이기 때문엘 앨비스 연산에 따라 예외가 던져졌다.
printShippingLabel(Person("Alexey", null)) // java.lang.IllegalArgumentException: No address

```

## 5. 안전한 캐스트: as?

지금까지 null을 안전하게 다루는 방법을 보았다면, 이번에는 타입을 안전하게 다루는 방법을 볼 것이다.

코틀린에서는 as로 지정한 타입을 바꿀 수 있다. 또한 코틀린에서는 is 를 통해 변환 가능한 타입인지 검사할 수도 있다.

> is 는 형변환이 가능하다면 true반환, 불가면 false 반환
> as 는 형변환이 가능하다면 해당 인스턴스를 반환, 불가면 null를 반환

```kotlin
class Person(val name: String)

val kim = "이름"
if(kim is Person){
    val test1 = kim as Person
}
val test2 = kim as Person // java.lang.ClassCastException
```

하지만 `as?` 연사자를 이용하면 더 간단하게 바꿀 수 있다.
`as?`는 값을 대상 타입으로 변환할 수 있는 변환하고 할 수 없으면 null을 반환한다.
겁사하는 과정이 필요 없어진다.

![image](https://github.com/user-attachments/assets/87bd52f6-0643-4770-a9b0-9a59a3a4395a)

`as?` 연산자는 equals를 구현할 때 특히 유용하다.
자바로 구현할 때는 null 검사 if문이 많이 필요했고 타입 캐스팅이 필요했다.
하지만 코를른의 as?를 이용하면 하면 한줄로 간단히 만들 수 있다.

```java
    @Override
    public boolean equals(Object obj) {
      if (this == obj)
        return true;
      if (obj == null)
        return false;
      if (getClass() != obj.getClass())
        return false;
      Person other = (Person) obj; // 명시적으로로 바꾸어주어야함
      if (!getEnclosingInstance().equals(other.getEnclosingInstance()))
        return false;
      return Objects.equals(firstName, other.firstName) && Objects.equals(lastName, other.lastName);
    }
    private Main getEnclosingInstance() {
      return Main.this;
    }
```

```kotlin
class Person(val firstName: String, val lastName: String){
    override fun equals(o: Any?): Boolean{
        val otherPerson = o as? Person ?: return false

        return otherPerson.firstName == firstName && otherPerson.lastName == lastName
    }

    ovverride fun hashCode(): Int = firstName.hashCode() * 37 + lastName.hashCode()
}

val p1 = Person("Dmitry", "jemerov")
val p2 = Person("Dmitry", "Jemerov")
println(p1 == p2) // true

println(p1.equals(42)) // false
```

## 6. 널 아님 단언: !!

널이 아니라는 것을 알고 있어서 컴파일러에게 알려주고 싶을 때도 있다.

`!!` 널 단언을 통해. 널이 아니라고 알려줄 수 있다.
그런데 만약 잘못 생각해서 null이 넘어 왔다면 NullPointerException이 발생한다.

![image](https://github.com/user-attachments/assets/e42e2cb7-5d5c-499e-b114-b89df6b73122)

```kotlin
fun ignoreNulls(s: String?){
    val sNotNull: String = s!!
    println(sNotNull.length)
}
ignoreNulls(null) // Exception in thread "main" kotlin.KotlinNullPointerException as <...>.ignoreNulls(07_NotnullAssertions.kt:2)
```

오류가 발생할 때 몇번째 줄인지는 알려주지만, 어느 열에서 발생했는지는 알려주지 않는다.
따라서 아래같은 연쇄 사용은 피하라.

```
person.company!!.address!!.country
```

> !!가 마치 컴파일러에세 소리를 지르는 느낌이 든다. 무례해 보이는 것은 코틀린이 의도한 바이다. 단언하기 보다는 더 나은 방법을 찾아보라는 의도로 !!라는 못생긴 기호를 선택했다.

사용자가 isEnable이 true일 때만 actionPerformed를 호출한다고 생각해보자.
하지만 컴파일러는 selectedValue가 null이 아닌지 알 수 없다.
따라서 개발자가 null이 아니라는 것을 알고 있으니 `!!`를 통해 널이 아니라고 말할 수 있다.

isEnabled가 true -> actionPerformed 호출

```kotlin
class copyRowAction(val list: JList<String>) : AbstractAction(){
    override fun isEnabled(): Boolean = list.selectedValue != null
    override fun actionPerformed(e: ActionEvent){
        val value = list.selectedValue!!
    }
}
```

## 7. let 함수

let함수를 통해 널이 아닌지 검사한 후 변과를 변수에 넣는 작업을 간단하게 할 수 있다.
아래 코드에서 email 변수를 넘기고 싶은데 nullable이라 넘길 수 없다. 어떻게 해야할까?
넘기고 싶다면 if문을 통해 검사를 해야한 넘길 수 있다.

```kotlin
fun sendEmailTo(email: String) { /*...*/ }

val email: String? = ...
sendEmailTo(email) // ERROR: Type mismatch: inferred type is String? but String was excected

if (email != null) sendEmailTo(email)

```

let을 사용하면 검사를 하고 어떤 구문을 실행하고 싶은지 블록단위로 적을 수 있다.

```kotlin
email?.let { email -> sendEmailTo(email) }

email?.let { sendEmailTo(it) }  // 짧은 구문
```

![image](https://github.com/user-attachments/assets/91fdf939-c6f3-4a21-893e-071bb91826ae)

아래처럼 함수 결과식을 변수가 담는 과정없이 바로 ?.let 함수로 실행할 수 있다.

```kotlin
val person: Person? = getThe BestPersonInTheWorld()
if (person != null) sendEmailTo(person.email)

getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
```


그냥 let에 ?만 더한 것.
