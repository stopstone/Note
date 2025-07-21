# State와 Effect

> Jetpack Compose에서 **State**와 **Effect**는 둘 다 UI 상태 관리에 필요한 개념입니다.
> 각각의 역할과 사용 시점이 다른데 헷갈릴 수 있고, 판단이 안 설 수 있습니다.    
> State는 UI가 표시할 데이터를 담고, Effect는 State 변경에 따른 부가 효과(Side Effect)를 처리할 때 사용하는 개념입니다.  
> 특히 MVI 패턴에서는 State가 UI State를, Effect가 One-time Effect를 담당하여 단방향 데이터 흐름을 구현합니다.   
> 이 글에서는 두 개념의 차이점과 언제 어떤 것을 사용해야 하는지 알아보겠습니다.  

## 1. State란?

State는 **UI가 표시할 데이터를 담는 컨테이너**입니다.  
사용자 인터페이스가 어떤 정보를 보여줄지, 어떤 상태에 있는지를 나타내는 값들이라고 볼 수 있습니다

### State의 특징

* **UI 렌더링의 근거**: State가 변경되면 UI가 다시 그려집니다
* **데이터 홀더**: 실제 데이터를 저장하고 관리합니다
* **반응형**: State 변경 시 자동으로 UI가 업데이트됩니다

```kotlin
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) } // State
    
    Column {
        Text("Count: $count") // State를 UI에 표시
        Button(onClick = { count++ }) { // State 변경
            Text("증가")
        }
    }
}
```

### 여러 State를 Data Class로 관리하는 경우

실제 프로젝트에서는 여러 State를 하나의 Data Class로 관리하는 것이 일반적입니다.

```kotlin
// State를 담는 Data Class
data class LoginState(
    val email: String = "",
    val password: String = "",
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)

@Composable
fun LoginScreen() {
    var loginState by remember { mutableStateOf(LoginState()) }
    
    Column {
        TextField(
            value = loginState.email,
            onValueChange = { loginState = loginState.copy(email = it) }
        )
        TextField(
            value = loginState.password,
            onValueChange = { loginState = loginState.copy(password = it) }
        )
        
        if (loginState.isLoading) {
            CircularProgressIndicator()
        }
        
        loginState.errorMessage?.let { error ->
            Text(error, color = Color.Red)
        }
    }
}
```

**장점:**
* **관련 State들을 그룹화**: 논리적으로 연결된 State들을 하나로 관리
* **불변성 보장**: copy()를 통한 안전한 State 업데이트
* **디버깅 용이성**: 전체 State를 한 번에 확인 가능

## 2. Effect란?

Effect는 **State 변경에 따른 부수 효과를 처리하는 메커니즘**입니다.  
State가 바뀌었을 때 UI 렌더링 외에 추가로 해야 할 작업들을 담당해요.

### Effect의 종류

#### 1) LaunchedEffect
```kotlin
@Composable
fun UserProfileScreen(userId: String) {
    var user by remember { mutableStateOf<User?>(null) }
    
    LaunchedEffect(userId) { // Effect: userId가 변경될 때마다 API 호출
        user = userRepository.getUser(userId)
    }
    
    // UI 렌더링
    user?.let { user ->
        Text("이름: ${user.name}")
    }
}
```

**핵심**: LaunchedEffect는 첫 번째 매개변수(키)가 변경될 때만 실행됩니다.
```

### 여러 Effect를 Sealed Class로 관리하는 경우

실제 프로젝트에서는 여러 Effect를 Sealed Class로 관리하는 것이 일반적입니다.

```kotlin
// Effect를 담는 Sealed Class
sealed class LoginEffect {
    object NavigateToHome : LoginEffect()
    object ShowError : LoginEffect()
    data class ShowToast(val message: String) : LoginEffect()
}

