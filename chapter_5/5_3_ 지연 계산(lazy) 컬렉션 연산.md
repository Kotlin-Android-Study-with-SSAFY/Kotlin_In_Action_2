# 지연 계산(lazy) 컬렉션 연산

> Kotlin in action (p.223 ~ p.229)
>
> map이나 filter 같은 컬렉션 함수는 결과 컬렉션을 즉시 생성한다. → 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다

💡 시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렌션 연산을 연쇄할 수 있다. 

```kotlin
people.map(Person::name).filter {it.startWith("A")}
```

→ filter와 map은 리스트를 반환하기 때문에 위 코드를 실행하면 리스트를 2개 만드는 것이다. 한 리스트는 map의 결과를 담고, 다른 하나는 filter의 결과를 담는다.  (비효율적)

```kotlin
people.asSequence() //원본 컬렉션을 시퀀스로 변환한다. 시퀀스도 컬렉션과 똑같은 API 제공 
	.map(Person::name) 
	.filter(it.startWith("A") }
	.toList() //결과 시퀀스를 다시 리스트로 변환 
```

→ 중간 결과를 저장하는 컬렉션이 생기지 않기 때문에 원소가 많은 경우 성능이 눈에 띄게 좋아진다. 

코틀린 지연 계산 시퀀스는 Sequence 인터페이스에서 시작한다. 이 인터페이스는 단지 한 번에 하나씩 열거될 수 있는 원소의 시퀀스를 표현할 뿐이다.  즉, 한 번에 한 요소씩 처리하는 컬렉션이다. 

Sequence 안에는 iterator라는 단 하나의 메소드를 통해 시퀀스로부터 원소 값을 얻을 수 있다. 

**🚗 컬렉션 vs 🚶 시퀀스(Sequence)**

| 개념 | 컬렉션  | 시퀀스  |
| --- | --- | --- |
| 실행 방식 | **즉시 실행 (Eager)** | **지연 실행 (Lazy)** |
| 데이터 처리 | 모든 요소를 한 번에 변환 | 한 요소씩 변환 |
| 중간 연산 (`map`, `filter` 등) | 즉시 새 리스트 생성 | 최종 연산 시 한 번에 실행 |
| 사용 예시 | 작은 데이터 처리에 적합 | 대량 데이터 처리에 적합 |

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4, 5)

    // map이 전체 리스트를 변환한 후, filter가 실행됨 (비효율적)
    val result = list.map { 
        println("Mapping: $it")
        it * 2 
    }.filter { 
        println("Filtering: $it")
        it > 5 
    }

    println(result) 
}

```

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4, 5)

    val result = list.asSequence()
        .map { 
            println("Mapping: $it")
            it * 2 
        }
        .filter { 
            println("Filtering: $it")
            it > 5 
        }
        .toList() // 최종 연산 (여기서만 실행됨)

    println(result)
}

```

시퀀스의 원소는 필요할 때 비로소 계산된다. → 중간 처리 결과를 저장하지 않고도 연산을 연쇄적으로 적용해서 효율적으로 계산을 수행할 수 있다. 

> Quiz!!
asSequence 확장 함수를 호출하면 어떤 컬렉션이든 시퀀스로 바꿀 수 있을까?
> 

Sequence를 리스트로 만들 때는 **toList**를 사용한다. 

시퀀스에 대한 연산을 지연 계산하기 때문에 정말 계산을 실행하게 만들려면 최종 시퀀스의 원소를 하나씩 이터레이션하거나 최종 시퀀스를 변환해야 한다. 

## 시퀀스 연산 실행: 중간 연산과 최종 연산

중간 연산: 다른 시퀀스를 반환한다. → 그 시퀀스는 최초 시퀀스의 원소를 반환하는 방법을 안다. 

최종 연산: 결과를 반환→ 최초 컬렉션에 대해 변환을 적용한 시퀀스로부터 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 숫자 또는 객체이다. 

중간 연산은 항상 지연 계산된다.

```kotlin
listOf(1,2,3,4).asSequence()
	.map{ print("map($it)"); it*it}
	.filter{ print("filter($it)"); it%2==0}
```

이 코드를 실행하면 아무 내용도 출력되지 않는다. 이는 map과 filter 변환이 늦춰져서 결과를 얻을 필요가 있을때 (즉, 최종 연산이 호출될 때) 적용된다. 

최종연산을 호출하면 연기됐던 모든 계산이 수행된다.  

```kotlin
println(listOf(1,2,3,4).asSequence()
												.map{it*it}.find{it>3})
```

위 연산을 컬렉션에 수행하면? 

시퀀스를 사용하면?  지연 계산으로 인해 원소 중 일부의 계산을 이뤄지지 않는다. 

![image](https://github.com/user-attachments/assets/2b93b589-8fcf-4b37-93dd-004e1f5cddf0)


```kotlin
val people=listOf(Person("Alice", 29), Person("Bob",31), 
				Person("Charles",31), Person("Dan",21))

println(people.asSequence().map(Person::name)
				.filter{it.length<4}.toList())
				

println(people.asSequence().filter{it.name.length<4}
				.map(Person::name).toList())

```
![image](https://github.com/user-attachments/assets/8d454f99-74f9-42ac-b04b-7325189e8b12)


map을 먼저 하면 모든 원소를 반환한다. 하지만 filter를 먼저 하면 부적절한 원소를 제외하기 때문에 더 효율적이다. 

## 시퀀스 만들기

asSequence() 말고 generateSequence 함수를 사용할 수 있다. 

generateSequence: 이전의 원소를 인자로 받아 다음 원소를 계산한다. 

→ 컬렉션 없이도 직접 시퀀스를 만들 수 있다. 

```kotlin
val naturalNumbers= generateSequence(0) {it+1} //0부터 시작해서 다음 숫자를 계속 생성하는 시퀀스
val numbersTo100=naturalNumbers.takeWhile {it<=100} //100이하만 가져오기 
println(numbersTo100.sum()) //모든 지연 연산은 sum의 결과를 계산할 때 수행된다. 
```

takeWhile를 사용하여 무한 루프 방지
