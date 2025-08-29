# Android libs.versions.toml

> `libs.versions.toml`은 Android 프로젝트에서 **버전 카탈로그(Version Catalog)** 를 관리하는 설정 파일입니다.  
> Gradle 7.0부터 도입되어 의존성 관리를 더욱 체계적으로 할 수 있게 해주는 기능으로,  
> `gradle/libs.versions.toml` 파일에 모든 라이브러리 버전을 중앙에서 관리합니다.  

## 1. libs.versions.toml이란?

### 1.1. Version Catalog의 특징
- **중앙 집중식 버전 관리**: 한 파일에서 모든 의존성 버전 관리
- **타입 안전성**: IDE 자동완성과 컴파일 타임 오류 감지
- **리팩토링 용이성**: 변경사항이 모든 모듈에 자동 반영
- **Bundle 지원**: 관련 라이브러리들을 그룹으로 관리

### 1.2. 파일 구조
```
project-root/
├── gradle/
│   └── libs.versions.toml  # 버전 카탈로그 파일
├── app/
├── feature/
└── build.gradle.kts
```

## 2. libs.versions.toml 설정하기

### 2.1. Version Catalog 구조
```toml
# gradle/libs.versions.toml
[versions]
# 코어 라이브러리 버전
kotlin = "1.9.0"
compose = "2023.10.01"
compose-compiler = "1.5.3"
android-gradle-plugin = "8.1.0"

# 안드로이드 라이브러리 버전
androidx-core = "1.12.0"
androidx-lifecycle = "2.7.0"
androidx-navigation = "2.7.5"

# 서드파티 라이브러리 버전
retrofit = "2.9.0"
okhttp = "4.12.0"
hilt = "2.48"
room = "2.6.1"

[libraries]
# 코어 라이브러리
kotlin-stdlib = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlin" }
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "androidx-core" }

# Compose 라이브러리
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }

# 네트워킹 라이브러리
retrofit-core = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }

# 의존성 주입
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }

# 데이터베이스
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }

[plugins]
android-application = { id = "com.android.application", version.ref = "android-gradle-plugin" }
android-library = { id = "com.android.library", version.ref = "android-gradle-plugin" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }

[bundles]
# 자주 함께 사용되는 라이브러리들을 묶어서 관리
compose = [
    "compose-ui",
    "compose-ui-graphics", 
    "compose-ui-tooling",
    "compose-material3"
]
networking = [
    "retrofit-core",
    "retrofit-converter-gson",
    "okhttp",
    "okhttp-logging"
]
room = [
    "room-runtime",
    "room-ktx"
]
```

### 2.2. build.gradle.kts에서 사용하기
```kotlin
// app/build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.kapt)
    alias(libs.plugins.hilt)
}

android {
    namespace = "com.example.myapp"
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
}

dependencies {
    // 개별 라이브러리 사용
    implementation(libs.kotlin.stdlib)
    implementation(libs.androidx.core.ktx)
    
    // Compose BOM 사용
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.graphics)
    implementation(libs.compose.ui.tooling)
    implementation(libs.compose.material3)
    
    // Bundle 사용
    implementation(libs.bundles.networking)
    implementation(libs.bundles.room)
    
    // KAPT 컴파일러
    kapt(libs.hilt.compiler)
    kapt(libs.room.compiler)
}
```

## 3. 버전 관리 전략

### 3.1. **중앙 집중식 버전 관리**
```toml
[versions]
# 메이저 버전은 수동으로 관리
kotlin = "1.9.0"
compose = "2023.10.01"

# 마이너/패치 버전은 자동화 가능
androidx-core = "1.12.0"
androidx-lifecycle = "2.7.0"
```

### 3.2. **버전 업데이트 자동화**
```kotlin
// buildSrc/build.gradle.kts
plugins {
    id("com.github.ben-manes.versions") version "0.48.0"
}

// 버전 체크 명령어
// ./gradlew dependencyUpdates
```

### 3.3. **환경별 버전 관리**
```toml
# gradle/libs.versions.toml
[versions]
# 개발 환경
kotlin-dev = "1.9.0"
compose-dev = "2023.10.01"

# 프로덕션 환경  
kotlin-prod = "1.8.20"
compose-prod = "2023.08.00"

[libraries]
kotlin-stdlib-dev = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlin-dev" }
kotlin-stdlib-prod = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlin-prod" }
```

## 4. libs.versions.toml 사용의 장점

### 4.1. **중앙 집중식 버전 관리**
```kotlin
// 기존 방식 - 각 모듈마다 버전 관리
// app/build.gradle.kts
implementation("androidx.core:core-ktx:1.12.0")
implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")

// feature/build.gradle.kts  
implementation("androidx.core:core-ktx:1.12.0")
implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")

// TOML 방식 - 중앙에서 관리
// gradle/libs.versions.toml
androidx-core = "1.12.0"
androidx-lifecycle = "2.7.0"

// 모든 모듈에서
implementation(libs.androidx.core.ktx)
implementation(libs.androidx.lifecycle.viewmodel.ktx)
```

### 4.2. **타입 안전성**
```kotlin
// 기존 방식 - 문자열 오타 가능
implementation("androidx.core:core-ktx:1.12.0") // 오타 가능

// TOML 방식 - 컴파일 타임에 오류 감지
implementation(libs.androidx.core.ktx) // IDE 자동완성, 오타 방지
```

### 4.3. **리팩토링 용이성**
```kotlin
// 라이브러리 이름 변경 시
// gradle/libs.versions.toml에서만 수정하면 모든 모듈에 반영
[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "androidx-core" }
```

### 4.4. **Bundle을 통한 그룹 관리**
```toml
[bundles]
# 관련 라이브러리들을 묶어서 관리
compose = [
    "compose-ui",
    "compose-ui-graphics",
    "compose-ui-tooling", 
    "compose-material3"
]
```
