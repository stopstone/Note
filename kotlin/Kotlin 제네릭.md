# Kotlin 제네릭

> Kotlin의 **제네릭(Generic)** 을 잘 활용하면 타입 안전성을 보장하면서 코드 재사용성을 높일 수 있습니다.  
> 본 문서에서는 제네릭의 기본 개념부터 고급 활용법까지 실무 중심으로 정리해보겠습니다.  

---

## 1. 제네릭이란?

### 1-1. 제네릭의 필요성

제네릭이 없던 시절의 문제점을 살펴보겠습니다:

```kotlin
// 제네릭 없이 List를 사용하는 경우
val stringList = mutableListOf<Any>()
stringList.add("hello")
stringList.add(123) // 의도하지 않은 타입도 추가 가능

// 런타임에 타입 캐스팅 에러 발생 가능
val firstItem = stringList[0] as String // ClassCastException 위험
```

제네릭을 사용하면 이러한 문제를 해결할 수 있습니다:

```kotlin
// 제네릭을 사용한 타입 안전한 List
val stringList = mutableListOf<String>()
stringList.add("hello")
// stringList.add(123) // 컴파일 에러: 타입 불일치

val firstItem = stringList[0] // 안전한 String 타입
```

### 1-2. 제네릭의 장점

- **타입 안전성**: 컴파일 시점에 타입 오류를 잡을 수 있습니다.
- **코드 재사용성**: 하나의 클래스나 함수로 여러 타입을 처리할 수 있습니다.
- **타입 추론**: 컴파일러가 타입을 자동으로 추론해줍니다.
- **캐스팅 제거**: 런타임 캐스팅이 필요 없어집니다.

---

## 2. 제네릭 클래스

### 2-1. 기본 제네릭 클래스

```kotlin
// 제네릭 클래스 정의
class Box<T>(private var content: T) {
    fun getValue(): T = content
    
    fun setValue(value: T) {
        content = value
    }
}

// 사용 예시
val stringBox = Box<String>("Hello")
val intBox = Box<Int>(42)

println(stringBox.getValue()) // "Hello"
println(intBox.getValue())    // 42
```

### 2-2. 제네릭 클래스의 타입 제약

```kotlin
// 특정 타입만 허용하는 제네릭 클래스
class NumberBox<T : Number>(private var content: T) {
    fun getValue(): T = content
    
    fun setValue(value: T) {
        content = value
    }
    
    fun isPositive(): Boolean {
        return content.toDouble() > 0
    }
}

// 사용 예시
val intBox = NumberBox<Int>(42)
val doubleBox = NumberBox<Double>(3.14)
// val stringBox = NumberBox<String>("hello") // 컴파일 에러: String은 Number의 하위 타입이 아님
```

### 2-3. 다중 타입 제약

```kotlin
// 여러 타입 제약을 동시에 적용
class DataProcessor<T>(
    private val data: T
) where T : CharSequence, T : Comparable<T> {
    
    fun getLength(): Int = data.length
    
    fun isGreaterThan(other: T): Boolean = data > other
}

// 사용 예시
val processor = DataProcessor<String>("hello")
println(processor.getLength()) // 5
println(processor.isGreaterThan("hi")) // true
```

---

## 3. 제네릭 함수

### 3-1. 기본 제네릭 함수

```kotlin
// 제네릭 함수 정의
fun <T> swap(array: Array<T>, index1: Int, index2: Int) {
    val temp = array[index1]
    array[index1] = array[index2]
    array[index2] = temp
}

// 사용 예시
val intArray = arrayOf(1, 2, 3, 4, 5)
swap(intArray, 0, 4)
println(intArray.contentToString()) // [5, 2, 3, 4, 1]

val stringArray = arrayOf("A", "B", "C")
swap(stringArray, 0, 2)
println(stringArray.contentToString()) // [C, B, A]
```

### 3-2. 타입 제약이 있는 제네릭 함수

```kotlin
// Comparable을 구현한 타입만 허용
fun <T : Comparable<T>> findMax(list: List<T>): T? {
    if (list.isEmpty()) return null
    
    var max = list[0]
    for (item in list) {
        if (item > max) max = item
    }
    return max
}

// 사용 예시
val numbers = listOf(3, 1, 4, 1, 5, 9, 2, 6)
val maxNumber = findMax(numbers) // 9

val strings = listOf("apple", "banana", "cherry")
val maxString = findMax(strings) // "cherry"
```

### 3-3. 다중 타입 파라미터

