# runBlocking, 왜 함부로 쓰면 안 될까?

> `runBlocking`은 `suspend` 함수를 호출할 수 없는 상황에서 코루틴을 동기적으로 실행할 수 있게 해주는 도구입니다.  
> 하지만 안드로이드 앱 코드에 무분별하게 사용하면 UI 정지, 구조적 버그, 데드락 등 심각한 문제가 발생할 수 있습니다.  
> 이 글에서는 언제 `runBlocking`을 사용해야 하는지, 그리고 어떤 상황에서 피해야 하는지 알아보겠습니다.  

## 1. runBlocking이란?

`runBlocking`은 코루틴을 **동기적으로** 실행할 수 있게 해주는 코루틴 빌더입니다.  
이름 그대로 현재 스레드를 블로킹하며 내부 코드를 실행합니다.  
주로 `main()` 함수나 테스트 코드에서 사용되며,  
`suspend` 함수를 호출할 수 없는 상황에서 강제로 코루틴 컨텍스트를 만들기 위해 사용됩니다.

```kotlin
fun main() = runBlocking {
    val result = fetchData()
    println(result)
}
```

> 핵심은, "runBlocking은 스레드를 점유(block)한다"는 점입니다.

## 2. 왜 함부로 사용하면 안 될까요?

### 2.1. UI 스레드에서 블로킹 → ANR 유발

안드로이드에서 `runBlocking`을 메인 스레드에서 실행하면 UI를 정지시켜 **ANR(Application Not Responding)** 을 유발할 수 있습니다.

```kotlin
fun onClick() = runBlocking {
    delay(5000) // UI 스레드 정지
}
```

이렇게 작성하면 버튼 클릭 후 5초간 앱이 응답하지 않게 됩니다.  
이는 사용자 경험에 심각한 악영향을 줍니다.

### 2.2. 구조적 동시성과의 충돌

코루틴은 부모-자식 관계를 기반으로 한 구조적 동시성을 제공합니다.  
하지만 `runBlocking`은 **새로운 최상위 코루틴 컨텍스트**를 생성하여 기존 구조를 무시하게 됩니다.

```kotlin
viewModelScope.launch {
    // suspend 함수 내에서 runBlocking 호출 시
    runBlocking {
        // 새로운 코루틴 컨텍스트
    }
}
```

이렇게 되면 `viewModelScope`가 취소되더라도 `runBlocking` 내부 코드는 취소되지 않아 버그가 발생할 수 있습니다.

### 2.3. 데드락 발생 가능성

다음과 같은 상황에서는 `runBlocking`이 데드락(deadlock)을 유발할 수 있습니다.

```kotlin
val dispatcher = Executors.newSingleThreadExecutor().asCoroutineDispatcher()

runBlocking(dispatcher) {
    launch { delay(1000); println("child") }
}
```

`runBlocking`이 dispatcher의 유일한 스레드를 점유하고 있기 때문에, 내부의 `launch`가 실행될 스레드가 없어 데드락이 발생하게 됩니다.

## 3. 언제 사용해야 할까요?

* `main()` 함수 진입점
* 테스트 코드에서 `suspend` 함수를 호출해야 할 때

```kotlin
@Test
fun testSomething() = runBlocking {
    val result = repository.getData()
    assertEquals(expected, result)
}
```

이처럼 코루틴 환경이 보장되지 않는 코드 진입점에서만 사용하는 것이 바람직합니다.

## 4. 대안은 무엇일까요?

* 애플리케이션 코드에서는 `runBlocking` 대신 `CoroutineScope.launch`, `async`, `withContext` 등을 사용하는 것이 좋습니다.
* 구조적 동시성을 유지할 수 있는 방식으로 설계하는 것이 중요합니다.

```kotlin
viewModelScope.launch {
    val data = repository.getData()
    _state.value = data
}
```

이처럼 비동기 처리를 구성하면 UI가 멈추지 않고, 구조적으로도 안정적인 코드를 만들 수 있습니다.
