# collectAsState vs collectAsStateWithLifecycle

> Jetpack Compose에서 Flow를 상태로 수집할 때 사용하는 두 가지 방식인 `collectAsState()`와 `collectAsStateWithLifecycle()`은 겉보기엔 비슷하지만 중요한 차이를 가지고 있습니다.  
> 이 글에서는 각각의 동작 방식, 차이점, 언제 어떤 것을 써야 하는지에 대해 정리합니다.  

---

## 1. collectAsState()

`collectAsState()`는 Compose의 `State<T>`로 변환하여 recomposition을 유도하기 위해 사용됩니다.  
하지만 Android의 Lifecycle을 인식하지 못하기 때문에, Composable이 화면에서 보이지 않더라도 Flow를 계속 수집할 수 있다는 단점이 있습니다.

```kotlin
val uiState by viewModel.stateFlow.collectAsState()
```

* Composition에 들어갈 때 Flow 수집 시작
* Lifecycle 상태와 무관하게 항상 수집
* 리소스 낭비 가능성 존재

---

## 2. collectAsStateWithLifecycle()

`collectAsStateWithLifecycle()`은 Android Lifecycle-aware 수집을 지원합니다.  
내부적으로 `repeatOnLifecycle(Lifecycle.State.STARTED)`를 사용하여, `STARTED` 상태 이상일 때만 Flow를 수집하고 그렇지 않으면 자동으로 중단됩니다.

```kotlin
val uiState by viewModel.stateFlow.collectAsStateWithLifecycle()
```

* Lifecycle이 `STARTED` 이상일 때만 수집
* 앱이 백그라운드로 전환되면 수집 자동 중단
* 리소스 낭비 방지, 안전한 상태 관리 가능

---

## 3. 차이 비교

| 항목           | collectAsState() | collectAsStateWithLifecycle() |
| ------------ | ---------------- | ----------------------------- |
| Lifecycle 인식 | 없음               | 있음                            |
| 수집 시점        | Composition 진입 시 | Lifecycle STARTED 상태          |
| 백그라운드 동작     | 계속 수집함           | 자동 중단됨                        |
| 리소스 효율성      | 낮음               | 높음                            |
| 사용 용도        | 간단한 UI           | Lifecycle-aware가 필요한 UI       |

---

## 4. 언제 어떤 걸 써야 하나?

### collectAsState()

* 항상 화면에 보여지는 단순한 Composable
* Lifecycle 영향을 받지 않는 데이터 수집

### collectAsStateWithLifecycle()

* 화면 이동, 백그라운드 전환 등 Lifecycle 변화가 있는 경우
* 네트워크나 리소스가 큰 Flow 수집이 필요한 경우

Lifecycle을 인식하지 못하고 Flow를 계속 수집하면 메모리 누수(leak) 또는 불필요한 로직 반복으로 이어질 수 있으므로,  
`collectAsStateWithLifecycle()` 사용이 기본 전략이 되어야 합니다.

---

## 참고 자료

* [Consuming flows safely in Jetpack Compose](https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3)
* [collectAsStateWithLifecycle - Android Docs](https://developer.android.com/reference/kotlin/androidx/lifecycle/compose/package-summary)
* [repeatOnLifecycle 이해하기](https://developer.android.com/topic/libraries/architecture/lifecycle#repeatonlifecycle)
