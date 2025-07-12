# Jetpack Compose 리컴포지션으로 인한 LaunchedEffect 중복 실행 문제 해결

## 문제 상황

그룹 생성 화면에서 "그룹 만들기" 버튼을 클릭했을 때, 키보드가 예상치 못한 시점에 다시 올라오는 문제가 발생했습니다.

### 예상된 플로우

1. 화면 진입 → 키보드 올라옴
2. 버튼 클릭 → 키보드 내려감 → 로딩 화면
3. 그룹 생성 완료 → 확인 화면으로 전환

### 실제 발생한 플로우

1. 화면 진입 → 키보드 올라옴
2. 버튼 클릭 → 키보드 내려감 → 로딩 화면
3. 그룹 생성 완료 → **키보드가 다시 올라옴** → 확인 화면으로 전환

## 문제 원인 분석

### 1차 분석: UI 상태 변화 추적

```kotlin
// 문제가 되었던 코드 구조
@Composable
fun GroupCreateScreen() {
    if (uiState.isLoading) {
        ZubZubLoadingProgress()
    } else {
        GroupCreateContent() // 여기서 문제 발생
    }
}
```

UI 상태 변화:

* `isLoading = true` → GroupCreateContent 사라짐
* `isLoading = false` → GroupCreateContent 다시 나타남

### 2차 분석: 리컴포지션 확인

GroupCreateContent 내부에 디버깅 로그를 추가하여 리컴포지션 발생 여부를 확인했습니다.

```kotlin
@Composable
private fun GroupCreateContent() {
    LaunchedEffect(Unit) {
        println("GroupCreateContent 리컴포지션 발생!")
        println("키보드 올리기 LaunchedEffect 실행!")
        focusRequester.requestFocus()
        keyboardController?.show()
    }
}
```

### 테스트 결과

```text
// 첫 번째 실행 (화면 진입)
GroupCreateContent 리컴포지션 발생!
키보드 올리기 LaunchedEffect 실행!

// 두 번째 실행 (로딩 완료 후)
GroupCreateContent 리컴포지션 발생!
키보드 올리기 LaunchedEffect 실행!
```

**결론: GroupCreateContent가 두 번 생성되면서 LaunchedEffect(Unit)이 중복 실행됨**

## 문제의 핵심

### Composable 생명주기와 LaunchedEffect

```kotlin
LaunchedEffect(Unit) {
    // Unit을 key로 사용하면 Composable이 새로 생성될 때마다 실행됨
    keyboardController?.show()
}
```

GroupCreateContent가 완전히 새로 생성되면서 LaunchedEffect가 재실행되는 현상이 문제의 원인이었습니다.

### UI 상태 변화 시퀀스

1. `isLoading = false` → GroupCreateContent 렌더링 → LaunchedEffect 실행
2. `isLoading = true` → GroupCreateContent 제거 → LaunchedEffect 정리
3. `isLoading = false` → GroupCreateContent 재렌더링 → **LaunchedEffect 재실행**

## 해결 방안 시도

### 시도 1: LaunchedEffect key 변경

```kotlin
// 실패: 여전히 재실행됨
LaunchedEffect("keyboard_init") {
    keyboardController?.show()
}
```

### 시도 2: DisposableEffect 사용

```kotlin
// 실패: DisposableEffect도 재실행됨
DisposableEffect(Unit) {
    keyboardController?.show()
    onDispose { }
}
```

### 시도 3: 복잡한 상태 관리 (isSuccess 추가)

```kotlin
// 실패: 복잡성만 증가, 근본적 해결 안됨
data class GroupCreateUiState(
    val isLoading: Boolean = false,
    val isSuccess: Boolean = false,
    // ...
)
```

## 최종 해결 방법

### 해결책: Composable 스코프 변경

키보드 제어 로직을 리컴포지션의 영향을 받지 않는 상위 컴포넌트로 이동했습니다.

```kotlin
@Composable
fun GroupCreateScreen() {
    val focusRequester = remember { FocusRequester() }
    val keyboardController = LocalSoftwareKeyboardController.current

    // 상위 스코프에서 한 번만 실행
    LaunchedEffect(Unit) {
        focusRequester.requestFocus()
        keyboardController?.show()
    }

    if (uiState.isLoading) {
        ZubZubLoadingProgress()
    } else {
        GroupCreateContent(
            focusRequester = focusRequester // 전달만 함
        )
    }
}

@Composable
private fun GroupCreateContent(
    focusRequester: FocusRequester
) {
    // 키보드 제어 로직 제거
    ZubZubInputTextField(
        modifier = Modifier.focusRequester(focusRequester)
    )
}
```

### 해결 후 테스트 결과

```text
// 단 한 번만 실행됨
GroupCreateScreen 키보드 초기화!
```

## 핵심 원리

### 1. Composable 생명주기 이해

* **GroupCreateScreen**: 네비게이션 단위로 한 번만 생성
* **GroupCreateContent**: UI 상태에 따라 생성/제거 반복

### 2. LaunchedEffect 실행 조건

```kotlin
LaunchedEffect(key) {
    // key가 변경되거나 Composable이 새로 생성될 때 실행
}
```

### 3. 상태 관리 레벨 분리

* **영구적 상태**: 상위 컴포넌트에서 관리
* **일시적 UI**: 하위 컴포넌트에서 관리

## 학습 포인트

1. **LaunchedEffect는 Composable 생명주기와 밀접한 관련이 있음**
2. **UI 상태 변화로 인한 Composable 재생성을 고려해야 함**
3. **부수 효과(Side Effect)는 적절한 스코프에서 관리해야 함**
4. **복잡한 상태 관리보다는 구조적 해결이 더 효과적일 수 있음**

## 예방 방법

1. **LaunchedEffect 사용 시 Composable 생명주기 고려**
2. **일회성 초기화는 상위 안정적인 스코프에서 처리**
3. **디버깅 로그를 활용한 리컴포지션 추적**
4. **UI 상태 변화에 따른 컴포넌트 생성/제거 패턴 파악**
