# Kotlin reified

> Kotlin의 `reified`는 제네릭 타입 정보를 런타임까지 유지할 수 있게 하는 키워드입니다.   
> 일반적인 제네릭의 타입 소거(Type Erasure) 문제를 해결하고, 타입 안전한 코드를 작성할 수 있게 도와줍니다.   
> 이 글에서는 `reified`의 개념, 사용법, 활용 예를 정리해보겠습니다.

---

## 1. 왜 reified가 필요한가?

### 1-1. 제네릭의 타입 소거 문제

Java와 Kotlin의 제네릭은 **타입 소거(Type Erasure)**를 사용합니다.  
이는 컴파일 시점에만 타입 정보를 확인하고, 런타임에는 타입 정보가 사라진다는 의미입니다.

```kotlin
// 컴파일 시점에는 타입 정보가 있지만
val stringList: List<String> = listOf("hello", "world")

// 런타임에는 타입 정보가 사라짐
if (stringList is List<String>) { // 컴파일 에러
    println("String 리스트입니다")
}
```

### 1-2. reified의 해결책

`reified` 키워드를 사용하면 제네릭 타입 정보를 런타임까지 유지할 수 있습니다.

```kotlin
inline fun <reified T> checkType(value: Any): Boolean {
    return value is T // 런타임에 타입 체크 가능
}

// 사용 예시
val result = checkType<String>("hello") // true
val result2 = checkType<Int>("hello")   // false
```

---

## 2. reified의 제약사항

`reified`는 다음과 같은 제약사항이 있습니다:

- **inline 함수에서만 사용 가능**: `reified`는 `inline` 키워드와 함께 사용해야 합니다.
- **함수 파라미터로만 사용**: 클래스 레벨에서는 사용할 수 없습니다.
- **타입 파라미터에만 적용**: 일반 파라미터에는 사용할 수 없습니다.

```kotlin
// 올바른 사용법
inline fun <reified T> createInstance(): T {
    return T::class.java.newInstance()
}

// 잘못된 사용법
class MyClass<reified T> { // 컴파일 에러
    fun doSomething() {}
}
```

---

## 3. 실무 활용 예시

### 3-1. JSON 직렬화/역직렬화

Gson이나 Jackson 같은 JSON 라이브러리에서 타입 안전한 직렬화를 구현할 수 있습니다.

```kotlin
inline fun <reified T> Gson.fromJson(json: String): T {
    return this.fromJson(json, T::class.java)
}

// 사용 예시
data class User(val name: String, val age: Int)

val json = """{"name": "신짱구", "age": 5}"""
val user: User = gson.fromJson(json) // 타입 추론으로 User 타입 자동 지정
```

### 3-2. SharedPreferences에서 타입 안전한 데이터 저장

```kotlin
inline fun <reified T> SharedPreferences.getTypedValue(key: String, defaultValue: T): T {
    return when (T::class) {
        String::class -> getString(key, defaultValue as String) as T
        Int::class -> getInt(key, defaultValue as Int) as T
        Boolean::class -> getBoolean(key, defaultValue as Boolean) as T
        else -> throw IllegalArgumentException("지원하지 않는 타입입니다: ${T::class}")
    }
}

// 사용 예시
val userName: String = prefs.getTypedValue("user_name", "기본값")
val userAge: Int = prefs.getTypedValue("user_age", 0)
```

### 3-3. ViewModel 생성

Android의 ViewModel을 타입 안전하게 생성할 수 있습니다.

```kotlin
inline fun <reified T : ViewModel> Fragment.getViewModel(): T {
    return ViewModelProvider(this)[T::class.java]
}

// 사용 예시
class MainFragment : Fragment() {
    private val viewModel: MainViewModel by lazy { getViewModel() }
}
```

### 3-4. Intent에서 데이터 추출

```kotlin
inline fun <reified T> Intent.getTypedExtra(key: String): T? {
    return when (T::class) {
        String::class -> getStringExtra(key) as T?
        Int::class -> getIntExtra(key, -1) as T?
        Boolean::class -> getBooleanExtra(key, false) as T?
        else -> null
    }
}

// 사용 예시
val userName: String? = intent.getTypedExtra("user_name")
val userAge: Int? = intent.getTypedExtra("user_age")
```

---

## 4. 고급 활용 기법

### 4-1. 타입 기반 팩토리 패턴

