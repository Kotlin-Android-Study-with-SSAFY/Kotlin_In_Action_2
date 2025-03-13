# 수신 객체 지정 람다: with와 apply

> Kotlin in action (p.236 ~ p.241)

수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있는 코틀린만의 독특한 기능을 설명한다. 그런 람다를 **수신 객체 지정 람다**라고 부른다. `with`함수는 수신 객체 지정 람다를 사용한다.

### with 함수

코틀린은 `with`라는 라이브러리 함수를 통해 어떤 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있는 기능을 제공한다.

`with`를 사용하지 않은 경우

```kotlin
fun alphabet () : String {
	val result = StringBuilder()
	for(letter in 'A'..'Z'){
		result.append(letter)
	}
	result.append("\nNow I Know the alphabet!")
	return result.toString()
}

>>> println(alphabet())
ABCEDFGHIJKLMNOPQRSTUWXYZ
Now I Know the alphabet!
```

`with`를 사용한 경우

```kotlin
fun alphabet () : String {
	val stringBuilder = StringBuilder()
	return with(stringBuilder) {
		for(letter in 'A'..'Z'){
			this.apeend(letter)
		}
		append("\nNow I Know the alphabet!")
		this.toString()
	}
}

```

해당 코드에서 `with`문은 파라미터가 2개 있는 함수이다. 첫 번째 파라미터는 `stringBuilder`이고 두 번째 파라미터는 람다이다. 람다를 괄호 밖으로 빼내는 관례를 사용함에 따라 전체 함수 호출이 언어가 제공하는 특별 구문처럼 보인다.

`with` 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 받은 람다 본문에서는 `this`를 사용해 그 수신 객체에 접근할 수 있다. 일반적인 this와 마찬가지로 this와 점(.)을 사용하지 않고 프로퍼티나 메소드 이름만 사용해도 수신 객체의 멤버에 접근할 수 있다.

```kotlin
fun alphabet () = with(stringBuilder) {
	for(letter in 'A'..'Z'){
		apeend(letter)
	}
	append("\nNow I Know the alphabet!")
	toString()
}
```

불필요한 stringBuilder 변수를 없애면 alphabet 함수가 식의 결과를 바로 반환하게 된다. 따라서 식을 본문으로 하는 함수로 표현할 수 있다.

`with`가 반환하는 값은 람다 코드를 실행한 결과이며 그 결과는 람다 식의 본문에 있는 마지막 식의 값이다. 하지만 때로는 람다의 결과 대신 수신 객체가 필요한 경우도 있다. 그럴  때는 `apply` 라이브러리 함수를 사용할 수 있다.

### apply 함수

`apply` 함수는 거의 `with`와 같다. 유일한 차이는 `apply`는 항상 자신에게 전달된 객체를 반환한다는 점이다.

다음은 `apply`를 써서 `aphabet` 함수를 다시 리팩터링한 결과이다.

```kotlin
fun alphabet() = StringBuilder().apply {
	for(letter in 'A'..'Z'){
		apeend(letter)
	}
	append("\nNow I Know the alphabet!")
}.toString()
```

`apply`는 확장 함수로 정의돼 있다. `apply`의 수신 객체가 전달받은 람다의 수신 객체가 된다.이 함수에서 `apply`를 실행한 결과는 `StringBuilder` 객체이다. 따라서 그 객체의 `toString`을 호출해서 `String` 객체를 얻을 수 있다.

이런 `apply` 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다. 자바에서는 보통 별도의 `Builder` 객체가 이런 역할을 담당한다.

다음은 `apply`를 객체 초기화에 활용하는 예이다.

```kotlin
fun createViewWithCustomAttributes(context: Context) = TextView(context).apply {
	text = "Sample Text"
	textSize = 20.0
	setPadding(10, 0, 0, 0)
}
```

`apply`함수를 사용하면 함수의 본문에 간결한 식을 사용할 수 있다. 새로운 `TextView` 인스턴스를 만들고 즉시 인스턴스를 `apply`에 넘긴다. `apply`에 전달된 람다 안에서는 `TextView`가 수신 객체가 된다. 따라서 원하는 대로 `TextView`의 메소드를 호출하거나 프로퍼티를 설정할 수 있다. 람다를 실행하고 나면 `apply`는 람다에 의해 초기화된 `TextView` 인스턴스를 반환한다.
