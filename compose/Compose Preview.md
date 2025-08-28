# Compose Preview

> **Compose Preview** 는 Jetpack Compose UI를 실시간으로 미리보기할 수 있는 도구입니다.  
> ViewModel이 포함된 복잡한 화면부터 간단한 컴포넌트까지, 다양한 상황에서 UI를 빠르게 확인하고 개발할 수 있습니다.  
> 이 글에서는 Compose Preview의 기본 사용법, ViewModel 통합, Preview 어노테이션 속성들을 정리합니다.  

---

## 1. Compose Preview란?

### 1-1. Compose Preview의 정의

- **정의**: Jetpack Compose UI를 Android Studio에서 실시간으로 미리보기하는 기능
- **목적**: UI 개발 속도 향상 및 디자인 검증
- **특징**: 실시간 렌더링, 다양한 기기/테마 미리보기, 인터랙션 테스트

### 1-2. Compose Preview의 장점

```kotlin
// Preview 없이 개발할 때 - 앱 실행 필요
@Composable
fun UserProfileScreen() {
    // UI 코드 작성 후 앱을 실행해서 확인
    // 변경사항이 있을 때마다 앱 재실행 필요
}

// Preview 사용할 때 - 즉시 확인 가능
@Preview(showBackground = true)
@Composable
fun UserProfileScreenPreview() {
    UserProfileScreen()
}
```

**주요 장점:**
- **빠른 개발**: 앱 실행 없이 UI 즉시 확인
- **실시간 피드백**: 코드 변경 시 즉시 반영
- **다양한 환경 테스트**: 여러 기기, 테마, 다크모드 미리보기
- **디자인 검증**: 디자이너와의 협업 효율성 증대
- **코드 품질 향상**: UI 로직과 비즈니스 로직 분리

---

## 2. 기본 Preview 사용법

### 2-1. 간단한 Preview

```kotlin
@Composable
fun GreetingCard(name: String) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = "Hello $name!",
                style = MaterialTheme.typography.headlineMedium
            )
            Text(
                text = "Welcome to Compose Preview",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}

@Preview(showBackground = true)
@Composable
fun GreetingCardPreview() {
    MaterialTheme {
        GreetingCard("Android Developer")
    }
}
```

### 2-2. 여러 Preview 동시 사용

```kotlin
@Preview(
    name = "Light Mode",
    showBackground = true,
    uiMode = Configuration.UI_MODE_NIGHT_NO
)
@Preview(
    name = "Dark Mode", 
    showBackground = true,
    uiMode = Configuration.UI_MODE_NIGHT_YES
)
@Composable
fun GreetingCardPreview() {
    MaterialTheme {
        GreetingCard("Android Developer")
    }
}
```

---

## 3. ViewModel이 포함된 화면 Preview

### 3-1. 기본 ViewModel Preview

```kotlin
// ViewModel 정의
class UserProfileViewModel : ViewModel() {
    private val _userState = MutableStateFlow(UserState())
    val userState: StateFlow<UserState> = _userState.asStateFlow()
    
    init {
        // Preview용 더미 데이터
        _userState.value = UserState(
            name = "김개발",
            email = "dev@example.com",
            profileImage = "https://example.com/profile.jpg",
            isLoading = false
        )
    }
}

// UI 컴포넌트
@Composable
fun UserProfileScreen(
    viewModel: UserProfileViewModel = hiltViewModel()
) {
    val userState by viewModel.userState.collectAsState()
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        if (userState.isLoading) {
            CircularProgressIndicator()
        } else {
            UserProfileContent(userState)
        }
    }
}

// Preview용 ViewModel 생성
@Preview(showBackground = true)
@Composable
fun UserProfileScreenPreview() {
    val previewViewModel = UserProfileViewModel()
    
    MaterialTheme {
        UserProfileScreen(viewModel = previewViewModel)
    }
}
```

### 3-2. Hilt ViewModel Preview

```kotlin
// Hilt ViewModel
@HiltViewModel
class ProductListViewModel @Inject constructor(
    private val productRepository: ProductRepository
) : ViewModel() {
    private val _products = MutableStateFlow<List<Product>>(emptyList())
    val products: StateFlow<List<Product>> = _products.asStateFlow()
    
    init {
        loadProducts()
    }
    
    private fun loadProducts() {
        viewModelScope.launch {
            _products.value = productRepository.getProducts()
        }
    }
}

// Preview용 더미 데이터
@Preview(showBackground = true)
@Composable
fun ProductListScreenPreview() {
    val previewViewModel = remember {
        ProductListViewModel(
            ProductRepositoryImpl() // 더미 구현체
        )
    }
    
    MaterialTheme {
        ProductListScreen(viewModel = previewViewModel)
    }
}
```