```kotlin
inline fun <reified T> createInstance(vararg args: Any): T {
    return T::class.java.getDeclaredConstructor(*args.map { it::class.java }.toTypedArray())
        .newInstance(*args)
}

// 사용 예시
data class Person(val name: String, val age: Int)
val person: Person = createInstance("김철수", 25)
```

### 4-2. 타입 안전한 이벤트 버스

```kotlin
class EventBus {
    private val listeners = mutableMapOf<Class<*>, MutableList<(Any) -> Unit>>()
    
    inline fun <reified T> subscribe(noinline listener: (T) -> Unit) {
        val type = T::class
        listeners.getOrPut(type) { mutableListOf() }.add { event ->
            listener(event as T)
        }
    }
    
    inline fun <reified T> post(event: T) {
        val type = T::class
        listeners[type]?.forEach { listener ->
            listener(event)
        }
    }
}

// 사용 예시
data class UserEvent(val userId: String)
val eventBus = EventBus()

eventBus.subscribe<UserEvent> { event ->
    println("사용자 이벤트: ${event.userId}")
}

eventBus.post(UserEvent("user123"))
```

### 4-3. 타입 기반 캐싱

```kotlin
class TypeSafeCache {
    private val cache = mutableMapOf<Class<*>, Any>()
    
    inline fun <reified T> get(): T? {
        return cache[T::class] as T?
    }
    
    inline fun <reified T> set(value: T) {
        cache[T::class] = value
    }
}

// 사용 예시
val cache = TypeSafeCache()
cache.set("hello")
val cachedString: String? = cache.get<String>()
```

---

## 5. 성능 고려사항

### 5-1. 인라인 함수의 특성

`reified`는 `inline` 함수에서만 사용되므로, 함수가 호출되는 모든 지점에 코드가 복사됩니다.  
이는 다음과 같은 영향을 미칩니다:

- **코드 크기 증가**: 함수가 여러 곳에서 호출되면 바이너리 크기가 커집니다.
- **컴파일 시간 증가**: 인라인 처리로 인해 컴파일 시간이 늘어날 수 있습니다.

### 5-2. 최적화 팁

```kotlin
// 좋은 예: 자주 사용되는 타입 체크
inline fun <reified T> isType(value: Any): Boolean {
    return value is T
}

// 피해야 할 예: 복잡한 로직을 reified로 처리
inline fun <reified T> complexOperation(value: T): T {
    // 복잡한 로직...
    return value
}
```

---

## 6. 주의사항과 한계

### 6-1. 타입 파라미터의 한계

```kotlin
// 불가능: 제네릭 타입 파라미터는 reified로 처리할 수 없음
inline fun <reified T> createList(): List<T> {
    return listOf() // T의 구체적인 타입을 알 수 없음
}

// 가능: 구체적인 타입으로 제한
inline fun <reified T> createTypedList(): List<T> {
    return when (T::class) {
        String::class -> listOf("") as List<T>
        Int::class -> listOf(0) as List<T>
        else -> emptyList()
    }
}
```

### 6-2. 런타임 타입 체크의 한계

```kotlin
// 불가능: 제네릭 타입의 구체적인 정보는 런타임에 알 수 없음
inline fun <reified T> getGenericType(): Class<*> {
    return T::class.java // List<String>의 경우 List::class.java만 반환
}
```

---

## 7. 실무에서의 베스트 프랙티스

### 7-1. 타입 안전성 우선

```kotlin
// 좋은 예: 타입 안전성 보장
inline fun <reified T> SharedPreferences.getTypedValue(key: String): T? {
    return when (T::class) {
        String::class -> getString(key, null) as T?
        Int::class -> if (contains(key)) getInt(key, 0) as T? else null
        else -> null
    }
}

// 피해야 할 예: 타입 캐스팅 위험
inline fun <reified T> SharedPreferences.getUnsafeValue(key: String): T? {
    return getString(key, null) as T? // 런타임 에러 가능성
}
```

### 7-2. 명확한 에러 처리

```kotlin
inline fun <reified T> createInstanceOrNull(): T? {
    return try {
        T::class.java.newInstance()
    } catch (e: Exception) {
        null
    }
}
```

---

## 참고 자료

- [Kotlin 공식 문서: Inline Functions](https://kotlinlang.org/docs/inline-functions.html)
- [Kotlin 공식 문서: Reified Type Parameters](https://kotlinlang.org/docs/inline-functions.html#reified-type-parameters)
- 『Kotlin in Action』 Chapter 9: 제네릭스
- 『Effective Kotlin』 Item 47: 인라인 함수의 타입 파라미터를 reified로 만들라 
