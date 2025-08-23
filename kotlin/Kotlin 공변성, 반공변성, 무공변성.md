# Kotlin 공변성, 반공변성, 무공변성

> **공변성(Covariance)**, **반공변성(Contravariance)**, **무공변성(Invariance)** 은  
> 제네릭 타입에서 타입 안전성을 보장하면서도 유연성을 제공하는 개념입니다.    
> Kotlin에서는 `out`과 `in` 키워드를 통해 공변성과 반공변성을 명시적으로 표현할 수 있습니다.

---

## 1. 공변성, 반공변성, 무공변성이란?

### 공변성 (Covariance)
```kotlin
// 공변성: 하위 타입이 상위 타입으로 대체 가능
// T가 하위 타입이면 Box<T>도 Box<상위타입>의 하위 타입

// 예시: String은 Any의 하위 타입
val stringList: List<String> = listOf("신짱구", "신형만")
val anyList: List<Any> = stringList  // 공변성으로 인해 가능
```

### 반공변성 (Contravariance)
```kotlin
// 반공변성: 상위 타입이 하위 타입으로 대체 가능
// T가 상위 타입이면 Box<T>도 Box<하위타입>의 하위 타입

// 예시: Any는 String의 상위 타입
val anyConsumer: Consumer<Any> = { obj -> println(obj) }
val stringConsumer: Consumer<String> = anyConsumer  // 반공변성으로 인해 가능
```

### 무공변성 (Invariance)
```kotlin
// 무공변성: 타입 변환이 불가능
// Box<String>과 Box<Any>는 서로 다른 타입으로 취급

// 예시: MutableList는 무공변적
val stringList: MutableList<String> = mutableListOf("신짱구", "신형만")
// val anyList: MutableList<Any> = stringList  // 컴파일 오류! 무공변성
```

---

## 2. Kotlin에서의 표현 방법

### out 키워드 (공변성)
```kotlin
// out 키워드로 공변성 선언
class Producer<out T>(private val value: T) {
    fun get(): T = value
    // fun set(value: T) { }  // 컴파일 오류! T를 입력으로 사용할 수 없음
}

// 사용 예시
val stringProducer: Producer<String> = Producer("신짱구")
val anyProducer: Producer<Any> = stringProducer  // 공변성으로 가능

val result: Any = anyProducer.get()  // 안전함
```

### in 키워드 (반공변성)
```kotlin
// in 키워드로 반공변성 선언
class Consumer<in T> {
    fun consume(value: T) {
        println("소비: $value")
    }
    // fun get(): T { }  // 컴파일 오류! T를 출력으로 사용할 수 없음
}

// 사용 예시
val anyConsumer: Consumer<Any> = Consumer<Any>()
val stringConsumer: Consumer<String> = anyConsumer  // 반공변성으로 가능

stringConsumer.consume("신짱구")  // 안전함
```

---

## 3. 실제 활용 예시

### 1. List의 공변성
```kotlin
// List는 공변적 (out T)
val stringList: List<String> = listOf("신짱구", "신형만", "봉미선")
val anyList: List<Any> = stringList  // 공변성으로 가능

// 읽기만 가능 (안전함)
anyList.forEach { item ->
    println("이름: $item")
}

// 쓰기는 불가능 (타입 안전성 보장)
// anyList.add("철수")  // 컴파일 오류!
```

### 2. Comparator의 반공변성
```kotlin
// Comparator는 반공변적 (in T)
val anyComparator: Comparator<Any> = compareBy { it.toString() }
val stringComparator: Comparator<String> = anyComparator  // 반공변성으로 가능

val names = listOf("신짱구", "신형만", "봉미선")
names.sortedWith(stringComparator)  // 안전함
```

### 3. 함수 타입의 공변성/반공변성
```kotlin
// 함수 타입에서의 공변성/반공변성
val stringToInt: (String) -> Int = { it.length }
val anyToInt: (Any) -> Int = stringToInt  // 공변성 (출력 타입)

val intToString: (Int) -> String = { it.toString() }
val intToAny: (Int) -> Any = intToString  // 공변성 (출력 타입)

val anyConsumer: (Any) -> Unit = { println(it) }
val stringConsumer: (String) -> Unit = anyConsumer  // 반공변성 (입력 타입)
```

### 4. 무공변성의 예시
```kotlin
// MutableList는 무공변적 (읽기/쓰기 모두 가능)
val stringList: MutableList<String> = mutableListOf("신짱구", "신형만")
val anyList: MutableList<Any> = mutableListOf("봉미선", 123)

// 타입 변환 불가능
// val convertedList: MutableList<Any> = stringList  // 컴파일 오류!

// 이유: 타입 안전성 보장
// 만약 변환이 가능하다면:
// anyList.add(123)  // String 리스트에 Int 추가 가능해짐 (위험!)
```

---

## 4. 안드로이드에서의 활용

### 1. Repository 패턴에서의 공변성
```kotlin
// Repository 인터페이스 (공변적)
interface Repository<out T> {
    fun getAll(): List<T>
    fun getById(id: Long): T?
    // fun save(item: T)  // 불가능! T를 입력으로 사용할 수 없음
}

// 구현체
class UserRepository : Repository<User> {
    override fun getAll(): List<User> = listOf(
        User("신짱구", 5),
        User("신형만", 35)
    )
    
    override fun getById(id: Long): User? = User("신짱구", 5)
}

// 사용
val userRepo: Repository<User> = UserRepository()
val anyRepo: Repository<Any> = userRepo  // 공변성으로 가능

val users: List<Any> = anyRepo.getAll()  // 안전함
```

### 2. Event Handler에서의 반공변성
```kotlin
// Event Handler (반공변적)
interface EventHandler<in T> {
    fun handle(event: T)
}

// 구체적인 이벤트들
sealed class UserEvent {
    data class Login(val username: String) : UserEvent()
    data class Logout(val userId: Long) : UserEvent()
}

// Any 이벤트를 처리하는 핸들러
val anyEventHandler: EventHandler<Any> = object : EventHandler<Any> {
    override fun handle(event: Any) {
        println("이벤트 처리: $event")
    }
}

// UserEvent 전용 핸들러로 사용 가능 (반공변성)
val userEventHandler: EventHandler<UserEvent> = anyEventHandler

// 사용
userEventHandler.handle(UserEvent.Login("신짱구"))
```

### 3. ViewModel에서의 활용
```kotlin
// 상태 관리에서의 공변성
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}

// 사용
val userState: UiState<User> = UiState.Success(User("신짱구", 5))
val anyState: UiState<Any> = userState  // 공변성으로 가능

when (anyState) {
    is UiState.Loading -> { /* 로딩 처리 */ }
    is UiState.Success -> {
        val user = anyState.data  // Any 타입으로 접근
    }
    is UiState.Error -> { /* 에러 처리 */ }
}
```

### 4. 무공변성의 활용
```kotlin
// 데이터 저장소 (무공변적)
class DataStore<T> {
    private val data = mutableListOf<T>()
    
    fun add(item: T) {
        data.add(item)
    }
    
    fun get(index: Int): T {
        return data[index]
    }
    
    fun getAll(): List<T> {
        return data.toList()
    }
}

// 사용
val userStore: DataStore<User> = DataStore()
userStore.add(User("신짱구", 5))

// 타입 변환 불가능 (무공변성)
// val anyStore: DataStore<Any> = userStore  // 컴파일 오류!

// 이유: 읽기/쓰기 모두 가능하므로 타입 안전성 보장 필요
```

---
