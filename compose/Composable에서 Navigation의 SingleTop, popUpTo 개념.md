# Composable에서 Navigation의 SingleTop, popUpTo 개념

> Jetpack Compose에서 Navigation을 구성하다 보면 `launchSingleTop`, `popUpTo`, `restoreState` 같은 속성이 있습니다.  
> 이들은 화면 전환의 동작 방식을 세밀하게 제어하기 위한 기능입니다.  
> 이번 글에서는 각각의 개념을 실제 예제와 함께 간단명료하게 정리해보겠습니다.  

---

## 1. launchSingleTop이란?

### ● 정의

`launchSingleTop = true`는 **이미 백 스택에 동일한 경로(route)** 가 있다면,  
새로운 인스턴스를 생성하지 않고 **기존 인스턴스를 재사용**합니다.  

### ● 언제 유용한가요?

* 홈 화면에서 다시 홈으로 이동할 때
* 탭 형태의 UI에서 중복된 화면을 계속 쌓지 않게 하고 싶을 때

### ● 예시 코드

```kotlin
navController.navigate("home") {
    launchSingleTop = true
}
```

위 코드는 현재 백 스택에 "home"이 있으면 **새로운 home 인스턴스를 생성하지 않습니다.**

---

## 2. popUpTo란?

### ● 정의

`popUpTo("route")`는 **지정한 route까지 백 스택을 제거(pop)** 합니다.

### ● 옵션: `inclusive = true/false`

* `true`: 지정한 route도 제거함
* `false`: 지정한 route는 남겨둠

### ● 언제 유용한가요?

* 로그인 후 홈 화면으로 이동하면서 로그인 화면을 스택에서 제거할 때
* 특정 플로우를 끝내고 루트로 돌아갈 때

### ● 예시 코드

```kotlin
navController.navigate("home") {
    popUpTo("login") {
        inclusive = true
    }
}
```

→ 위 코드는 login 화면까지 백 스택을 비우고, home으로 이동합니다.

---

## 3. restoreState이란?

### ● 정의

`restoreState = true`는 **popUpTo로 인해 제거된 화면을 다시 만들지 않고, 기존 상태를 복원**합니다.

### ● 사용 조건

* `popUpTo`에 지정한 route가 `saveState = true`로 저장되어 있어야 함

### ● 예시 코드

```kotlin
navController.navigate("home") {
    popUpTo("home") {
        saveState = true
    }
    launchSingleTop = true
    restoreState = true
}
```

→ 이 코드는 home을 백 스택에서 제거한 후, 다시 이동하면서 이전 상태를 복원합니다.

---

## 4. 실전 예시: 탭 UI에서의 활용

```kotlin
BottomNavigationItem(
    selected = currentDestination == "home",
    onClick = {
        navController.navigate("home") {
            popUpTo(navController.graph.startDestinationId) {
                saveState = true
            }
            launchSingleTop = true
            restoreState = true
        }
    }
)
```

→ 탭을 눌러도 스택이 쌓이지 않고, 이전 화면 상태를 그대로 복원합니다.

---

## 참고자료

* [공식 Navigation 문서](https://developer.android.com/jetpack/compose/navigation)
* [Now in Android 예제](https://github.com/android/nowinandroid)
* [Toss Android GitHub](https://github.com/toss/toss-android)