@Composable
fun LoginScreen() {
    var loginState by remember { mutableStateOf(LoginState()) }
    
    // Effect 처리
    LaunchedEffect(loginState) {
        when {
            loginState.isLoading -> {
                // 로딩 중일 때의 Effect
            }
            loginState.errorMessage != null -> {
                // 에러 발생 시 Effect
            }
        }
    }
    
    // UI 렌더링
    Column {
        // ... UI 코드
    }
}
```

**장점:**
* **타입 안전성**: 컴파일 타임에 모든 Effect 케이스 처리 보장
* **명확한 의도**: 각 Effect의 목적이 명확히 구분됨
* **확장성**: 새로운 Effect 추가가 용이함

#### 2) SideEffect
```kotlin
@Composable
fun ThemeAwareScreen() {
    var isDarkMode by remember { mutableStateOf(false) }
    
    SideEffect { // Effect: State 변경 시 시스템 설정 변경
        val window = (LocalView.current.context as? Activity)?.window
        window?.let {
            WindowCompat.setDecorFitsSystemWindows(it, !isDarkMode)
        }
    }
    
    // UI 렌더링
    Switch(
        checked = isDarkMode,
        onCheckedChange = { isDarkMode = it }
    )
}
```

#### 3) DisposableEffect
```kotlin
@Composable
fun LocationTracker() {
    var location by remember { mutableStateOf<Location?>(null) }
    
    DisposableEffect(Unit) { // Effect: 컴포넌트 생명주기에 따른 리스너 등록/해제
        val locationListener = object : LocationListener {
            override fun onLocationChanged(newLocation: Location) {
                location = newLocation
            }
        }
        locationManager.addLocationListener(locationListener)
        
        onDispose {
            locationManager.removeLocationListener(locationListener)
        }
    }
    
    // UI 렌더링
    location?.let { loc ->
        Text("위치: ${loc.latitude}, ${loc.longitude}")
    }
}
```

## 3. State vs Effect의 핵심 차이점

| 구분 | State | Effect |
|------|-------|--------|
| **목적** | UI 표시 데이터 | 부수 효과 처리 |
| **트리거** | 사용자 액션, 외부 이벤트 | State 변경, 생명주기 |
| **결과** | UI 리컴포지션 | 외부 시스템과의 상호작용 |
| **예시** | 텍스트 내용, 체크박스 상태 | API 호출, 로그 기록, 애니메이션 |

### 구체적인 예시로 보는 차이

```kotlin
@Composable
fun UserListScreen() {
    var users by remember { mutableStateOf<List<User>>(emptyList()) } // State
    var isLoading by remember { mutableStateOf(false) } // State
    
    LaunchedEffect(Unit) { // Effect: 컴포넌트 진입 시 API 호출
        isLoading = true
        try {
            users = userRepository.getUsers()
        } finally {
            isLoading = false
        }
    }
    
    // UI 렌더링 (State 기반)
    if (isLoading) {
        CircularProgressIndicator()
    } else {
        LazyColumn {
            items(users) { user ->
                Text(user.name)
            }
        }
    }
}
```

**분석:**
* `users`, `isLoading` → **State**: UI가 표시할 데이터
* `LaunchedEffect` → **Effect**: State 변경을 위한 API 호출

## 4. 언제 State를, 언제 Effect를 쓸까?

### State를 사용하는 경우
* UI에 표시할 데이터가 필요할 때
* 사용자 입력을 받아야 할 때
* 컴포넌트의 현재 상태를 나타낼 때

```kotlin
@Composable
fun LoginForm() {
    var email by remember { mutableStateOf("") } // State
    var password by remember { mutableStateOf("") } // State
    
    Column {
        TextField(
            value = email,
            onValueChange = { email = it } // State 변경
        )
        TextField(
            value = password,
            onValueChange = { password = it } // State 변경
        )
    }
}
```

### Effect를 사용하는 경우
* State 변경에 따른 추가 작업이 필요할 때
* 외부 시스템과 상호작용할 때
* 컴포넌트 생명주기에 따른 작업이 필요할 때

```kotlin
@Composable
fun SearchScreen() {
    var searchQuery by remember { mutableStateOf("") }
    var searchResults by remember { mutableStateOf<List<Item>>(emptyList()) }
    
    LaunchedEffect(searchQuery) { // Effect: 검색어 변경 시 API 호출
        if (searchQuery.isNotEmpty()) {
            searchResults = searchRepository.search(searchQuery)
        }
    }
    
    // UI 렌더링
    LazyColumn {
        items(searchResults) { item ->
            Text(item.title)
        }
    }
}
```

## 5. 주의할 점

### State 사용 시 주의사항
* **불필요한 State는 피하기**: 계산 가능한 값은 State로 만들지 마세요
* **State 호이스팅**: 공통 부모로 State를 끌어올리세요
* **State 정규화**: 복잡한 State는 단순한 형태로 분해하세요

```kotlin
// 나쁜 예: 불필요한 State
@Composable
fun BadExample() {
    var fullName by remember { mutableStateOf("") }
    var firstName by remember { mutableStateOf("") }
    var lastName by remember { mutableStateOf("") }
    
    // fullName은 firstName + lastName으로 계산 가능
}

// 좋은 예: 계산 가능한 값은 State로 만들지 않음
@Composable
fun GoodExample() {
    var firstName by remember { mutableStateOf("") }
    var lastName by remember { mutableStateOf("") }
    
    val fullName = "$firstName $lastName" // 계산된 값
}
```

### Effect 사용 시 주의사항
* **무한 루프 방지**: Effect 내에서 State를 변경할 때 주의하세요
* **적절한 키 사용**: LaunchedEffect의 키를 신중하게 선택하세요
* **리소스 정리**: DisposableEffect에서 onDispose를 잊지 마세요

```kotlin
// 나쁜 예: 무한 루프
@Composable
fun BadEffect() {
    var count by remember { mutableStateOf(0) }
    
    LaunchedEffect(count) { // count가 변경될 때마다 실행
        count++ // 다시 count 변경 → 무한 루프!
    }
}

// 좋은 예: 적절한 키 사용
@Composable
fun GoodEffect() {
    var count by remember { mutableStateOf(0) }
    
    LaunchedEffect(Unit) { // 한 번만 실행
        // 초기화 작업
    }
}
```

## 참고자료

* [State in Compose - Android Developers](https://developer.android.com/jetpack/compose/state)
* [Side-effects in Compose - Android Developers](https://developer.android.com/jetpack/compose/side-effects)
* [Thinking in Compose - 공식 아키텍처 가이드](https://developer.android.com/jetpack/compose/architecture) 
