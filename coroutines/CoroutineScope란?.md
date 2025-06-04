# CoroutineScope란?

> 코루틴을 사용할 때 가장 기본이자 핵심이 되는 개념이 바로 CoroutineScope입니다.  
> 코루틴의 생명주기와 실행 환경을 정의하는 이 개념을 정확히 이해하면, 불필요한 메모리 누수나 비효율적인 작업을 방지할 수 있어요.  

---

## 1. CoroutineScope란?

CoroutineScope는 **코루틴이 어떤 컨텍스트(Context) 안에서 실행되는지를 정의**하고, 그 **생명주기를 함께 관리**해주는 역할을 합니다.

코루틴을 시작할 때는 반드시 어떤 스코프 안에서 실행해야 하며, 스코프가 종료되면 그 안에서 실행 중이던 코루틴도 함께 취소됩니다.

---

## 2. CoroutineScope의 구성

CoroutineScope는 **CoroutineContext**를 포함하고 있어요. 이 Context에는 다음과 같은 요소들이 포함됩니다:

| 요소           | 설명                                                              |
| ------------ | --------------------------------------------------------------- |
| `Dispatcher` | 코루틴이 어떤 스레드에서 실행될지 결정 (`Dispatchers.Main`, `Dispatchers.IO`, 등) |
| `Job`        | 코루틴의 생명주기를 관리, 취소하거나 완료 상태를 추적할 수 있음                            |

예시:

```kotlin
val scope = CoroutineScope(Dispatchers.Main + Job())
```

---

## 3. CoroutineScope와 Android에서 제공하는 Scope들

### CoroutineScope (기본)

`CoroutineScope`는 **코루틴을 실행하기 위한 기본 단위**예요. 생명주기를 수동으로 관리해야 하며, 직접 `Job`을 붙여서 취소할 수 있어요.

```kotlin
val scope = CoroutineScope(Dispatchers.IO + Job())

fun someWork() {
    scope.launch {
        // 비동기 작업
    }
}

fun clear() {
    scope.cancel() // 수동으로 스코프 종료
}
```

* 보통 커스텀 유틸 클래스, Repository 내부에서 사용되며, ViewModel이나 LifecycleOwner와 연결되어 있지 않기 때문에 **명시적인 취소가 필수**예요.
* `Job` 없이 만들면 코루틴을 취소할 수 없고 메모리 누수가 발생할 수 있어요.

---

### viewModelScope

* ViewModel이 소멸될 때 자동으로 코루틴이 취소됩니다.
* UI 데이터를 로드하거나 비즈니스 로직을 수행할 때 가장 자주 사용합니다.

```kotlin
class MyViewModel : ViewModel() {
    fun load() {
        viewModelScope.launch {
            val data = repo.fetch()
            _state.value = data
        }
    }
}
```

---

### lifecycleScope

* LifecycleOwner(Activity, Fragment)의 생명주기를 따릅니다.
* UI 갱신용 비동기 작업 처리에 적합합니다.

```kotlin
lifecycleScope.launch {
    val result = repository.getData()
    // UI 처리
}
```

> Fragment에서는 **viewLifecycleOwner.lifecycleScope**를 쓰는 게 안전해요.

---

### rememberCoroutineScope

* Jetpack Compose에서 사용하는 CoroutineScope예요.
* 해당 Composable이 Composition에서 제거될 때 함께 취소됩니다.

```kotlin
@Composable
fun SaveButton() {
    val coroutineScope = rememberCoroutineScope()
    Button(onClick = {
        coroutineScope.launch {
            // 저장 처리
        }
    }) {
        Text("Save")
    }
}
```

---

### GlobalScope

- 앱 전체 생명주기를 따르며, **절대 자동으로 취소되지 않아요**.
- 메모리 누수, 예외 처리 미흡 등 **위험 요소가 많기 때문에 가급적 사용하지 않습니다.**

```kotlin
val job = GlobalScope.launch {
    // 장시간 작업
}

// 이후 필요 시 명시적으로 취소해야 함
job.cancel()

