# State Hoisting

> State Hoisting은 원래 React에서 먼저 소개된 개념으로, 여러 컴포넌트가 같은 상태를 공유할 때 공통 부모 컴포넌트로 상태를 끌어올리는 패턴입니다.  
> Jetpack Compose도 이 개념을 활용하여, Composable 내부가 아닌 상위에서 상태를 관리하도록 권장합니다.  
> 이 글에서는 Compose에서 State Hoisting이 무엇인지, 왜 필요한지, 언제 사용하는 것이 좋은지를 예시와 함께 알아보겠습니다.  

## 1. State Hoisting이란?

State Hoisting은 간단히 말해 **Composable 내부에서 상태를 직접 관리하지 않고, 상위 컴포저블에서 관리하게 위임하는 패턴**입니다.

즉, 상태(state)와 그 상태를 변경하는 로직(updater)을 외부에서 전달받는 것이라고 볼 수 있습니다.

> "State는 끌어올리고 (hoist), Composable은 단순히 UI만 그려라"

### 예시로 보는 구조

```kotlin
// State Hoisting을 적용한 Composable
@Composable
fun Counter(
    count: Int,
    onCountChange: (Int) -> Unit
) {
    Button(onClick = { onCountChange(count + 1) }) {
        Text("Count: $count")
    }
}

// 상태를 관리하는 부모 Composable
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    Counter(count = count, onCountChange = { count = it })
}
```

## 2. 왜 State Hoisting을 써야 할까?

### 1) 재사용성과 테스트가 좋아집니다

내부에 상태를 가지고 있지 않으면, 똑같은 UI를 다양한 로직에 맞게 재활용할 수 있어요.
또한 테스트할 때도 UI만 단독으로 테스트하기 쉬워집니다.

### 2) 단방향 데이터 흐름이 유지돼요

상태는 한 방향으로만 흐르니까, 디버깅이나 유지보수가 훨씬 쉬워집니다.

### 3) ViewModel이나 부모 View와의 연결이 쉬워집니다

상태를 외부로 뺄 수 있으니까, ViewModel에 넣거나 다른 상태들과 함께 관리하기가 좋습니다.

## 3. 언제 State Hoisting을 쓰면 좋을까?

* 재사용 가능한 Custom Composable을 만들 때
* ViewModel에서 상태를 직접 관리할 때
* 여러 상태가 서로 영향을 주고받는 UI에서 상태를 한 곳으로 모으고 싶을 때

## 4. State Hoisting이 필요 없는 경우도 있어요

* 해당 Composable이 매우 작고 재사용 가능성이 거의 없을 때
* 상태를 외부에 노출하면 더 복잡해질 때
* 내부에서 자체적으로만 쓰는 상태일 때 (예: 애니메이션 관련 상태)

---

## 참고 자료

* [Android 공식 문서 - State and Jetpack Compose](https://developer.android.com/jetpack/compose/state)
* [Compose Samples - Crane](https://github.com/android/compose-samples)
