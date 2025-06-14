# SharedFlow vs Channel

> 안드로이드 앱 개발에서는 사용자 인터랙션이나 일회성 이벤트를 처리할 때 `Channel`이나 `SharedFlow`를 자주 사용하게 됩니다.  
> 특히, MVI 패턴이나 Compose 환경에서는 `Effect` 이벤트 처리 시 어떤 방식을 선택할지 고민되는 경우가 많습니다.  

> 이번 글에서는 `Channel`과 `SharedFlow`의 개념, 차이점, 사용 사례를 정리하고, 언제 어떤 것을 쓰는 것이 적절한지 분석해보겠습니다.

## 1. Channel과 SharedFlow의 기본 개념

### Channel

* **Producer–Consumer 큐**
* **한 번만 소비 가능 (단방향, 1:1)**
* `send()`로 전달 → `receive()`로 수신
* 소비되지 않으면 대기하거나 버퍼링

### SharedFlow

* **Broadcast Flow** (1\:N)
* 여러 collect 지점에서 동시에 수신 가능
* 항상 열려 있는 Hot Flow
* `emit()`으로 전달 → `collect {}`로 수신
* `replay`, `bufferCapacity`, `onBufferOverflow` 등 설정 가능

## 2. 주요 차이점 비교

| 항목        | Channel        | SharedFlow                       |
| --------- | -------------- | -------------------------------- |
| 소비자 수     | 단일 소비자 (1:1)   | 다수 소비자 (1\:N)                    |
| 이벤트 소비 방식 | 소비하면 사라짐       | 모든 구독자에게 전달                      |
| 종료 여부     | `close()` 가능   | 항상 열려 있음                         |
| 예외 전파     | 예외 발생 시 코루틴 취소 | 기본적으로 예외 전파 없음                   |
| 버퍼 처리     | 버퍼 크기 지정 가능    | `replay`, `bufferCapacity` 설정 가능 |

## 3. 언제 무엇을 사용할까?

### Channel이 적합한 경우

* **Effect(일회성 이벤트) 처리**
* **Toast, Navigation 등 단일 이벤트 처리**
* **UI 단위 Action 처리 (버튼 클릭, 폼 제출 등)**
* **Backpressure 대응**이 필요한 경우 (버퍼 전략 활용)

```kotlin
private val _uiEvent = Channel<UiEvent>(Channel.UNLIMITED)
val uiEvent = _uiEvent.receiveAsFlow()

fun sendEffect(event: UiEvent) {
    viewModelScope.launch {
        _uiEvent.send(event)
    }
}
```

### SharedFlow가 적합한 경우

* **여러 collect 지점에 이벤트를 전달해야 하는 경우**
* **Compose 화면 여러 곳에서 동시에 수신해야 하는 경우**
* **`replay` 설정으로 최신 값을 유지해야 하는 경우**

```kotlin
private val _event = MutableSharedFlow<AppEvent>(replay = 0)
val event = _event.asSharedFlow()

fun emitEvent(e: AppEvent) {
    viewModelScope.launch {
        _event.emit(e)
    }
}
```

## 참고자료

* [공식 문서 - Kotlin Coroutine Channel](https://kotlinlang.org/docs/channels.html)
* [공식 문서 - SharedFlow](https://kotlinlang.org/docs/flow.html#shared-flows)
