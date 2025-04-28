# Kotlin DSL 정리하기

> Kotlin DSL(Domain-Specific Language)은 코틀린 언어 특성을 기반으로, 특정 도메인에 최적화된 선언적 언어를 구축하는 방법입니다.  
> Android 개발 환경에서는 Gradle 빌드 스크립트, Jetpack Compose, Navigation 등에서 광범위하게 활용되고 있습니다.  
>
> Android 개발에서 Kotlin DSL이 활용되는 방식은, 기존의 명령형(Imperative) 코드를 넘어 선언형(Declarative) 스타일로 작성할 수 있도록 지원하는 것입니다.  

---

## 1. Kotlin DSL란?

Kotlin DSL은 코틀린 언어 특성을 활용해 특정 도메인에 특화된 언어를 만드는 기법입니다. 주요 특징은 다음과 같습니다.

- **람다와 수신 객체(Lambda with Receiver)** 를 활용하여 문법적 구성을 자연스럽게 만듭니다.
- **invoke 연산자** 오버로딩을 통해 메서드 호출처럼 객체를 다룰 수 있습니다.
- **타입 안전성**을 보장하며, IDE 자동완성 및 컴파일 타임 오류 검사가 가능합니다.

Kotlin DSL은 코드가 특정 도메인을 기술할 때, 자연스럽게 그 구조를 드러내도록 돕습니다.  
이를 통해 코드가 더 이상 "어떻게"가 아니라 "무엇을" 기술하는 데 집중하게 만듭니다.

『Kotlin in Action』에서는 DSL을 다음처럼 설명합니다.

- 람다와 수신 객체를 활용하여 블록 구조를 자연스럽게 구성한다.
- invoke 연산자를 오버로딩하여 DSL의 유연성을 강화한다.
- HTML 빌더, 테스트 스크립트, 빌드 스크립트 등에서 적용할 수 있다.

## 2. Gradle 빌드 스크립트에서의 Kotlin DSL 적용

Gradle 빌드 스크립트에서도 Kotlin DSL을 적용할 수 있습니다. 이를 통해 얻을 수 있는 장점은 다음과 같습니다.

- **정적 타입 검사**가 가능하여 빌드 오류를 컴파일 시점에 잡을 수 있습니다.
- **IDE 지원**으로 자동완성, 문서 조회, 리팩토링이 용이합니다.
- **가독성**이 향상되어 유지보수가 편리해집니다.

기존 Groovy DSL에서는 다음과 같이 작성하였습니다.

```groovy
defaultConfig {
    applicationId "com.example.app"
    minSdkVersion 21
    targetSdkVersion 33
}
```

Kotlin DSL로 전환하면 다음과 같이 간결하고 명시적으로 작성할 수 있습니다.

```kotlin
defaultConfig {
    applicationId = "com.example.app"
    minSdk = 21
    targetSdk = 33
}
```

Groovy DSL에서는 속성 설정 시 문자열이나 숫자 리터럴을 직접 명시했지만, Kotlin DSL은 타입이 명확하여 IDE 지원과 컴파일 오류 검사가 가능해졌습니다.  
Kotlin은 정적 타입 언어이므로, IntelliJ 기반 IDE가 코드를 분석하고 타입을 추론하는 데 최적화되어 있어 자동완성과 타입 체크가 매우 강력하게 지원됩니다.  
Kotlin DSL은 JetBrains의 IntelliJ 플랫폼과 밀접하게 통합되어 있어, Groovy DSL 대비 강력한 자동완성, 리팩토링, 타입 추론 기능을 지원합니다.  

Android Studio Giraffe 이후부터는 새 프로젝트 생성 시 기본적으로 Kotlin DSL을 사용합니다.

## 3. 안드로이드 개발에서의 DSL 활용

> Android 개발에서 Kotlin DSL이 활용되는 방식은, 기존의 명령형 코드를 넘어 선언형 스타일로 작성할 수 있도록 지원하는 것입니다.  
> 특히 Jetpack Compose나 Navigation은 코틀린의 람다와 수신 객체를 활용하여 특정 도메인을 설명하는 전용 문법처럼 보이게 구성되어 있습니다.  

### 3.1 Jetpack Compose

Jetpack Compose는 **UI 도메인을 기술하는 DSL**입니다. UI를 선언형으로 작성할 수 있도록 코틀린의 수신 객체와 람다를 적극 활용하여 UI 컴포넌트를 트리 구조로 선언합니다.

```kotlin
Column {
    Text(text = "안녕하세요.")
    Button(onClick = { /* 클릭 동작 */ }) {
        Text("버튼")
    }
}
```

- `Column`이 수신 객체 역할을 하고,
- 그 안에서 `Text`, `Button`을 선언하는 방식입니다.

Compose는 DSL 구조를 통해 직관적인 UI 코드 작성이 가능하며, XML 레이아웃 대비 생산성과 유연성이 크게 향상됩니다.

### 3.2 Navigation DSL

Navigation Compose는 **앱 플로우를 기술하는 DSL**입니다. XML 대신 코틀린 코드로 그래프를 선언할 수 있으며, 타입 안전성과 가독성이 뛰어납니다.

```kotlin
navigation(startDestination = "home") {
    composable("home") { HomeScreen() }
    composable("detail") { DetailScreen() }
}
```

Compose Navigation에서 `navigation` 블록은 서브 그래프를 정의하는 역할을 합니다.  
하나의 큰 NavGraph를 여러 개의 작은 NavGraph로 나누어 관리할 수 있으며, 앱 구조를 모듈화하고 유지보수성을 높이는 데 중요한 역할을 합니다.

## 4. 참고 자료

- [Kotlin 공식 문서 - Type-safe builders](https://kotlinlang.org/docs/type-safe-builders.html)
- [Gradle Kotlin DSL Primer](https://docs.gradle.org/current/userguide/kotlin_dsl.html)
- 『Kotlin in Action』 Chapter 11
- [Android Developers Blog - Kotlin DSL 기본 적용](https://android-developers.googleblog.com/2023/04/kotlin-dsl-is-now-default-for-new-gradle-builds.html)
