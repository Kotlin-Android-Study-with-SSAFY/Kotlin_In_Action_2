# 컬렉션 함수형 api

> Kotlin in action (p.214 ~ p.222)

컬렉션을쉽게 다루고 코드를 간결하게 만들 수 있는 컬렉션 API를 소개한다.
람다를 지원하는 언어들에서 찾아볼 수 있는 함수들을 설명한다.

> **람다**란?
> 기본적으로 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다

## 1. 필수적인 함수: filter와 map
### filter
컬렉션을 돌면서 람다가 true를 반환하는 원소만 모은다.

```kotlin
val list = listOf(1, 2, 3, 4) 
println(list.filter { it % 2 == 0}) // [2, 4]
```
![image](https://github.com/user-attachments/assets/c7882fb4-cb45-4954-83aa-04f3aed14046)


객체들도 원하는 원소만 손쉽게 걸러낼 수 있다.
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob" ,31)}
println(poeple.filter { it.age > 30 }) // [Person(name=Bob, age=31)]
```


### map
map은 컬렉션을 돌면서 각 원소에 람다를 적용해 그 결과를 새 컬렉션으로 만든다.

```kotlin
val list = listOf(1, 2, 3, 4)
println(list.map { it * it }) // [1, 4, 9, 16]
```

컬렉션 예제는 아래와 같다.
```kotlin
val people = listOf(Person("Alice" , 29), Person("Bob" , 31))
println(people.map { it.name }) // [Alice, Bob]
```

---
멤버 참조로 아래처럼 작성할 수도 있다.
```kotlin
people.map(Person::name)
```

아래와 같이연쇄로 호출할 수도 있다.
```kotlin
poeple.filter { it.age > 30 }.map(Person::name) // [Bob]
```

위 코드의 전체 식은 아래와 같다.
인자 마지막이 람다식이라면 괄호 밖으로 뺄 수 있고, 유일한 인자라면 괄호도 생략할 수 있기 때문에 `{ }` 를 바로 사용했다. _5.1.3_
```kotlin
people.filter({ it.age > 30 }).map(Person::name)
```

연쇄적인 괄호를 더 정확히 하면
```kotlin
(people.filter({ it.age > 30 })).map(Person::name)
```
하지만 위 예제 구문이 간결하므로 빨리 익숙해지는 편이 낫다.

---

가장 나이가 많은 사람의 이름을 알고 싶을 때 아래처럼 쓸 수 도 있다.

```kotlin
poeple.filter { it.age == people.maxBy(Person::age)!!.age }
````
하지만 사람이 100명이라면 maxBy를 100번 수행한다.
따라서 아래처럼 한 번만 계산하므로 미리 계산하는 편이 낫다. 겉으로는 단순해보여도 내부 로직이 복잡할 수 있다.

```kotlin
val maxAge = people.maxBy(Person::age)!!.age 
poeple.filter { it.age == maxAge }
````

---

맵은 key와 value를 처리하는 함수가 각각 존재한다.
`filterKeys` , `mapKeys` 는 키를 걸러내거나 변환하고, `filterValues` 와 `mapValues` 는 값을 걸러 내거나 변환한다.

```kotlin
val numbers = mapOf(0 to "zero", 1 to "one")
println(numbers.mapValues { it.value.toUpperCase() })
{0=AERO, 1=ONE}
```


## 2. all, any, count, find: 컬렉션의 술어 적용

일단 어떤 사람의 나이가 27살 이하인지 판단한느 술어 함수를 만들자.
이 함수를 재활용하면서 all, any, cound, find가 무엇을 하는지 차이가 무엇인지 알아보겠다.

```kotlin
val canBeInclub27 = { p: Person -> p.age <= 27 }
```

### all

all은 컬렉션의 모든 원소가 이 조건을 만족하는지를 boolean타입으로 반환한다.

```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.all(canBeInClub27)) // false
```

### any

any는 만족하는 원소가 하나라도 있는지를 반환한다.

```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.any(canBeInClub27)) // true
```


조건에 !any를 수행한 결과과 , all 결과에 ! 부정을 수행한 결과도 같다.
가독성을 위해 앞에 아래 all처럼 !를 붙이지 않는 편이 낮다.

```kotlin
val list = listOf(1, 2, 3)
println(!list.all { it == 3}) // true
println(list.any { it != 3 }) // true
```

### count

count는 만족하는 원소의 개수를 Int 형으로 반환하는 API다.

```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.count(canBeInClub27)) // 1
```

count존재를 잊고 filter에 size를 붙이는 경우가 있는데, 이렇게 처리하면 컬렉션을 따로 저장하기 때문에 count가 훨씬 효율적이다.

```kotlin
// 권장X
println(people.filter(canBeInClub27).size // 1
```

### find

find는 조건을 만족하는 원소가 하나라도 있는 경우 가장 먼저 만족하는 요소를 반환한다.
만족하는 원소가 하나도 없는 경우 null를 반환한다.
이름이 더 명확한 firstOrNull 를 써도 된다.

```kotlin
val people = listOf(Person("Alice", 27), Person("Bob", 31))
println(people.find(canBeInClub27)) // Person(name=Alice, age=27)
```
![image](https://github.com/user-attachments/assets/85056fc6-7a49-4b25-bd1e-c1e55f4147df)


## 3. groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

```kotlin
val people = listOf(Person("Alice", 31), Person("Bob", 29), Person("Carol", 31))
println(people.groupBy { it.age })
```

위 연산은 it.age가 키다. 그 기 값에 따라 그룹으로 나누어진 map이 반환된다.
결과 타입은 Map<Int, List<Person>>이다.


groupBy를 아래처럼 문자열 첫 글자에 따라 분류하는 코드로 만들 수도 있다.
여기서 first는 멤버가 아니라 확장함수다.

```kotlin
val list = listOf("a", "ab", "b")
println(list.groupBy(String:first)) // {a=[a, ab], b=[b]}
```

## 4. flatMap와 flatten: 중첨된 컬렉션 안의 원소 처리

### flatMap

flatMap은 람다를 모든 요소에 적용을 하고, 결과는 한 리스트로 모은다.
map를 적용하고 flatten한다.

```kotlin
book.flatMap { it.authors }.toSet()
```

```kotlin
val strings = listOf("abc", "def")
println(strings.flatMap { it.toList() }) // [a, b, c, d, e, f]
```

toSet()를 활용해 아래처럼 작가를 모아둔 집합을 만들 수 있다.

```kotlin
val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")), Book("Mort", listOf("TerryPratchett")), Book("Good Omens", listOf("Terry Pratchett", "Neil Gaiman")))
println(books.flatMap { it.authors }.toSet()) // [Jasper Fforde, Terry Pratchett, Neil Gaiman]
```

### flatten

map을 통해 변환해야할 내용이 딱히 없다면 바로 flatten을 사용하면 된다.

```kotlin
val listofLists = listOf(listOf(1), listOf(2, 3), listOf(4, 5, 6)
println(listofLists.flatten()) // [1, 2, 3, 4, 5, 6]
```

