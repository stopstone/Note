# Android Jetpack Compose Theme, 알고 쓰자

## 1. Theme이란 무엇인가

**Theme**은 앱 전체 또는 특정 컴포저블 트리 하위에서 **색상(Color)**, **서체(Typography)**, **도형(Shape)** 등의 스타일 속성을 일관되게 적용할 수 있도록 설정하는 시스템입니다.

Compose에서는 `MaterialTheme` 컴포저블을 통해 이 Theme을 설정하고 하위 컴포저블들이 해당 스타일 정보를 자연스럽게 상속받습니다.

**즉, Theme은 일종의 스타일 기본값이며, 앱 전반에 걸쳐 일관성 있는 UI를 만들기 위해 필수적인 구성 요소입니다.**

> 참고: 이 문서는 **Material 3 기준**으로 Compose Theme을 설명합니다.
> (Material 2와는 구성 요소 이름 등이 다를 수 있어요)

---

## 2. Theme를 사용하는 이점

- **일관성 있는 디자인 유지**
  - 전체 앱에 동일한 색상, 폰트, 도형 스타일을 적용할 수 있습니다.

- **다크 테마, 라이트 테마 지원이 용이**
  - 시스템 설정에 따라 자동으로 밝은/어두운 테마를 전환할 수 있습니다.

- **Dynamic Color 지원**
  - Android 12 이상에서는 시스템 배경화면 등에서 색상을 추출하여 앱 테마에 반영할 수 있습니다.

- **유지보수성 향상**
  - 색상, 폰트 변경 시 한 곳만 수정하면 전체 앱에 반영할 수 있습니다.

- **코드 간결화**
  - 컴포저블 함수 안에서 색상, 스타일을 직접 지정하지 않고 Theme을 통해 쉽게 적용할 수 있습니다.

- **테스트 및 프리뷰 지원**
  - 다양한 Theme을 적용한 UI를 쉽게 테스트하고 Preview에서 확인할 수 있습니다.

> Dynamic Color란? 시스템 배경화면, 테마 색상 등을 앱 색상으로 동적으로 적용하는 기능입니다.
> Android 12 (API 31) 이상에서 지원합니다.

---

## 3. 기본 사용법

### 3-1. Theme 정의

```kotlin
@Composable
fun ThemeTestTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = false,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

### 3-2. 다크모드를 막고 싶다면?

다크모드 색상을 미지원 하고 싶거나, 정의가 안되었다면 막을 수 있습니다.

다크모드를 막고 싶다면 `darkTheme`를 무조건 `false`로 설정하여 항상 밝은 테마만 사용하도록 강제할 수 있습니다.

```kotlin
@Composable
fun ThemeTestTheme(
    content: @Composable () -> Unit
) {
    val colorScheme = LightColorScheme

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

### 3-3. Theme 적용

```kotlin
@Composable
fun AppScreen() {
    ThemeTestTheme {
        Text(
            text = "Hello Theme",
            style = MaterialTheme.typography.titleLarge,
            color = MaterialTheme.colorScheme.primary
        )
    }
}
```

---

## 4. MaterialTheme 구성 요소

`MaterialTheme`는 `colorScheme`, `typography`, `shapes` 세 가지 주요 속성을 제공합니다.

### 4-1. colorScheme

앱의 색상을 체계적으로 관리하는 시스템입니다.

| 속성 이름 | 의미 |
| :-------- | :--- |
| **primary** | 앱의 주요 색상입니다. 버튼, 토글 등 중요한 인터페이스 요소에 사용합니다. |
| **onPrimary** | primary 색상 위에 올라가는 텍스트나 아이콘 색상입니다. |
| **secondary** | primary를 보조하는 색상입니다. 부가적 인터페이스 요소에 사용합니다. |
| **onSecondary** | secondary 색상 위에 올라가는 텍스트나 아이콘 색상입니다. |
| **background** | 앱 전체 배경색입니다. |
| **onBackground** | background 위에 올라가는 텍스트나 아이콘 색상입니다. |
| **surface** | 카드, 시트 같은 표면 컴포넌트 색상입니다. |
| **onSurface** | surface 위에 올라가는 텍스트나 아이콘 색상입니다. |
| **error** | 오류 상태를 표시하는 색상입니다. |
| **onError** | error 색상 위에 표시할 텍스트나 아이콘 색상입니다. |

**"on" 접두어**는 해당 배경색 위에 표시되는 **콘텐츠(텍스트, 아이콘 등)**의 색상을 의미합니다.

### 4-2. typography

앱 전반에 사용할 서체 스타일을 관리합니다. 기본적으로 Material 3에서는 다음과 같은 텍스트 스타일 계층을 제공합니다.

| 스타일 이름 | 용도 |
| :---------- | :--- |
| **displayLarge**, **displayMedium**, **displaySmall** | 대형 제목 텍스트입니다. |
| **headlineLarge**, **headlineMedium**, **headlineSmall** | 주요 섹션 헤드라인입니다. |
| **titleLarge**, **titleMedium**, **titleSmall** | 일반 제목 텍스트입니다. |
| **bodyLarge**, **bodyMedium**, **bodySmall** | 본문 텍스트입니다. |
| **labelLarge**, **labelMedium**, **labelSmall** | 레이블, 보조 텍스트입니다. |

`MaterialTheme.typography`를 통해 이 스타일들을 가져와 `Text` 컴포저블에 적용할 수 있습니다.

### 4-3. shapes

앱에 사용되는 도형 모서리 스타일을 관리합니다. 버튼, 카드, 텍스트 필드 등의 컴포넌트에 적용됩니다.

| 도형 이름 | 의미 |
| :-------- | :--- |
| **extraSmall**, **small**, **medium**, **large**, **extraLarge** | 모서리 반경에 따라 다양한 크기의 둥근 도형을 설정할 수 있습니다. |

---

## 5. Custom Theme 만들기

### 5-1. 만드는 방법

1. Color, Typography, Shapes 파일을 별도로 생성합니다.
2. ColorScheme, Typography, Shapes 객체를 정의합니다.
3. 이를 `MaterialTheme`에 연결한 Theme 컴포저블을 작성합니다.

### 5-2. 적용하는 방법

작성한 Custom Theme를 앱 최상단에서 감싸주면 됩니다.

```kotlin
@Composable
fun MyCustomTheme(content: @Composable () -> Unit) {
    val colorScheme = MyColorScheme
    val typography = MyTypography
    val shapes = MyShapes

    MaterialTheme(
        colorScheme = colorScheme,
        typography = typography,
        shapes = shapes,
        content = content
    )
}
```

---

## 참고 자료

- [Android Developers - Material Design Theming in Compose](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [HolyKisa 블로그 - Jetpack Compose 테마 정리](https://holykisa.tistory.com/112)
- [Pluu 블로그 - Compose Material 3 가이드](https://pluu.github.io/blog/android/2024/05/23/compose/)

---
