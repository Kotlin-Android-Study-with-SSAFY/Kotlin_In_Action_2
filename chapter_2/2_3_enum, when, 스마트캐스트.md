# enum, when, 스마트 캐스트

> Kotlin in action (p.77 ~ p.89)

## **코틀린의 선택 표현과 처리: enum과 when**

이번 절에서는 코틀린의 `when` 식에 대해 설명한다. `when`은 자바의 `switch`를 대체하는 강력한 기능이며, 코틀린 프로그래밍에서 자주 사용될것이다. 코틀린에서 `enum`을 선언하는 방법과 스마트 캐스트에 대해서도 살펴본다.

### enum 클래스 정의

`enum class` 키워드를 사용하여 열거형 클래스를 정의할 수있다. `enum` 클래스 안에는 프로퍼티나 메소드를 정의할 수도있다. 이때, 상수 목록과 메소드 정의 사이에 세미콜론(;)을 넣어줘야한다.

**프로퍼티와 메소드가 있는 enum 클래스 선언하기**

```kotlin
enum class Color(val r: Int, val g: Int, val b: Int) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255),
    INDIGO(75, 0, 130),
    VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

### **when으로 enum 클래스 다루기**

`when` 식은 자바의 `switch` 문과 유사하지만, 훨씬 더강력하다. 값을 만들어내는 식이기 때문에, 식을 본문으로 하는 함수에 바로 사용할 수 있다.

**when을 사용해 올바른 enum 값 찾기**

```kotlin
// when
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }

// 한 when 분기 안에 여러 값 사용하기
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
        Color.GREEN -> "neutral"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
    }
```

`when` 식에서는 각 분기의 끝에 `break`를 넣을 필요가 없다. 한 분기 안에서 여러 값을 매치 패턴으로 사용할 수도 있다. enum 상수 값을 임포트하면 enum 클래스 수식자 없이 enum을 사용할 수 있다.

### **when과 임의의 객체를 함께 사용**

코틀린의 `when`은 분기 조건에 상수뿐 아니라 임의의 객체를 허용한다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")
    }
```

`when` 식은 인자 값과 매치하는 조건 값을 찾을 때까지 각 분기를 검사한다. 모든 분기 식에서 만족하는 조건을 찾을 수 없다면 `else` 분기의 문장을 실행한다.

### **인자 없는 when 사용**

위의 함수는 호출될 때마다 함수 인자로 주어진 두 색이 when의 분기 조건에 있는 다른 두 색과 같은지 비교하기 위해 여러 Set 인스턴스를 생성한다. 인자가 없는 `when` 식을 사용하면 불필요한 객체 생성을 막을 수 있다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2 == YELLOW) ||
        (c1 == YELLOW && c2 == RED) ->
            ORANGE
        (c1 == YELLOW && c2 == BLUE) ||
        (c1 == BLUE && c2 == YELLOW) ->
            GREEN
        (c1 == BLUE && c2 == VIOLET) ||
        (c1 == VIOLET && c2 == BLUE) ->
            INDIGO
        else -> throw Exception("Dirty color")
    }
```

`when`에 인자가 없으려면 각 분기의 조건이 불리언 결과를 계산하는 식이어야 한다.

### **스마트 캐스트: 타입 검사와 타입 캐스트를 조합**

코틀린에서는 `is`를 사용해 변수 타입을 검사한다. `is` 검사는 자바의 `instanceof`와 비슷하지만, 코틀린에서는 컴파일러가 캐스팅을 해주는 **스마트 캐스트** 기능이 있다.

**if 연쇄를 사용해 식을 계산하기**

```kotlin
fun eval(e: Expr): Int {
    if (e is Num) {
        return e.value // 스마트 캐스트
    }
    if (e is Sum) {
        return eval(e.right) + eval(e.left) // 스마트 캐스트
    }
    throw IllegalArgumentException("Unknown expression")
}
```

스마트 캐스트는 is로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다.

원하는 타입으로 명시적으로 타입 캐스팅하려면 `as` 키워드를 사용한다.

```kotlin
val n = e as Num
```

### **if를 when으로 변경**

코틀린에서는 `if`가 값을 만들어내기 때문에 자바와 달리 3항 연산자가 따로 없다. 이런 특성을 사용하면 `eval` 함수에서 `return`문과 중괄호를 없애고 `if`식을 본문으로 사용해 더 간단하게 만들 수 있다.

**값을 만들어내는 if 식**

```kotlin
fun eval(e: Expr) : Int =
		if (e is Num) {
				e.value
		} else if (e is Sum) {
				eval(e.right) + eval(e.left)
		} else {
				throw IllegalArgumentException("Unknown expression")
		}
```

**if 중첩 대신 when 사용하기**

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

`if`나 `when`의 각 분기에서 수행해야 하는 로직이 복잡해지면 분기 본문에 블록을 사용할 수 있다.

## **if와 when의 분기에서 블록 사용**

`if`나 `when` 모두 분기에 블록을 사용할 수 있다. 블록의 마지막 문장이 블록 전체의 결과가 된다.

**분기에 복잡한 동작이 들어가 있는 when 사용하기**

```kotlin
fun evalWithLogging(e: Expr): Int =
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

블록의 마지막 식이 블록의 결과라는 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립한다. 식이 본문인 함수는 블록을 본문으로 가질 수 없고 블록이 본문인 함수는 내부에 `return` 문이 반드시 있어야 한다.