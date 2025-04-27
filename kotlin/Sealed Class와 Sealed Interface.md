# Kotlin Sealed Class & Sealed Interface 정리

> Kotlin에서는 복잡한 상태나 다양한 타입의 분기를 깔끔하게 다루기 위해 `sealed` 문법을 제공합니다.  
> `sealed class`와 `sealed interface`는 컴파일 타임에 타입을 명확하게 제한하여 안전하고 가독성 높은 코드를 작성할 수 있도록 도와줍니다.   
> 주로 상태 관리, API 응답 처리, 다양한 타입별 로직 분기에 활용됩니다.  

## 1. Sealed Class란?

Sealed class는 상속 가능한 하위 클래스의 종류를 컴파일 타임에 모두 알 수 있도록 제한하는 추상 클래스입니다.  
덕분에 `when` 표현식에서 모든 하위 클래스를 모든 경우를 빠짐없이 처리할 수 있으며,  
컴파일러가 모든 경우를 다뤘는지 검사해주어 타입 안전성을 높일 수 있습니다.

### Sealed Class 주요 특징
- 특정 계층 구조 내에서 하위 클래스를 명확하게 제한합니다.
- `when` 문 사용 시 `else` 없이 모든 하위 타입을 처리할 수 있습니다.
- 하위 클래스는 반드시 **같은 파일 내에 정의**해야 합니다.
- 생성자(protected 또는 private)를 가질 수 있어 상태를 보관할 수 있습니다.


## 2. Sealed Interface란?

Sealed interface는 Kotlin 1.5부터 지원되는 기능으로, sealed class와 개념은 비슷하지만 몇 가지 중요한 차이점이 있습니다. 
Sealed interface는 다중 구현을 허용하며, 상태를 보관할 수 없습니다. 주로 다양한 타입 간의 공통된 행위만 정의하고, 데이터나 상태를 직접 포함하지 않을 때 사용합니다.

### Sealed Interface 주요 특징
- 여러 sealed interface를 동시에 구현할 수 있습니다.
- 생성자를 가질 수 없으며, 상태(프로퍼티)를 직접 보관할 수 없습니다.
- 구현체는 **같은 모듈** 내에서만 정의할 수 있습니다. 이는 모듈 단위로 타입 안정성을 보장하기 위함입니다.
- 주로 '행위'를 제한할 때 사용합니다.


## 3. Sealed Class vs Sealed Interface

| 항목 | Sealed Class | Sealed Interface |
|------|--------------|------------------|
| **상속** | 단일 상속만 가능 | 다중 구현 가능 |
| **생성자** | 가질 수 있음 | 가질 수 없음 |
| **상태 보관** | 가능 | 불가 |
| **상속 제한 범위** | 같은 파일 | 같은 모듈 |
| **주요 용도** | 데이터, 상태 표현 | 행위 정의 |


## 4. 사용 예제

### 4-1. Sealed Class 사용 예제

```kotlin
sealed class ApiResponse {
    data class Success(val data: String) : ApiResponse()
    data class Error(val error: String) : ApiResponse()
    object Loading : ApiResponse()
}

fun handleResponse(response: ApiResponse) {
    when (response) {
        is ApiResponse.Success -> println("Success with data: ${response.data}")
        is ApiResponse.Error -> println("Error: ${response.error}")
        is ApiResponse.Loading -> println("Loading...")
    }
}
```

### 4-2. Sealed Interface 사용 예제

```kotlin
sealed interface UiState

object Loading : UiState

data class Success(val data: String) : UiState

data class Error(val message: String) : UiState

fun render(state: UiState) {
    when (state) {
        is Loading -> showLoading()
        is Success -> showData(state.data)
        is Error -> showError(state.message)
    }
}
```


## 5. 언제 사용할까요?

- **Sealed Class**: 상태(state)나 데이터를 가지는 구조를 정의할 때.
- **Sealed Interface**: 다양한 타입이 공통된 기능(행위)을 가질 때. 또는 여러 계층 구조가 필요한 경우.


## 참고 자료

- [Kotlin 공식 문서 - Sealed Classes and Interfaces](https://kotlinlang.org/docs/sealed-classes.html)

