# Android 릴리즈모드와 디버그모드

> 안드로이드 앱 개발에서 **빌드 모드(Build Mode)** 는 앱의 성능, 보안, 디버깅 기능에 직접적인 영향을 미칩니다.
> **디버그 모드(Debug Mode)** 는 개발 과정에서 사용하며, **릴리즈 모드(Release Mode)** 는 실제 사용자에게 배포할 때 사용합니다.
> 이 글에서는 두 모드의 차이점과 설정 방법, 주의사항을 정리합니다.

---

## 1. 디버그 모드 (Debug Mode)

### 1-1. 디버그 모드란?

- **정의**: 개발 과정에서 사용하는 빌드 모드
- **목적**: 개발자의 디버깅과 테스트를 위한 최적화
- **특징**: 빠른 빌드, 상세한 로그, 디버깅 정보 포함

### 1-2. 디버그 모드의 특징

```kotlin
// 디버그 모드에서만 실행되는 코드 예시
if (BuildConfig.DEBUG) {
    Log.d("DebugMode", "디버그 정보 출력")
    // 디버깅 전용 기능들
}
```

**주요 특징:**
- **빠른 빌드 속도**: 최적화 과정을 생략하여 빠른 빌드
- **디버깅 정보 포함**: 스택 트레이스, 로그 정보 등 포함
- **서명 키**: 개발용 디버그 키로 서명
- **프로가드 비활성화**: 코드 난독화 및 최적화 비활성화
- **로그 출력**: 모든 로그 레벨 출력 가능

### 1-3. 디버그 모드 설정

```gradle
android {
    buildTypes {
        debug {
            debuggable true
            minifyEnabled false
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.debug
        }
    }
}
```

---

## 2. 릴리즈 모드 (Release Mode)

### 2-1. 릴리즈 모드란?

- **정의**: 실제 사용자에게 배포하는 빌드 모드
- **목적**: 성능 최적화와 보안 강화
- **특징**: 최적화된 성능, 보안 강화, 최소한의 로그

### 2-2. 릴리즈 모드의 특징

```kotlin
// 릴리즈 모드에서 로그 제한 예시
if (BuildConfig.DEBUG) {
    Log.d("DebugInfo", "디버그 정보")
} else {
    // 릴리즈 모드에서는 중요한 로그만 출력
    Log.e("ErrorInfo", "에러 정보")
}
```

**주요 특징:**
- **성능 최적화**: 코드 최적화 및 압축으로 성능 향상
- **보안 강화**: 코드 난독화, 디버깅 정보 제거
- **서명 키**: 릴리즈용 키로 서명
- **프로가드 활성화**: 코드 난독화 및 최적화
- **로그 제한**: 에러 로그만 출력하거나 로그 비활성화

### 2-3. 릴리즈 모드 설정

```gradle
android {
    buildTypes {
        release {
            debuggable false
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            
            // 릴리즈 모드 전용 설정
            buildConfigField "String", "API_BASE_URL", "\"https://api.production.com\""
            buildConfigField "boolean", "ENABLE_LOGGING", "false"
        }
    }
}
```

---

## 3. 빌드 모드 설정 방법

### 3-1. Gradle 설정

```gradle
android {
    signingConfigs {
        debug {
            storeFile file('debug.keystore')
            storePassword 'android'
            keyAlias 'androiddebugkey'
            keyPassword 'android'
        }
        
        release {
            storeFile file('release.keystore')
            storePassword System.getenv('KEYSTORE_PASSWORD')
            keyAlias System.getenv('KEY_ALIAS')
            keyPassword System.getenv('KEY_PASSWORD')
        }
    }
    
    buildTypes {
        debug {
            debuggable true
            minifyEnabled false
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.debug
            
            // 디버그 모드 전용 설정
            buildConfigField "String", "API_BASE_URL", "\"https://api.staging.com\""
            buildConfigField "boolean", "ENABLE_LOGGING", "true"
        }
        
        release {
            debuggable false
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            
            // 릴리즈 모드 전용 설정
            buildConfigField "String", "API_BASE_URL", "\"https://api.production.com\""
            buildConfigField "boolean", "ENABLE_LOGGING", "false"
        }
    }
}
```

### 3-2. 코드에서 빌드 모드 확인

```kotlin
class BuildConfigHelper {
    companion object {
        fun isDebugMode(): Boolean {
            return BuildConfig.DEBUG
        }
        
        fun getApiBaseUrl(): String {
            return BuildConfig.API_BASE_URL
        }
        
        fun shouldEnableLogging(): Boolean {
            return BuildConfig.ENABLE_LOGGING
        }
    }
}

// 사용 예시
class NetworkManager {
    fun makeApiCall() {
        if (BuildConfigHelper.shouldEnableLogging()) {
            Log.d("NetworkManager", "API 호출 시작")
        }
        
        // API 호출 로직
    }
}
```

### 3-3. 빌드 명령어

```bash
# 디버그 모드 빌드
./gradlew assembleDebug

# 릴리즈 모드 빌드
./gradlew assembleRelease

# 특정 모드로 앱 실행
./gradlew installDebug
./gradlew installRelease
```

---

## 4. 주의사항

### 4-1. 릴리즈 모드 주의사항

**보안 관련:**
- 민감한 정보가 로그에 출력되지 않도록 주의
- 디버그 전용 코드가 릴리즈에 포함되지 않도록 확인
- API 키나 비밀번호 등이 코드에 하드코딩되지 않도록 주의

```kotlin
// 잘못된 예시 - 릴리즈에서도 로그 출력
Log.d("UserInfo", "사용자 비밀번호: $password")

// 올바른 예시 - 빌드 모드에 따른 조건부 로그
if (BuildConfig.DEBUG) {
    Log.d("UserInfo", "사용자 정보: $userInfo")
}
```

**성능 관련:**
- 릴리즈 모드에서 불필요한 연산 제거
- 메모리 누수 확인
- ANR 발생 가능성 체크

### 4-2. 디버그 모드 주의사항

**개발 효율성:**
- 디버그 전용 기능이 실제 동작에 영향을 주지 않도록 주의
- 테스트 데이터와 실제 데이터 구분
- 디버그 로그가 너무 많아지지 않도록 관리

### 4-3. 공통 주의사항

**환경 설정:**
- API 엔드포인트, 데이터베이스 설정 등 환경별 분리
- 빌드 모드별 리소스 파일 관리
- 서명 키 보안 관리

```kotlin
// 환경별 설정 관리 예시
object AppConfig {
    val apiBaseUrl: String = BuildConfig.API_BASE_URL
    val enableLogging: Boolean = BuildConfig.ENABLE_LOGGING
    val isDebugMode: Boolean = BuildConfig.DEBUG
    
    fun getDatabaseName(): String {
        return if (isDebugMode) "app_debug.db" else "app.db"
    }
}
```

---
