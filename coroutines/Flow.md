# Kotlin Flow

> 과거에는 LiveData가 비동기 상태 관리를 위한 기본 선택이었습니다.  
> 그러나 복잡한 비동기 흐름과 다양한 요구사항을 만족시키기 위해 Flow가 대세로 자리 잡고 있습니다.  
> 최근 안드로이드 개발에서는 "ViewModel은 StateFlow, UI는 collect로 수집"하는 구조를 많이 채택하고 있습니다.  
> 그래서 이번 문서에서는 Flow가 무엇인지, 어떻게 활용할 수 있는지에 대해 정리합니다.

## 1. 비동기 처리는 왜 필요할까요?
안드로이드 애플리케이션에서는 네트워크 요청이나 데이터베이스 접근처럼 시간이 오래 걸리는 작업을 자주 처리해야 합니다.  
이러한 작업을 메인(UI) 스레드에서 실행하면 앱이 멈추거나 "Application Not Responding(ANR)" 오류가 발생할 수 있습니다.  
따라서 비동기 처리는 필수적입니다.

Kotlin에서는 `Coroutines`과 `Flow`를 이용하여 비동기 작업을 쉽게 처리할 수 있습니다.  
기존에는 안드로이드에서 비동기 데이터를 관리하기 위해 **LiveData**를 주로 사용했지만,  
복잡한 데이터 흐름이나 다양한 비동기 처리 요구사항을 다루기에는 한계가 있었습니다.

---