```kotlin
// 여러 타입 파라미터를 사용하는 함수
fun <T, R> transformList(
    list: List<T>,
    transformer: (T) -> R
): List<R> {
    return list.map(transformer)
}

// 사용 예시
val numbers = listOf(1, 2, 3, 4, 5)
val strings = transformList(numbers) { it.toString() }
println(strings) // [1, 2, 3, 4, 5]

val doubled = transformList(numbers) { it * 2 }
println(doubled) // [2, 4, 6, 8, 10]
```

---

## 4. 제네릭 인터페이스

### 4-1. 기본 제네릭 인터페이스

```kotlin
// 제네릭 인터페이스 정의
interface Repository<T> {
    fun save(item: T)
    fun findById(id: String): T?
    fun findAll(): List<T>
    fun delete(id: String)
}

// 구현 클래스
class UserRepository : Repository<User> {
    private val users = mutableMapOf<String, User>()
    
    override fun save(item: User) {
        users[item.id] = item
    }
    
    override fun findById(id: String): User? {
        return users[id]
    }
    
    override fun findAll(): List<User> {
        return users.values.toList()
    }
    
    override fun delete(id: String) {
        users.remove(id)
    }
}

// 사용 예시
data class User(val id: String, val name: String, val email: String)
val userRepo = UserRepository()
userRepo.save(User("1", "김철수", "kim@example.com"))
```

### 4-2. 제네릭 인터페이스의 타입 제약

```kotlin
// 특정 타입만 허용하는 제네릭 인터페이스
interface NumberProcessor<T : Number> {
    fun process(value: T): T
    fun compare(value1: T, value2: T): Int
}

class IntProcessor : NumberProcessor<Int> {
    override fun process(value: Int): Int = value * 2
    
    override fun compare(value1: Int, value2: Int): Int = value1.compareTo(value2)
}

class DoubleProcessor : NumberProcessor<Double> {
    override fun process(value: Double): Double = value * 1.5
    
    override fun compare(value1: Double, value2: Double): Int = value1.compareTo(value2)
}
```

---

## 5. 제네릭의 타입 변성 (Variance)

### 5-1. 불변 (Invariant)

기본적으로 제네릭은 불변입니다:

```kotlin
class Box<T>(var value: T)

// 무공변으로 인한 제약
val stringBox = Box<String>("hello")
// val anyBox: Box<Any> = stringBox // 컴파일 에러: 타입 불일치
```

### 5-2. 공변 (Covariant) - out

`out` 키워드를 사용하면 공변을 가집니다:

```kotlin
// 공변 제네릭 클래스
class ReadOnlyBox<out T>(private val value: T) {
    fun getValue(): T = value
    // fun setValue(value: T) // 컴파일 에러: out 위치에서만 사용 가능
}

// 사용 예시
val stringBox = ReadOnlyBox<String>("hello")
val anyBox: ReadOnlyBox<Any> = stringBox // 가능: String은 Any의 하위 타입
```

### 5-3. 반공변 (Contravariant) - in

`in` 키워드를 사용하면 반공변을 가집니다:

```kotlin
// 반공변 제네릭 클래스
class WriteOnlyBox<in T> {
    private var value: T? = null
    
    fun setValue(value: T) {
        this.value = value
    }
    // fun getValue(): T // 컴파일 에러: in 위치에서만 사용 가능
}

// 사용 예시
val anyBox = WriteOnlyBox<Any>()
val stringBox: WriteOnlyBox<String> = anyBox // 가능: Any는 String의 상위 타입
```

### 5-4. 활용

```kotlin
// 공변 활용: 읽기 전용 컬렉션
interface ReadOnlyRepository<out T> {
    fun findById(id: String): T?
    fun findAll(): List<T>
}

// 반공변 활용: 쓰기 전용 컬렉션
interface WriteOnlyRepository<in T> {
    fun save(item: T)
    fun delete(id: String)
}

// 불변: 읽기/쓰기 모두 가능
interface Repository<T> {
    fun save(item: T)
    fun findById(id: String): T?
    fun findAll(): List<T>
    fun delete(id: String)
}
```

---

## 6. 제네릭의 타입 소거 (Type Erasure)

### 6-1. 타입 소거의 개념

Kotlin의 제네릭은 런타임에 타입 정보가 소거됩니다:

```kotlin
val stringList: List<String> = listOf("hello", "world")
val intList: List<Int> = listOf(1, 2, 3)

// 런타임에는 둘 다 List로만 인식됨
println(stringList.javaClass) // class java.util.Arrays$ArrayList
println(intList.javaClass)    // class java.util.Arrays$ArrayList
```