### 3-3. 상태별 Preview

```kotlin
@Composable
fun LoginScreen(
    viewModel: LoginViewModel = hiltViewModel()
) {
    val loginState by viewModel.loginState.collectAsState()
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        when (loginState) {
            is LoginState.Initial -> LoginForm(
                onLoginClick = { email, password ->
                    viewModel.login(email, password)
                }
            )
            is LoginState.Loading -> CircularProgressIndicator()
            is LoginState.Success -> LoginSuccessContent()
            is LoginState.Error -> LoginErrorContent(
                message = loginState.message,
                onRetry = { viewModel.retry() }
            )
        }
    }
}

// 각 상태별 Preview
@Preview(name = "Initial State")
@Composable
fun LoginScreenInitialPreview() {
    val previewViewModel = remember {
        LoginViewModel().apply {
            // 초기 상태로 설정
        }
    }
    
    MaterialTheme {
        LoginScreen(viewModel = previewViewModel)
    }
}

@Preview(name = "Loading State")
@Composable
fun LoginScreenLoadingPreview() {
    val previewViewModel = remember {
        LoginViewModel().apply {
            // 로딩 상태로 설정
            setLoginState(LoginState.Loading)
        }
    }
    
    MaterialTheme {
        LoginScreen(viewModel = previewViewModel)
    }
}

@Preview(name = "Error State")
@Composable
fun LoginScreenErrorPreview() {
    val previewViewModel = remember {
        LoginViewModel().apply {
            // 에러 상태로 설정
            setLoginState(LoginState.Error("로그인에 실패했습니다."))
        }
    }
    
    MaterialTheme {
        LoginScreen(viewModel = previewViewModel)
    }
}
```

---

## 4. Preview 어노테이션 속성들

### 4-1. 기본 속성

```kotlin
@Preview(
    name = "Custom Preview Name",           // Preview 이름 지정
    showBackground = true,                  // 배경 표시 여부
    backgroundColor = 0xFF00FF00,          // 배경색 (ARGB)
    showSystemUi = false,                  // 시스템 UI 표시 여부
    group = "User Interface"               // Preview 그룹화
)
@Composable
fun CustomPreviewExample() {
    Text("Hello Preview!")
}
```

### 4-2. 기기 및 화면 속성

```kotlin
@Preview(
    name = "Phone Preview",
    device = Devices.PHONE,                // 기기 타입
    widthDp = 360,                        // 너비 (dp)
    heightDp = 640,                       // 높이 (dp)
    locale = "ko"                         // 로케일 설정
)
@Preview(
    name = "Tablet Preview",
    device = Devices.TABLET,
    widthDp = 600,
    heightDp = 800
)
@Composable
fun ResponsivePreviewExample() {
    ResponsiveLayout()
}
```

### 4-3. 테마 및 UI 모드 속성

```kotlin
@Preview(
    name = "Light Theme",
    uiMode = Configuration.UI_MODE_NIGHT_NO,    // 라이트 모드
    showBackground = true
)
@Preview(
    name = "Dark Theme",
    uiMode = Configuration.UI_MODE_NIGHT_YES,   // 다크 모드
    showBackground = true
)
@Preview(
    name = "Large Font",
    fontScale = 1.5f                            // 폰트 크기 스케일
)
@Composable
fun ThemePreviewExample() {
    MaterialTheme {
        Column {
            Text(
                text = "Hello World",
                style = MaterialTheme.typography.headlineMedium
            )
            Button(onClick = {}) {
                Text("Click Me")
            }
        }
    }
}
```

### 4-4. 고급 속성

```kotlin
@Preview(
    name = "System UI Preview",
    showSystemUi = true,                   // 시스템 UI 표시
    systemUi = SystemUiMode.EDGE_TO_EDGE   // 엣지 투 엣지 모드
)
@Preview(
    name = "Landscape Preview",
    device = Devices.PHONE,
    widthDp = 640,
    heightDp = 360
)
@Preview(
    name = "Pixel 6 Preview",
    device = "spec:shape=Normal,width=411,height=914,unit=dp,dpi=420"  // 커스텀 기기 스펙
)
@Composable
fun AdvancedPreviewExample() {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(MaterialTheme.colorScheme.background)
    ) {
        Text(
            text = "Advanced Preview",
            modifier = Modifier.align(Alignment.Center)
        )
    }
}
```