## 2. Flow란 무엇인가요요?
**Flow**는 Kotlin에서 제공하는 코루틴 기반 비동기 데이터 스트림 API입니다.
![flow-entities](https://github.com/user-attachments/assets/55216ede-eb1d-4fa2-8aa0-6b018a64fcce)

- 하나 이상의 데이터를 순차적으로 비동기 발행할 수 있습니다.
- 기본적으로 **Cold Stream**입니다. (구독이 시작되어야 데이터가 생성됩니다)
  - `flow { emit(...) }` 형태로 만들 경우, collect가 호출될 때마다 새로 실행됩니다.
- Room, Retrofit 등 다양한 Jetpack 라이브러리들과 자연스럽게 통합할 수 있습니다.

### Cold Stream vs Hot Stream

| 구분 | Cold Stream | Hot Stream |
|:--|:--|:--|
| 데이터 생성 시점 | 구독(collect) 시작 시 생성 | 구독 여부와 무관하게 생성 |
| 예시 | Flow 기본 구현 | StateFlow, SharedFlow |

### Flow를 수집(collect)하는 이유

Flow는 선언만으로는 동작하지 않고, 반드시 `collect`를 호출해야 실제로 데이터가 생성되고 발행됩니다.  
이를 "수집(collecting)"한다고 표현합니다.

```kotlin
flowOf(1, 2, 3) // 1, 2, 3을 발행하는 Flow 생성
    .collect { value ->
        println(value) // 값을 출력
    }
```

**정리**
- Flow는 Cold Stream이므로 `collect`가 호출되어야 데이터가 흐르기 시작합니다.
- `collect`는 suspend 함수이므로 코루틴 범위 안에서 호출해야 합니다.

### Flow의 장점
- 코루틴과 완벽하게 통합됩니다.  
- 안드로이드 프레임워크에 의존하지 않아, 순수 Kotlin 환경에서도 사용할 수 있습니다.  

### Flow의 단점
- 기본적으로 생명주기 인식이 없습니다. 따라서 `repeatOnLifecycle`이나 `flowWithLifecycle`과 같은 생명주기 안전 수집 코드가 필요합니다.  
- Backpressure를 직접 관리해야 할 경우 복잡해질 수 있습니다.  
(Backpressure란 생산 속도가 소비 속도를 초과할 때 발생하는 문제를 의미합니다)  
- 예외 처리를 위해 try-catch 대신 `catch`, `retry`, `onEach` 등의 연산자를 사용해야 하므로 초보자에게 다소 진입장벽이 있을 수 있습니다.  

#### Flow 수집 중 예외 처리 예시

```kotlin
flow {
    emit(loadFromNetwork()) // 네트워크에서 데이터 로드
}.catch { e ->
    emit(handleError()) // 에러 발생 시 처리
}
```

### Flow 활용 방법

#### flowOn을 통한 스레드 변경

```kotlin
flow {
    emit(loadFromNetwork()) // 네트워크 작업
}.flowOn(Dispatchers.IO) // IO 스레드에서 실행
```

#### collectLatest로 최신 데이터만 수집하기

```kotlin
flowOf(1, 2, 3)
    .collectLatest { value ->
        delay(100)
        println(value) // 가장 마지막 값만 출력
    }
```

#### flatMapLatest로 최신 요청 유지하기

```kotlin
flowOf(query)
    .flatMapLatest { searchApi.search(it) }
    .collect { result ->
        // 결과 처리
    }
```

#### Flow의 취소(Cancellation)

Flow는 코루틴의 취소 메커니즘을 따릅니다.  
`collect`가 수행되는 코루틴이 취소되면 Flow 수집도 즉시 중단됩니다.  
별도의 추가 코드 없이 자연스럽게 취소가 지원됩니다.  

#### SharedFlow의 replay 활용하기

- `SharedFlow`는 기본적으로 상태를 가지지 않는 Hot Stream입니다.  
- `replay`를 설정하면 과거 N개의 값을 버퍼에 저장하여, 새로운 구독자에게 과거 데이터를 재전달할 수 있습니다.  
- 이를 통해 이벤트 손실 없이 과거 이벤트를 수신할 수 있습니다.  

---

## 3. LiveData와 비교

### LiveData의 역할
LiveData는 안드로이드 생명주기를 인식하는 데이터 홀더 클래스입니다.   
LiveData는 Activity나 Fragment의 생명주기를 자동으로 감지하여, 소멸 시 옵저버를 제거해 메모리 누수를 방지합니다.  
따라서 안드로이드의 종속성을 가집니다.  

> **LifecycleOwner**란 Activity, Fragment처럼 생명주기를 가지는 컴포넌트를 의미합니다.  

단순한 상태 전달에는 적합했지만, 복잡한 스트림 처리(여러 이벤트를 조합하거나, 변환하는 등)에는 한계가 있었습니다.  

### Flow의 장점
Flow는 다양한 연산자를 제공하여 복잡한 데이터 흐름을 간결하게 표현할 수 있습니다.  
또한 Android 플랫폼에 종속되지 않기 때문에 Domain Layer나 Data Layer에서도 자유롭게 사용할 수 있으며, 테스트 코드 작성도 훨씬 수월합니다.  

---

## 4. LiveData vs Flow 정리

| 항목 | LiveData | Flow |
|:--|:--|:--|
| 라이프사이클 인식 | 자동 인식 | 기본 인식 안 함 (repeatOnLifecycle 필요) |
| 플랫폼 종속성 | Android에 종속 | Kotlin만으로 사용 가능 |
| 데이터 스트림 | 기본 1회성 (상태 보관) | 연속적인 데이터 스트림 가능 |
| 변환(map, filter 등) | MediatorLiveData로 간접 지원 | 다양한 연산자 제공 |
| 테스트 용이성 | 약간 복잡 | 비교적 쉽게 단위 테스트 가능 |
| 주요 사용처 | 주로 UI 레이어 | Domain Layer, Data Layer 전반 |
| 널 허용 여부 | 기본적으로 null 허용 | 기본적으로 초기값 필요, null 지양 |

---

## 5. 왜 LiveData 대신 Flow/StateFlow를 사용할까요요?

클린 아키텍처를 지향하는 경우, **Domain Layer**와 **Data Layer**는 Android에 의존하지 않는 것이 이상적입니다.  
따라서 ViewModel에서는 Flow를 StateFlow로 변환하여 관리하고,  
UI 레이어에서는 StateFlow를 안전하게 수집하거나 필요에 따라 `asLiveData()`로 변환하여 사용할 수 있습니다.  

```kotlin
val uiState: StateFlow<UiState> = _uiState

lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { uiState ->
            // UI 업데이트
        }
    }
}
```

필요에 따라 LiveData로 변환하여 사용할 수도 있습니다.  

```kotlin
val liveDataState: LiveData<UiState> = viewModel.uiState.asLiveData()
```

### 여기서 stateIn은?

Flow는 cold stream이기 때문에 구독이 있을 때만 데이터가 흐릅니다.  
반면, UI에서는 상태를 항상 즉시 접근할 수 있어야 합니다. 이때 사용하는 것이 `stateIn`입니다.

`stateIn`은 Flow를 Hot Stream처럼 만들어줍니다.   
ViewModel 안에서는 `MutableStateFlow`를 사용하거나, 또는 일반 Flow를 `stateIn`을 통해 변환하여 항상 마지막 상태를 유지하는 방식으로 사용합니다.  

```kotlin
val uiState: StateFlow<UiState> = repository.getUiStateFlow()
    .stateIn(
        viewModelScope,
        SharingStarted.WhileSubscribed(5000),
        initialValue = UiState(),
    )
```

- `viewModelScope`: CoroutineScope를 지정합니다.  
- `SharingStarted.WhileSubscribed(5000)`: 구독자가 없으면 5초 후 스트림을 중단합니다.  
- `initialValue`: 초기 값을 지정합니다. (UI는 항상 즉시 상태를 읽을 수 있어야 하기 때문입니다)  

### SharingStarted 종류
- `WhileSubscribed(timeoutMillis)`: 구독자가 없으면 일정 시간 후 스트림 중단.  
- `Eagerly`: 즉시 스트림을 시작합니다.  
- `Lazily`: 첫 구독자가 생길 때 스트림을 시작합니다.  

### StateFlow vs SharedFlow 간단 비교

| 항목 | StateFlow | SharedFlow |
|:--|:--|:--|
| 상태 보유 여부 | 최신 값 1개 보관 | 상태를 보관하지 않음 |
| 주 용도 | UI 상태 관리 | 단발성 이벤트 처리 |
| 초기 값 필요 여부 | 필요함 | 필요 없음 |
| 구독자 처리 | 최신 상태를 즉시 제공 | 설정에 따라 리플레이 가능 |

---

## 6. 참고 자료

- [공식 문서 - Kotlin Flow](https://developer.android.com/kotlin/flow)
- [올리브영 기술 블로그 - Android에서 StateFlow 사용](https://oliveyoung.tech/2022-12-14/Android-State-Flow/)
- [홍범기 블로그 - Android UI에서 Flow 수집하는 안전한 길](https://hongbeomi.medium.com/%EB%B2%88%EC%97%AD-android-ui%EC%97%90%EC%84%9C-flow%EB%A5%BC-%EC%88%98%EC%A7%91%ED%95%98%EB%8A%94-%EC%95%88%EC%A0%84%ED%95%9C-%EA%B8%B8-bd8449e67ec3)
- [F-Lab 블로그 - 코루틴과 Flow 이해하기](https://f-lab.kr/insight/understanding-coroutines-and-flow)
- [F-Lab 블로그 - 안드로이드에서 Coroutine과 Flow 이해하기](https://f-lab.kr/insight/understanding-coroutine-and-flow-in-android-20240620)
- [홍범기 블로그 - Kotlin Coroutine Flow 소개](https://medium.com/hongbeomi-dev/kotlin-coroutine-flow-ac07cfdca42d)
- [현대자동차그룹 기술 블로그 - Kotlin Flow를 이용한 비동기 처리](https://developers.hyundaimotorgroup.com/blog/609)

