# Types
Kotlin 의 `Int`, `Long` 은 Java 의 `int`, `long` 으로 변환된다
# Generic
Generic 클래스의 기본 원칙은 무공변하다.
- `in / out`은 파라미터 입장에서 생각해야한다.
- `out` : (함수 파라미터 입장에서의) 생산자, 공변, 자바에서는 `<out T> = <? extends T>`
- `in` : (함수 파라미터 입장에서의) 소비자, 반공변, 자바에서는 `<in T> = <? super T>`
```kotlin
class Cage<T> {}
// 메서드에 적용
class Cage<T> {
	fun put(animal: T) { ... }
	// out 을 추가해 공변(상속관계를 허용)하게 해준다.
	// out 을 붙이면 생산자 역할만 가능(꺼내는거)
	fun moveFrom(other: Cage<out T>) {
		other.getFirst() // out은 꺼내는건 가능
		other.put(this.getFirst()) // 에러! out은 넣는건 불가능
		...
	}
	
	// in 을 추가해 반공변하게 할 수 있다.
	// in 을 붙이면 소비자 역할만 가능하다 (넣는거)
	fun moveTo(other: Cate<in T>) {
	}
}

// 코틀린에서는 클래스에도 변성을 적용할 수 있다. (흔하지 않음)
// @UnsafeVariance 애노테이션을 붙여서 in/out 과 다른 변성이 들어오게 허용할수 있음
// 생산만 하는 클래스에는 out
class Cage<out T> {}
// 소비만 하는 클래스라면 in
class Cage<in T> {}
```
- **무공변(in-variant)** :  상속관계가 있더라도 두 클래스간에 아무것도 아닌것으로 간주함
- **공변(co-variant)** : 상속관계가 있다면 두 클래스간 변형을 허용함
- **반공변(contra-variant) :** 상속관계가 역전되는 관계
	![[Screenshot 2025-11-11 at 17.33.41.png]]

## Generic 에 제약걸기
```kotlin
// 제약 조건 추가
class Cage<T : Animal> {}
// 여러 제약 조건 추가
class Cage<T>(
	private val animals: MutableList<T> = mutableListOf(),
) where T : Animal, T : Comparable<T> {}

// 응용: Non-Null 타입만 허용, T만 지정하면 T? 도 허용됨
class Cage<T : Any>
```

## Generic 함수
```kotlin
// 제네릭을 이용한 확장함수
fun <T> List<T>.hasIntersection(other: List<T>): Boolean {
	return (this.toSet() intersection other.toSet()).isNotEmpty()
}
```

## 타입소거 Star projection
자바의 Generic은 1.5에서 도입되었기 때문에 그 이전버전과의 호환성을위해 런타임시 List의 타입정보를 지우게된다. 그로인해 `List<String>` 이던 `List<Int>` 이던 런타임시에는 그냥 `List` 이다 이를 타입 소거라고 한다.
```kotlin
fun checkList(data: Any) {
	// Error!
	if (data is List<String>) {
	}
}
```
Star projection 을 이용해 최소한 `List` 인지는 확인할 수 있다.
```kotlin
fun checkList(data: Any) {
	// data 가 List인지 확인할 수 있다.
	// 하지만 구체적인 타입정보를 모르기때문에 List에 데이터를 추가할 수 없다.
	if (data is List<*>) {
		val element: Any? = data[0]
	}
}
```
### 제네릭 함수에서도 타입 정보는 사라진다
런타임때 제네릭의 타입정보가 사라지기 때문에 에러가 난다.
```kotlin
fun <T> T.toSuperString(): String {
	// T가 무엇인지 런타임 때도 알 수 없기 때문에 오류가난다.
	print(T::class.java.name)
	return "Super $this"
}
```
> `T::class` 는 클래스 정보를 가져오는 코드

위 문제는 `refied` 키워드 + `inline` 함수로 해결할 수 있다.
```kotlin
inline fun <reified T> List<*>.hasAnyInstanceOf() : Boolean {
	return this.any { it is T }
}
```
> `inline` 함수 : 코드의 본문을 호출 지점으로 이동시켜 컴파일되는 함수

`inline` 함수이기 때문에 타입 파라미터 `T` 를 제네릭의 타입 파라미터로 간주하지 않기에 에러가 나지 않는다.

