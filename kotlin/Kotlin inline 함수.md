# Kotlin inline 함수

> Kotlin의 inline 함수는 함수 호출 시 성능 최적화를 위해 사용되는 기능입니다.  
> 특히 고차 함수나 람다를 자주 사용하는 상황에서 유용하게 활용할 수 있습니다.  
> 본 문서에서는 inline 함수의 전반적인 내용을 설명드립니다.

## 1. 왜 inline 함수가 생겼을까요?

Kotlin에서는 고차 함수를 사용할 때 람다 표현식이 객체로 변환되어 heap에 할당되고, 함수 호출이 발생합니다.  
이러한 동작은 불필요한 객체 생성 및 호출 오버헤드로 인해 성능 저하를 일으킬 수 있습니다.

이를 해결하기 위해 Kotlin은 `inline` 키워드를 제공하여,  
**컴파일 시 함수 본문을 호출 지점에 삽입**함으로써 이러한 비용을 줄일 수 있도록 설계하였습니다.

## 2. inline 함수의 장점

* **성능 향상**: 함수 호출과 람다 객체 생성을 제거하여 실행 속도를 개선할 수 있습니다.
* **비지역적 반환 허용**: 람다 내부에서 외부 함수를 `return`할 수 있습니다.
* **실행 시간 타입 유지 (reified)**: 제네릭 타입 정보를 런타임까지 유지할 수 있어, 타입 정보를 명시적으로 다룰 수 있습니다.

### reified 예시

```kotlin
inline fun <reified T> Gson.fromJson(json: String): T =
    this.fromJson(json, T::class.java)

val json = "{\"name\":\"지석오빠\"}"
val user: User = gson.fromJson(json)
```

이처럼 `reified` 키워드는 inline 함수와 함께 사용되어, 제네릭 타입 정보를 런타임에도 유지할 수 있게 해줍니다.  
이를 통해 리플렉션 없이도 타입 기반 로직을 구현할 수 있습니다.

## 3. inline 함수의 동작 방식

inline 함수는 컴파일러가 해당 함수를 호출한 위치에 함수 본문을 복사하여 삽입합니다.  
이에 따라 실제 호출이 발생하지 않으며, 함수 호출에 필요한 스택 프레임도 생성되지 않습니다.  
람다 역시 객체로 변환되지 않고 코드로 직접 삽입됩니다.

## 4. 예시: inline과 성능 최적화 - measureTimeMillis 함수

Kotlin 표준 라이브러리에서 제공하는 `measureTimeMillis` 함수는 코드 블록이 수행되는 데 걸리는 시간을 측정할 때 사용됩니다.  
이 함수는 `inline`으로 정의되어 있어 람다 블록이 객체화되지 않고 호출 지점에 직접 삽입됩니다.

```kotlin
inline fun measureTimeMillis(block: () -> Unit): Long {
    val start = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - start
}
```

### 사용 예시

```kotlin
val elapsed = measureTimeMillis {
    Thread.sleep(100)
}
println("수행 시간: ${elapsed}ms")
```

### 컴파일 후 예상 코드 (단순화된 형태)

```kotlin
val elapsed = run {
    val start = System.currentTimeMillis()
    Thread.sleep(100)
    System.currentTimeMillis() - start
}
println("수행 시간: ${elapsed}ms")
```

````

이 예제에서 `run` 함수는 inline으로 선언되어 있어, 내부 람다 블록이 객체화되지 않고 직접 삽입됩니다.

### 컴파일 후 예상 코드 (단순화된 형태)
```kotlin
val result = {
    val a = 10
    val b = 20
    a + b
}()
println(result)
````

## 5. noinline과 crossinline

inline 함수에서 모든 람다 파라미터가 자동으로 인라인되는 것은 아닙니다.  
이 경우를 제어하기 위한 키워드로 `noinline`과 `crossinline`이 있습니다.

### noinline

특정 람다 파라미터를 인라인하지 않고, 객체로 유지하고 싶을 때 사용합니다.

```kotlin
inline fun sample(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    inlined()
    notInlined()
}
```

### crossinline

람다 안에서 비지역적 `return`을 방지하고 싶을 때 사용합니다.

```kotlin
inline fun execute(crossinline block: () -> Unit) {
    val runnable = Runnable { block() }
    runnable.run()
}
```

## 참고 자료

[Kotlin 공식 문서: Inline Functions](https://kotlinlang.org/docs/inline-functions.html)  
[Kotlin in Action 6장: 고차 함수와 inline 키워드]