### 6-2. 타입 소거로 인한 제약

```kotlin
// 불가능: 런타임에 제네릭 타입 정보를 알 수 없음
fun <T> checkType(list: List<T>): Boolean {
    return list is List<String> // 컴파일 에러
}

// 가능: 구체적인 타입으로 제한
fun checkStringList(list: List<*>): Boolean {
    return list.all { it is String }
}
```

### 6-3. reified를 활용한 해결책

```kotlin
// reified를 사용한 타입 안전한 체크
inline fun <reified T> checkType(value: Any): Boolean {
    return value is T
}

// 사용 예시
println(checkType<String>("hello")) // true
println(checkType<Int>("hello"))    // false
```

---

## 7. 실무 활용 예시

### 7-1. API 응답 처리

```kotlin
// 제네릭을 활용한 API 응답 래퍼
sealed class ApiResult<out T> {
    data class Success<T>(val data: T) : ApiResult<T>()
    data class Error(val message: String) : ApiResult<Nothing>()
    object Loading : ApiResult<Nothing>()
}

// API 서비스 인터페이스 - 구체적인 타입으로 제한
interface ApiService {
    suspend fun getUsers(): ApiResult<List<User>>
    suspend fun getOrders(): ApiResult<List<Order>>
}

// 사용 예시
class UserApiService : ApiService {
    override suspend fun getUsers(): ApiResult<List<User>> {
        return ApiResult.Success(listOf(User("1", "김철수", "kim@example.com")))
    }
    
    override suspend fun getOrders(): ApiResult<List<Order>> {
        return ApiResult.Success(listOf(Order("1", "상품A")))
    }
}

// 데이터 클래스
data class User(val id: String, val name: String, val email: String)
data class Order(val id: String, val productName: String)
```

### 7-2. 이벤트 버스

```kotlin
// 제네릭을 활용한 타입 안전한 이벤트 버스
class EventBus {
    private val listeners = mutableMapOf<Class<*>, MutableList<(Any) -> Unit>>()
    
    fun <T> subscribe(eventType: Class<T>, listener: (T) -> Unit) {
        listeners.getOrPut(eventType) { mutableListOf() }.add { event ->
            listener(event as T)
        }
    }
    
    fun <T> post(event: T) {
        val eventType = event::class.java
        listeners[eventType]?.forEach { listener ->
            listener(event)
        }
    }
}

// 사용 예시
data class UserEvent(val userId: String)
data class OrderEvent(val orderId: String)

val eventBus = EventBus()
eventBus.subscribe(UserEvent::class.java) { event ->
    println("사용자 이벤트: ${event.userId}")
}
```

### 7-3. 캐시 매니저

```kotlin
// 제네릭을 활용한 타입 안전한 캐시 매니저
class CacheManager {
    private val cache = mutableMapOf<String, Any>()
    
    fun <T> put(key: String, value: T) {
        cache[key] = value
    }
    
    @Suppress("UNCHECKED_CAST")
    fun <T> get(key: String): T? {
        return cache[key] as? T
    }
    
    fun remove(key: String) {
        cache.remove(key)
    }
    
    fun clear() {
        cache.clear()
    }
}

// 사용 예시
val cacheManager = CacheManager()
cacheManager.put("user", User("1", "김철수"))
cacheManager.put("settings", mapOf("theme" to "dark"))

val user: User? = cacheManager.get("user")
val settings: Map<String, String>? = cacheManager.get("settings")
```

### 7-4. ViewModel 팩토리

```kotlin
// 제네릭을 활용한 ViewModel 팩토리
class ViewModelFactory<T : ViewModel>(
    private val creator: () -> T
) : ViewModelProvider.Factory {
    
    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return creator() as T
    }
}

// 사용 예시
class MainViewModel : ViewModel() {
    // ViewModel 구현
}

val factory = ViewModelFactory { MainViewModel() }
val viewModel: MainViewModel = ViewModelProvider(this, factory)[MainViewModel::class.java]
```


## 참고 자료

- [Kotlin 공식 문서: Generics](https://kotlinlang.org/docs/generics.html)
- [Kotlin 공식 문서: Type Projections](https://kotlinlang.org/docs/generics.html#type-projections)
- 『Kotlin in Action』 Chapter 9: 제네릭스
- 『Effective Kotlin』 Item 24: 제네릭 타입과 variance 한정자를 활용하라 
