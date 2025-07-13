# LaunchedEffect란?

> Jetpack Compose를 쓰다 보면 네트워크 요청이나 Toast, Navigation 같은 부수효과(Side-effect)를 실행해야 할 때가 있습니다.  
> 컴포지션이 꼬이거나 앱이 이상하게 동작하지 않게  
> 안전하게 쓰라고 있는 게 바로 `LaunchedEffect`입니다.  
> 이 글에서는 LaunchedEffect가 무엇인지에 대해 학습하며 정리합니다.  

---

## 1. LaunchedEffect란?

`LaunchedEffect`는 컴포저블 내부에서 코루틴을 실행하게 해주는 API입니다.  
Composition에 안전하게 연결된 CoroutineScope를 제공해주기 때문에, UI 상태 변화에 맞춰 비동기 처리를 할 수 있습니다.  

```kotlin
@Composable
fun MyScreen() {
    LaunchedEffect(Unit) {
        delay(1000)
        println("1초 뒤 실행됨")
    }
}
```

* 위 예시는 화면이 그려질 때 한 번만 실행 됩니다.  
* key를 `Unit`으로 주면 최초 컴포지션에서만 실행되고 이후에는 재실행되지 않습니다.  

---

## 2. key의 의미

`LaunchedEffect(key)`에서 key는 “언제 다시 실행할 건지”를 결정하는 기준이다.

```kotlin
LaunchedEffect(viewModel.uiState) {
    if (uiState.isSuccess) {
        navController.navigate("Home")
    }
}
```

* key가 바뀔 때마다 블록 내부가 재실행 됩니다.  
* 주로 Toast, Navigation 같은 일회성 이벤트에 사용됩니다.  

---

## 3. 자주 하는 실수

### key를 무조건 Unit으로 고정

* 조건이 바뀌어도 실행되지 않음
* 화면 진입 시 한 번만 실행될 때만 Unit 사용해야 함

### 계속 바뀌는 값으로 key 설정

```kotlin
LaunchedEffect(System.currentTimeMillis()) {
    // 계속 실행됨
}
```

### 상태가 아닌 이벤트 처리할 때

* `StateFlow`는 상태 변경에 반응하므로, 이벤트처럼 쓰면 재발생 못 시킴
* `SharedFlow`나 `SingleEvent` 패턴 써야 함

---

## 4. SharedFlow + LaunchedEffect 예시

```kotlin
LaunchedEffect(Unit) {
    viewModel.eventFlow.collectLatest { event ->
        when (event) {
            is UiEvent.ShowToast -> {
                Toast.makeText(context, event.message, Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

* `collectAsState()`가 아닌 `collect`로 구독해야 일회성 이벤트가 정상 처리됨

---

### 참고 자료

* [공식 문서 - LaunchedEffect](https://developer.android.com/reference/kotlin/androidx/compose/runtime/LaunchedEffect)
* [Compose Side-effects 문서](https://developer.android.com/jetpack/compose/side-effects)