# lateinit
인스턴스화 시점과 프로퍼티(변수) 초기화 시점을 분리하고 싶을 때 사용한다. `liateinit` 변수는 컴파일 단계에서 `nullable` 변수로 바뀌고, 변수에 접근하려 할 때 `null` 이면 예외가 발생한다.
- 초기값을 지정하지 않고도 `null` 이 들어갈 수 없는 변수를 선언할 수 있다.
- 초기값이 지정되지 않았는데 변수를 사용하려 하면 예외가 발생한다.
- `lateinit` 을 `primitive type` 에 사용할 수 없다.
```kotlin
class Person {
	lateinit var name: String
}
```
- `primitive type` 에 `lateinit` 이 필요할 때는 `by notNull()` 을 사용
```kotlin
class Person {
	var age: Int by notNull()
}
```

# by lazy
초기화를 `get` 호출 전으로 지연시킨 변수.
초기화 로직은 변수 선언과 동시에 한 곳에만 위치할 수 있다.
```kotlin
class Person {
	val name: String by lazy {
		Threada.sleep(2_000)
		"김두한"
	}
}
```
## by lazy 의 원리
- `getValue`와 `setValue`가 있어야 `by lazy` 를 사용할 수 있다.
![[Pasted image 20251116105747.png]]
# 코틀린의 표준 위임 객체
### observable
`by Delegates.observable() {}`
- js 의 `onChange` 같은것
```kotlin
class Person {
	var age: Int by Delegates.observable(20) { property, oldValue, newValue -> 
		println("...")
	}
}
```
### vetoable
`vetoable`
setter 가 호출될 때 `onChange` 함수가 `true` 를 반환하면 변경 적용, `false` 를 반환하면 이전 값이 그대로 남는다.
```kotlin
class Person {
	var age: Int by vetoable(20) { property, oldValue, newValue -> 
		newValue >= 1
	}
}
```
- 예제 코드에서는 `newValue` 가 `1` 이상일때만 값이 적용됨
### 또 다른 프로퍼티로 위임하기
```kotlin
class Person {
	@Deprecated("age를 사용하세요", ReplaceWith("age"))
	var num: Int = 0
	
	var age = Int by this::num
}
```
- 프로퍼티 앞에 `::`를 붙이면 위임객체로 사용할 수 있다.
### Map
- 위임된 `Map` 에서 값을 찾는다
```kotlin
class Person(map: Map<String, Any>) {
	val name: String by map
	val age: Int by map
}
```

# SAM
Single Abstract Method; 추상 메소드를 하나만 가지고 있는것을 말한다. 자바의 `FunctionalInterface` 를 생각하면 된다. 자바에서는 **람다**로 인스턴스화 할 수 있지만 코틀린에서는 **람다식**화할 수 없다.

그럴때는 **SAM 생성자** 방법을 사용하면 된다.
```kotlin
val filter = StringFilter { s -> s.startWith("A") }
```

## 코틀린에서 SAM Interface
코틀린만 사용할 때는 SAM Interface 를 사용할 일은 거의 없다.
```kotlin
fun interface KStringFilter {
	fun predicate(str: String): Boolean
}
```

# Reference
`::`

# 연산자 오버로딩
```kotlin
class Point(
	x: Int,
	y: Int,
) {
	operator fun inc(): Point {
		return Point(x+1, y+1)
	}
}

fun main() {
	var point = Point(20, 10)
	println(++point) // inc() 가 불림
}
```

# Kotlin DSL


# Plugins
## JMH
micro 벤치마킹 플러그인
- `/src/main` 밑에가 아니라 `/src/jmh/` 아래에 코드를 작성해야 한다.
- 벤치마크를 위해서는 상속 가능할 필요가 있기 때문에 `class` 에 `open` 을 붙여줘야 한다.

## Jackson
코틀린에서 ObjectMapper 를 사용하기 위해서는 data class 지원을 위한 모듈을 추가해줘야 한다.
```kotlin
implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
```
ObjectMapper 자체는 `spring-boot-starter-web` 혹은 개별 추가할 경우 `spring-boot-starter-json` 을 사용하면 된다.
```kotlin
implementation("org.springframework.boot:spring-boot-starter-json")
```
자바와 다른점은 `ObjectMapper()` 가 아니라 `jacksonObjectMapper()` 를 사용해서 인스턴스화해줘야 한다.
```kotlin
val mapper = jacksonObjectMapper()
```
