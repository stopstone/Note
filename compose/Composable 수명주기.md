# Composable 수명주기

> Jetpack Compose는 초기 컴포지션 시 처음으로 컴포저블을 실행할 때,  
> 컴포지션에서 UI를 기술하기 위해 호출하는 컴포저블을 추적합니다.
> 
> 그런 다음 앱 상태가 변경되면 Jetpack Compose는 리컴포지션을 예약합니다.     
> 리컴포지션은 Jetpack Compose가 상태 변경사항에 따라 변경될 수 있는 컴포저블을 다시 실행한 다음 변경사항을 반영하도록 컴포지션을 업데이트하는 것입니다.  
> 컴포지션은 초기 컴포지션을 통해서만 생성되고 리컴포지션을 통해서만 업데이트될 수 있습니다. 컴포지션을 수정하는 유일한 방법은 리컴포지션을 통하는 것입니다.  
> 
> `리컴포지션`을 정확히 관리하지 못하면 불필요한 계산, UI 깜빡임, 리소스 낭비가 발생할 수 있습니다.  
> 따라서 수명주기를 명확히 이해하고 다루는 것은 생각보다 더 중요한 일일지도 모릅니다.
> 이 문서에서는 컴포저블의 수명 주기에 관해 알아보며 Compose에서 컴포저블에 재구성이 필요한지를 결정하는 방법을 살펴봅니다.

## 1. **컴포저블 수명주기란?**
![image](https://github.com/user-attachments/assets/a0fbacac-a5a1-4f73-ba76-4aff2aca865c)
- `컴포지션 시작` → `0회 이상 리컴포지션` → `컴포지션 종료`로 이어지는 흐름입니다.

**컴포지션(Composition)**
  - 컴포저블이 UI 트리에 "처음 추가"될 때 실행됩니다.
  - 컴포저블 함수들이 호출되면서 컴포지션 트리를 구성하고 UI를 선언합니다.

**리컴포지션(Recomposition)**
  - 앱 상태(State)가 변경될 때, 변경된 상태를 읽은 컴포저블과 관련된 부분만 "다시 실행"하여 UI를 업데이트합니다.
  - 전체 트리를 다시 그리지 않고, 필요한 부분만 똑똑하게 갱신합니다.

**디컴포지션(Decomposition)**
  - 컴포저블이 UI 트리에서 제거될 때 발생합니다.
  - 메모리 정리, 리소스 해제가 필요하며 DisposableEffect를 통해 정리 작업을 할 수 있습니다.

- 컴포지션은 초기 컴포지션을 통해서만 생성되고, 리컴포지션을 통해서만 업데이트될 수 있습니다.
- 수정을 원할 경우 반드시 리컴포지션을 통해야 하며, 컴포지션을 직접 수정할 방법은 없습니다.


## 2. 호출 사이트(Call Site) 개념

- **호출 사이트란?**
  - 컴포저블 함수가 코드에서 호출된 "정확한 위치"를 의미합니다.
  - Compose 컴파일러는 호출 사이트를 고유하게 인식합니다.

- **중요한 이유**
  - 컴포저블의 수명주기와 식별은 호출 사이트를 기준으로 결정됩니다.
  - 같은 컴포저블 함수라도 호출 위치가 다르면 별개의 인스턴스로 관리됩니다.

- **예시**
```kotlin
@Composable
fun LoginScreen(showError: Boolean) {
    if (showError) {
        LoginError() // 조건부 호출
    }
    LoginInput() // 항상 호출
}
```
![image](https://github.com/user-attachments/assets/fa021c04-68a2-4bef-af26-5eefb4a64cd5)

- showError 값에 따라 LoginError가 조건부로 호출되고, LoginInput은 항상 호출됩니다.
- 호출 사이트가 다르기 때문에 각각 별도의 인스턴스로 컴포지션에 등록됩니다.


## 3. 리컴포지션 최적화 (Smart Recomposition)

- Compose는 State 객체 변화를 추적하여 리컴포지션을 트리거합니다.
- **변경된 State를 읽은 컴포저블만 리컴포지션**됩니다.
- 나머지 부분은 **건너뜁니다(skip)**.
- 이는 퍼포먼스 최적화에 매우 중요합니다.

- **컴포저블 입력 매개변수**가 이전과 동일하면 리컴포지션을 건너뛸 수 있습니다.


## 4. key를 사용한 컴포저블 인스턴스 식별

- 리스트를 그릴 때(for문, LazyColumn 등) 컴포저블이 여러 번 호출됩니다.
- 같은 호출 사이트 + 순서만으로 식별하면 리스트 변경 시 문제가 발생할 수 있습니다.
- **key()**를 사용해 각 컴포저블 인스턴스를 고유하게 구분해야 합니다.

- **key() 사용 예시**
```kotlin
@Composable
fun MoviesScreenWithKey(movies: List<Movie>) {
    Column {
        for (movie in movies) {
            key(movie.id) { // 각 movie를 고유하게 식별
                MovieOverview(movie)
            }
        }
    }
}
```

- **LazyColumn에서 key 사용 예시**
```kotlin
LazyColumn {
    items(movies, key = { movie -> movie.id }) { movie ->
        MovieOverview(movie)
    }
}
```

- **효과**
  - 리스트 아이템이 재정렬되거나 추가/삭제될 때 기존 인스턴스를 유지할 수 있습니다.
  - 부수 효과나 내부 상태를 잃지 않습니다.


## 5. 타입 안정성(Stable)과 리컴포지션

**Stable 타입이란?**
  - equals()로 비교했을 때 항상 결과가 일관됩니다.
  - 공개 속성이 변경되면 Compose에 알릴 수 있어야 합니다.
  - 모든 공개 속성도 안정적인 타입이어야 합니다.

**Compose가 기본적으로 Stable로 보는 타입**
  - Boolean, Int, Long, Float, Char
  - String
  - 모든 함수 타입(람다)
  - MutableState (값 변경 시 자동 알림)

**불안정하게 보는 경우**
  - 인터페이스 (기본적으로 안정적이지 않다고 가정)
  - 변경 가능한 공개 속성을 가진 클래스

**@Stable 어노테이션 사용**
  - 컴파일러에게 "이 타입은 안정적입니다"라고 명시할 수 있습니다.

**@Stable 예시**
```kotlin
@Stable
interface UiState<T : Result<T>> {
    val value: T?
    val exception: Throwable?

    val hasError: Boolean
        get() = exception != null
}
```

- 이처럼 인터페이스라도 @Stable을 붙이면 Compose가 안정적으로 판단하고 스마트 리컴포지션을 적용할 수 있습니다.


# 참고 자료

- [Jetpack Compose 공식 문서 - Compose 수명주기](https://developer.android.com/jetpack/compose/lifecycle#composition-lifecycle)
