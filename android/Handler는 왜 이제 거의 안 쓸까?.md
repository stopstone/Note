# Handler는 왜 이제 거의 안 쓸까?

> 안드로이드에서 Handler는 오랫동안 UI 스레드 작업을 지연시키거나, 다른 스레드와 통신하기 위한 핵심 도구였습니다.  
> Jetpack 라이브러리와 Coroutine이 등장하면서 Handler의 역할은 점차 축소되었고, 이제는 거의 사용되지 않는 기술로 여겨지고 있습니다.  
> 이 글에서는 Handler의 동작 방식과 함께, 왜 더 이상 많이 사용되지 않는지, 그리고 어떤 대체 기술들이 있는지를 살펴보겠습니다.  

## 1. Handler의 역할

Handler는 내부적으로 **Looper + MessageQueue** 구조를 기반으로 동작합니다.    
메시지를 큐에 넣고, 루프를 돌면서 메시지를 하나씩 꺼내 처리하는 방식이에요.  

대표적인 사용 예시는 다음과 같습니다:

```kotlin
val handler = Handler(Looper.getMainLooper())
handler.postDelayed({
    // 일정 시간 후 실행되는 작업
}, 2000L)
```

Handler는 다음과 같은 경우에 유용했습니다:

* UI 스레드에서의 작업 지연 (ex. splash 화면 딜레이)
* 백그라운드 스레드와 UI 스레드 간의 통신
* `HandlerThread`를 통해 별도 쓰레드에 메시지를 보내는 구조 구현

## 2. Handler가 이제는 잘 안 쓰이는 이유

### 구조적 복잡성과 에러 발생 가능성

* 메시지 큐 구조 자체가 **순서 꼬임**이나 **메시지 재사용 문제**, **동시성 이슈**를 유발할 수 있습니다.
* Runnable에 Activity나 Fragment의 Context를 참조하면 **Memory Leak**이 발생할 수 있습니다.

### 생명주기를 고려하지 않음

* Handler는 작업이 등록되면 무조건 실행되므로, 작업 등록 후 Activity가 종료되더라도 **작업이 계속 실행**됩니다.
* 이로 인해 **Crash**나 **불필요한 리소스 사용**이 발생할 수 있습니다.

### 대체 기술의 등장

Jetpack과 Kotlin Coroutine의 등장 이후, Handler가 하던 역할은 더 안전하고 직관적인 방식으로 대체되었습니다:

* **Coroutine + viewModelScope / lifecycleScope**
* **LiveData, StateFlow를 통한 상태 전달**
* **Jetpack Compose에서는 LaunchedEffect, rememberCoroutineScope**

## 3. Handler → Coroutine으로 바꾸는 예시

### 기존 Handler 방식

```kotlin
val handler = Handler(Looper.getMainLooper())
handler.postDelayed({
    // 작업
}, 2000L)
```

### Coroutine + ViewModelScope 방식

```kotlin
viewModelScope.launch {
    delay(2000L)
    // 작업
}
```

### Compose에서 LaunchedEffect 사용

```kotlin
LaunchedEffect(Unit) {
    delay(2000L)
    // 작업
}
```

## 4. 여전히 Handler가 필요한 경우

Handler는 완전히 사라진 기술은 아니며, 아래와 같은 상황에서는 여전히 필요할 수 있습니다:

* 외부 라이브러리 또는 **레거시 코드** 유지보수 중
* 커스텀 **HandlerThread**를 사용하는 구조
* **JNI/NDK** 연동 등 저수준 시스템과의 통신 필요 시

---

## 참고자료

* [Handler - Android Developers](https://developer.android.com/reference/android/os/Handler)
* [Kotlin Coroutines - Android Developers](https://developer.android.com/kotlin/coroutines)
* [LaunchedEffect - Compose](https://developer.android.com/jetpack/compose/side-effects#launchedeffect)
