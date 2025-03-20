# Null 활용 확장

> Kotlin in action (p.261 ~ p.273)

## **나중에 초기화할 프로퍼티 (lateinit)**

코틀린에서는 일반적으로 모든 프로퍼티를 생성자에서 초기화해야 한다. 그러나 다음과 같은 경우에는 프로퍼티를 생성자 밖에서 나중에 초기화해야 할 필요가 있다.

- 안드로이드의 **`onCreate`** 메소드나 JUnit의 **`@Before`** 메소드와 같이 객체 생성 이후 별도의 메소드에서 초기화해야 하는 경우
- 이때 널이 될 수 없는 타입을 사용하면 생성자에서 반드시 초기화해야 하므로, 일반적으로는 널이 될 수 있는 타입을 사용하게 된다.
- 하지만 이 경우, 프로퍼티 접근 시마다 널 체크(**`?.`**) 또는 널 아님 단언(**`!!`**)을 해야 하므로 코드가 복잡해진다.

이를 해결하기 위해 코틀린은 **`lateinit`** 변경자를 지원한다.

### **예시 코드**

널이 될 수 있는 타입 사용 시 (불편한 예):

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null

    @Before fun setup() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo", myService!!.performAction())
    }
}
```

**`lateinit`** 사용 시 (권장):

```kotlin
class MyTest {
    private lateinit var myService: MyService

    @Before fun setup() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo", myService.performAction())
    }
}
```

**주의 사항**

- **`lateinit`** 프로퍼티는 항상 **`var`** 이어야 한다. (**`val`** 은 불가능)
- 초기화 전에 접근하면 **`"lateinit property has not been initialized"`** 라는 명확한 예외가 발생한다.
- DI 프레임워크와 함께 자주 사용된다.

## **널이 될 수 있는 타입 확장**

널이 될 수 있는 타입(**`Type?`**)에 대한 확장 함수를 정의하면, 널 값을 안전하고 편리하게 처리할 수 있다.

### **예시 코드**

```kotlin
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {
        println("Please fill in the required fields")
    }
}

verifyUserInput(" ")   // 출력: Please fill in the required fields
verifyUserInput(null)  // 출력: Please fill in the required fields
```

**내부 구현 예시 (`isNullOrBlank`)**

```kotlin
fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank()
```

- 확장 함수 내부에서는 **`this`** 가 널일 수 있으므로 명시적으로 널 검사를 해야 한다.
- 자바와 달리 코틀린에서는 널이 될 수 있는 타입의 확장 함수 내부에서 **`this`** 가 널일 수 있다.

## **타입 파라미터의 널 가능성**

코틀린에서 함수나 클래스의 타입 파라미터는 기본적으로 널이 될 수 있다. 즉, 별도의 표시 없이도 타입 파라미터는 항상 널 가능성을 가진다.

### **예시 코드**

기본적으로 T는 널 가능:

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode())
}

printHashCode(null) // 가능 (T는 Any? 타입으로 추론 됨)
```

널 불가능한 상한 지정하기:

```kotlin
fun <T : Any> printHashCode(t: T) {
    println(t.hashCode())
}

// printHashCode(null) 호출 시 컴파일 에러 발생
```

- 타입 파라미터가 절대 널이 아님을 보장하려면 반드시 **`T : Any`** 와 같이 상한을 지정해야 한다.

## **널 가능성과 자바 (플랫폼 타입)**

자바에는 애노테이션으로 표시된 널 가능성 정보가 있다. `@Nullable String`은 코틀린에서 `String?`와 같고 `@NotNull String`은 `String`과 같다. 하지만 자바는 기본적으로 널 가능성 정보를 제공하지 않는다. 따라서 코틀린은 자바에서 가져온 타입을 **플랫폼 타입(platform type)** 으로 취급한다.

### **플랫폼 타입의 특징**

- 코틀린은 자바에서 가져온 타입을 "널 가능성 정보 없음" 상태로 취급하며, 이를 플랫폼 타입이라 부른다.
- 플랫폼 타입은 코틀린 코드에서 널 가능(**`String?`**) 또는 널 불가능(**`String`**) 어느 쪽으로든 사용할 수 있다.
- 플랫폼 타입을 다룰 때 책임은 전적으로 개발자에게 있다. 잘못된 사용 시 런타임에 예외가 발생할 수 있다.

### **예시 코드 (자바 클래스)**

```java
// Java 클래스 예시
public class Person {
    private final String name;

    public Person(String name) { this.name = name; }

    public String getName() { return name; }
}
```

코틀린에서 사용 시: 널 검사 없이 접근하면 위험할 수 있음

```kotlin
fun yellAt(person: Person) {
    println(person.name.toUpperCase() + "!!!")
}

/* yellAt(Person(null)) 호출 시 런타임 예외 발생
java.lang.IllegalArgumentException: Parameter specified as non-null
is null: method toUpperCase, parameter $receiver */ 
```

여기서 `NullPointerExcepion`이 아니라 `toUpperCase()`가 수신 객체로 널을 받을 수 없다는 자세한 예외가 발생함에 유의

안전한 접근 방법:

```kotlin
fun yellAtSafe(person: Person) {
    println((person.name ?: "Anyone").toUpperCase() + "!!!")
}

// yellAtSafe(Person(null)) 호출 시 안전하게 처리됨 ("ANYONE!!!")
```

**플랫폼 타입 도입 이유**

모든 자바 타입을 무조건 널 가능으로 처리하면 불필요한 널 검사가 많아지고 성능 저하 및 불편함이 발생한다. 따라서 코틀린 설계자는 자바에서 가져온 타입에 대해 개발자가 직접 책임지고 처리하도록 플랫폼 타입 개념을 도입했다.

## **상속 시 주의점 (자바 ↔ 코틀린)**

자바 인터페이스를 코틀린에서 구현할 때, 메소드 인자의 널 가능성을 명확히 결정해야 한다.

### **예시 코드 (자바 인터페이스)**

```java
interface StringProcessor {
    void process(String value);
}
```

코틀린 구현 시 두 가지 모두 가능:

널 불가능하게 구현:

```kotlin
class StringPrinter : StringProcessor {
    override fun process(value: String) {
        println(value)
    }
}
```

널 가능하게 구현:

```kotlin
class NullableStringPrinter : StringProcessor {
    override fun process(value: String?) {
        if (value != null) println(value)
    }
}
```

- 자바 인터페이스를 구현할 때 메소드 인자의 널 가능성을 명확히 결정하고 처리하는 것이 중요하다.
