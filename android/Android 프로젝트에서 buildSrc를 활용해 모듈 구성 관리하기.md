# Android 프로젝트에서 buildSrc를 활용해 모듈 구성 관리하기

> 공통 Android 설정을 매번 `build.gradle.kts` 파일마다 복붙하고 계신가요?  
> 모듈이 많아질수록 비효율은 커지고, 유지보수는 어려워집니다.  
> 이럴 때 `buildSrc` 디렉토리를 활용하면 공통 설정을 코드로 재사용할 수 있어 더욱 효율적인 프로젝트 구성이 가능합니다.  

---


## 1. buildSrc란?

`buildSrc`는 Gradle이 자동으로 인식하는 특별한 디렉토리입니다.  
이 디렉토리 안에서 Kotlin 코드로 빌드 설정을 작성하면, 모든 모듈에서 해당 설정을 재사용할 수 있습니다.  

### 왜 써야 할까?  
- **중복 제거**: compileSdk, jvmTarget 등 반복되는 설정 제거  
- **일관성 확보**: 모든 모듈의 빌드 환경을 하나로 통합  
- **코틀린 기반 설정**: 타입 안정성 + IDE 자동완성 지원  

---

## 2. Version Catalog 연동 (`libs.versions.toml`)

라이브러리 버전 정보는 `libs.versions.toml` 파일에 선언적으로 정리할 수 있습니다.  

```toml
[versions]
kotlin = "2.0.21"
agp = "8.9.2"

[libraries]
gradle = { module = "com.android.tools.build:gradle", version.ref = "agp" }
kotlin = { module = "org.jetbrains.kotlin:kotlin-gradle-plugin", version.ref = "kotlin" }
ksp.plugin = { module = "com.google.devtools.ksp:symbol-processing-gradle-plugin", version = "1.0.21" }
```

※ 앱 설정 값(compileSdk, targetSdk, jvmTarget 등)은 일반적으로 `Config.kt`에서 상수로 관리합니다.  

---

## 3. Config 객체 정의

```kotlin
object Config {
    const val compileSdk = 35
    const val minSdk = 26
    const val targetSdk = 35
    const val versionCode = 1
    const val versionName = "1.0"
    const val jvmTarget = "21"
    const val applicationId = "com.stopstone.practice"
}
```

---

## 4. VersionCatalog 확장 함수 (buildSrc)

```kotlin
import org.gradle.api.artifacts.VersionCatalog

fun VersionCatalog.getLibrary(library: String) = findLibrary(library).get()
```

> 라이브러리 참조 시 편리하게 불러올 수 있도록 `VersionCatalog`를 확장합니다.  

---

## 5. 앱 모듈 설정 (`build.gradle.kts:app`)

```kotlin
plugins {
    id("android-application-module")
    alias(libs.plugins.compose.compiler)
}

android {
    namespace = "com.stopstone.practice"
}

dependencies {
    /*
     여기에 필요한 라이브러리를 Version Catalog 기준으로 추가하세요.
     예시:
     implementation(libs.androidx.core.ktx)
     */
}
```

> 앱 모듈에서 공통 설정 플러그인과 Compose 컴파일러를 적용하고, Version Catalog로 선언된 의존성을 불러옵니다.  

---

## 6. 공통 모듈 설정을 어떻게 사용할까?

`buildSrc`에 정의한 설정 파일은 플러그인처럼 각 모듈에서 다음과 같이 적용할 수 있습니다.  

```kotlin
plugins {
    id("android-application-module")
}
```

> 공통화된 Android Application 설정을 적용합니다. 이를 통해 compileSdk, minSdk, targetSdk 등 반복되는 설정을 줄일 수 있습니다.  

이렇게 하면 `android-application-module.gradle.kts`에 정의된 모든 설정이 해당 모듈에 자동으로 적용됩니다.  
이 방식을 통해 `app`, `core`, `feature` 모듈 등 여러 모듈 간의 설정을 공통화할 수 있습니다.  

---

