# Channel send와 trySend의 차이점

> Channel을 사용할 때 데이터를 전송하는 방법으로 `send()`와 `trySend()`를 사용할 수 있습니다.  
> 모두 Channel에 데이터를 전송하지만, 동작 방식과 사용 시나리오가 다릅니다.  
> 이번 글에서는 차이점, 장단점, 그리고 언제 어떤 것을 사용해야 하는지 알아보겠습니다.

## 1. send()와 trySend()의 기본 개념

### send()
* **Suspending 함수** - Channel이 가득 찰 경우 대기
* Channel이 닫혀있으면 `ClosedSendChannelException` 발생
* 안전한 데이터 전송을 보장

### trySend()
* **Non-suspending 함수** - 즉시 결과 반환
* Channel이 가득 찰 경우 `ChannelResult.Failed` 반환
* Channel이 닫혀있으면 `ChannelResult.Closed` 반환

## 2. 동작 방식 비교

### send() 동작 방식

```kotlin
val channel = Channel<Int>(capacity = 2)

// Channel이 가득 찰 때까지 대기
launch {
    channel.send(1) // 즉시 성공
    channel.send(2) // 즉시 성공
    channel.send(3) // Channel이 가득 차서 대기 (suspend)
    println("모든 데이터 전송 완료")
}
```

### trySend() 동작 방식

```kotlin
val channel = Channel<Int>(capacity = 2)

// 즉시 결과 반환
launch {
    val result1 = channel.trySend(1) // ChannelResult.Success
    val result2 = channel.trySend(2) // ChannelResult.Success
    val result3 = channel.trySend(3) // ChannelResult.Failed (Channel이 가득 참)
    
    println("결과: $result1, $result2, $result3")
}
```

## 3. 상세 비교표

| 항목 | send() | trySend() |
|------|--------|-----------|
| **함수 타입** | Suspend 함수 | 일반 함수 |
| **대기 여부** | Channel이 가득 찰 경우 대기 | 즉시 결과 반환 |
| **예외 처리** | Channel 닫힘 시 예외 발생 | ChannelResult로 결과 반환 |
| **사용 시점** | 반드시 데이터를 전송해야 할 때 | 데이터 전송 실패가 허용될 때 |
| **코루틴 스코프** | 코루틴 스코프 내에서만 사용 | 어디서든 사용 가능 |

## 4. 실제 사용 예제

### 4.1 send() 사용 예제

```kotlin
class EventSender {
    private val eventChannel = Channel<Event>(Channel.UNLIMITED)
    
    // 이벤트를 반드시 전송해야 하는 경우
    suspend fun sendCriticalEvent(event: Event) {
        try {
            eventChannel.send(event) // 전송이 성공할 때까지 대기
            println("중요한 이벤트 전송 성공: $event")
        } catch (e: ClosedSendChannelException) {
            println("Channel이 닫혀있음: $e")
        }
    }
    
    // Channel 수신자
    fun startEventReceiver() {
        viewModelScope.launch {
            for (event in eventChannel) {
                handleEvent(event)
            }
        }
    }
}
```

### 4.2 trySend() 사용 예제

```kotlin
class UiEventManager {
    private val uiEventChannel = Channel<UiEvent>(capacity = 10)
    
    // UI 이벤트 전송 (실패해도 괜찮은 경우)
    fun sendUiEvent(event: UiEvent) {
        when (val result = uiEventChannel.trySend(event)) {
            is ChannelResult.Success -> {
                println("UI 이벤트 전송 성공: $event")
            }
            is ChannelResult.Failed -> {
                println("UI 이벤트 전송 실패 - Channel이 가득 참: $event")
                // 실패한 이벤트를 로그에 기록하거나 다른 처리
                logFailedEvent(event)
            }
            is ChannelResult.Closed -> {
                println("Channel이 닫혀있음")
            }
        }
    }
    
    // Channel 수신자
    fun startUiEventReceiver() {
        viewModelScope.launch {
            for (event in uiEventChannel) {
                handleUiEvent(event)
            }
        }
    }
}
```

## 5. 장단점 분석

### send()의 장단점

#### 장점
* **데이터 전송 보장**: Channel이 가득 찰 때까지 대기하여 데이터 손실 방지
* **예외 처리 명확**: Channel 닫힘 시 명확한 예외 발생
* **코드 가독성**: 의도가 명확하게 드러남 (반드시 전송해야 함)

#### 단점
* **코루틴 스코프 필요**: suspend 함수이므로 코루틴 스코프 내에서만 사용
* **블로킹 가능성**: Channel이 가득 차면 무한정 대기 가능
* **메모리 사용량**: 대기 중인 코루틴이 메모리를 점유

### trySend()의 장단점

#### 장점
* **비블로킹**: 즉시 결과를 반환하여 성능상 유리
* **어디서든 사용 가능**: 일반 함수이므로 코루틴 스코프 외부에서도 사용
* **유연한 처리**: 실패 시 다른 대안 처리 가능
* **Backpressure 처리**: Channel이 가득 찰 때 적절한 처리 가능

#### 단점
* **데이터 손실 가능**: Channel이 가득 찰 경우 데이터 전송 실패
* **복잡한 결과 처리**: ChannelResult를 통한 결과 처리 필요
* **의도 불분명**: 반드시 전송해야 하는지 선택적 전송인지 불분명

## 6. 언제 무엇을 사용할까?

### send()를 사용해야 하는 경우

* **중요한 데이터 전송**: 데이터 손실이 허용되지 않는 경우
* **로그 이벤트 전송**: 모든 로그를 반드시 기록해야 하는 경우
* **사용자 액션 처리**: 사용자의 모든 액션을 처리해야 하는 경우
* **네트워크 요청**: 모든 요청을 서버에 전송해야 하는 경우

```kotlin
// 중요 데이터 전송 - send() 사용
suspend fun sendCriticalData(data: CriticalData) {
    dataChannel.send(data) // 반드시 전송해야 함
}
```

### trySend()를 사용해야 하는 경우

* **UI 이벤트 전송**: UI 이벤트가 실패해도 앱 동작에 영향이 없는 경우
* **실시간 데이터**: 최신 데이터만 중요하고 이전 데이터는 무시해도 되는 경우
* **성능이 중요한 경우**: 블로킹을 피하고 즉시 응답해야 하는 경우
* **Backpressure 처리**: Channel이 가득 찰 때 적절한 처리 전략이 있는 경우

```kotlin
// UI 이벤트 전송 - trySend() 사용
fun sendUiEvent(event: UiEvent) {
    uiEventChannel.trySend(event) // 실패해도 괜찮음
}
```
