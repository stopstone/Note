# Compose에서 상태바 아이콘 색상 제어

> Jetpack Compose에서 `SideEffect`와 `WindowCompat`를 활용하여 상태바 아이콘 색상을 테마에 맞게 조정하는 방법을 알아봅니다.

---

## 1. 기본 개념

**상태바 아이콘 색상**은 Android 기기 상단의 시간, 배터리, 신호 등의 아이콘 색상입니다.

### 1-1. Edge-to-Edge 모드에서의 문제

`enableEdgeToEdge()`를 사용하면 상태바 영역까지 콘텐츠가 침범하여 **상태바 아이콘이 가려지는 현상**이 발생할 수 있습니다.

```kotlin
// 문제 상황: enableEdgeToEdge() 사용 시
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge() // 상태바 영역까지 콘텐츠 침범
        
        setContent {
            Box(modifier = Modifier.fillMaxSize()) {
                // 상태바 아이콘이 이 콘텐츠에 가려짐!
            }
        }
    }
}
```

| 모드 | 아이콘 색상 | 사용 시기 |
|------|-------------|-----------|
| **Light Mode** | 어두운 색 | 밝은 배경 위에서 |
| **Dark Mode** | 밝은 색 | 어두운 배경 위에서 |

---

## 2. 기본 구현

### 2-1. 필요한 의존성
```kotlin
implementation("androidx.core:core-ktx:1.12.0")
```

### 2-2. 기본 코드
```kotlin
@Composable
fun ThemeWithStatusBar(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme
    
    /* 스크린에서 상태바 아이콘 색상 제어 */
    val view = LocalView.current
    val isDarkTheme = darkTheme

    SideEffect {
        if (!view.isInEditMode) {
            val window = (view.context as Activity).window
            val insetsController = WindowCompat.getInsetsController(window, view)
            insetsController.isAppearanceLightStatusBars = !isDarkTheme
        }
    }

    MaterialTheme(
        colorScheme = colorScheme,
        content = content
    )
}
```

### 2-3. 핵심 요소
- **`LocalView.current`**: 현재 Compose UI View
- **`SideEffect`**: 부수 효과 처리
- **`WindowCompat.getInsetsController()`**: Window InsetsController
- **`isAppearanceLightStatusBars`**: 아이콘 밝기 모드 설정

**동작 원리:**
- `true`: 어두운 아이콘 (라이트 테마용)
- `false`: 밝은 아이콘 (다크 테마용)

---

## 3. 고급 활용

### 3-1. Dynamic Color와 함께
```kotlin
@Composable
fun DynamicThemeWithStatusBar(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    
    val view = LocalView.current

    SideEffect {
        if (!view.isInEditMode) {
            val window = (view.context as Activity).window
            val insetsController = WindowCompat.getInsetsController(window, view)
            val shouldUseLightStatusBars = !darkTheme && !dynamicColor
            insetsController.isAppearanceLightStatusBars = shouldUseLightStatusBars
        }
    }

    MaterialTheme(colorScheme = colorScheme, content = content)
}
```

---

## 4. 실전 활용

### 4-1. 이미지 배경
```kotlin
@Composable
fun ImageBackgroundScreen(imageUrl: String, content: @Composable () -> Unit) {
    val view = LocalView.current
    var isImageLoaded by remember { mutableStateOf(false) }
    
    SideEffect {
        if (!view.isInEditMode && isImageLoaded) {
            val window = (view.context as Activity).window
            val insetsController = WindowCompat.getInsetsController(window, view)
            insetsController.isAppearanceLightStatusBars = false
        }
    }
    
    Box(modifier = Modifier.fillMaxSize()) {
        AsyncImage(
            model = imageUrl,
            contentDescription = null,
            modifier = Modifier.fillMaxSize(),
            contentScale = ContentScale.Crop,
            onSuccess = { isImageLoaded = true }
        )
        content()
    }
}
```

---

## 5. 문제 해결

### 5-1. 일반적인 문제들

**상태바 색상이 변경되지 않음**
```kotlin
// 해결: EditMode 체크 추가
SideEffect {
    if (!view.isInEditMode) {  // 이 조건이 중요!
        val window = (view.context as Activity).window
        val insetsController = WindowCompat.getInsetsController(window, view)
        insetsController.isAppearanceLightStatusBars = !isDarkTheme
    }
}
```

**일부 기기에서 동작하지 않음**
```kotlin
// 해결: API 레벨 체크 추가
SideEffect {
    if (!view.isInEditMode && Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        val window = (view.context as Activity).window
        val insetsController = WindowCompat.getInsetsController(window, view)
        insetsController.isAppearanceLightStatusBars = !isDarkTheme
    }
}
```

**Edge-to-Edge 모드에서 상태바 아이콘이 가려짐**
```kotlin
// 문제: enableEdgeToEdge() 사용 시 콘텐츠가 상태바 영역을 침범하여 아이콘이 가려짐
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge() // 상태바 영역까지 콘텐츠 침범
        
        setContent {
            ThemeWithStatusBar {
                // 상태바 아이콘이 콘텐츠에 가려짐
            }
        }
    }
}

// 해결: WindowInsets를 활용한 패딩 추가
SideEffect {
    if (!view.isInEditMode) {
        val window = (view.context as Activity).window
        val insetsController = WindowCompat.getInsetsController(window, view)
        insetsController.isAppearanceLightStatusBars = !isDarkTheme
    }
}

// UI에서 상태바 영역만큼 패딩 추가
Box(
    modifier = Modifier
        .fillMaxSize()
        .padding(top = WindowInsets.statusBars.asPaddingValues().calculateTopPadding())
) {
    // 콘텐츠
}
```

---
