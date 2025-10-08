# Compile과 Build의 차이

---

## 1. Compile이란?

### 1.1 정의

**Compile(컴파일)** 은 소스 코드를 기계어 또는 중간 언어(바이트코드)로 변환하는 과정입니다.

### 1.2 주요 작업

- **소스 코드 분석**: 문법(Syntax) 검사
- **타입 검사**: 타입 안정성 확인
- **코드 변환**: 소스 코드를 바이트코드(.class) 또는 기계어로 변환
- **오류 검출**: 컴파일 타임 오류 발견

### 1.3 예시

```kotlin
// MainActivity.kt 파일 컴파일
// Kotlin 소스 코드 → JVM 바이트코드(.class)

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}

// 컴파일 결과: MainActivity.class 파일 생성
```

**Android에서의 컴파일**:
```bash
# Kotlin 파일 컴파일
kotlinc MainActivity.kt -include-runtime -d MainActivity.jar

# 결과: .kt → .class (JVM 바이트코드)
```

---

## 2. Build란?

### 2.1 정의

**Build(빌드)** 는 소스 코드를 실행 가능한 결과물(APK, AAB 등)로 만드는 전체 과정입니다.

Compile은 Build 과정의 일부입니다.

### 2.2 주요 작업

1. **소스 코드 컴파일**: .kt/.java 파일을 .class 파일로 변환
2. **리소스 처리**: XML, 이미지, 문자열 등 리소스 파일 처리
3. **의존성 해결**: 라이브러리 다운로드 및 연결
4. **DEX 변환**: .class 파일을 Dalvik Executable(.dex)로 변환
5. **패키징**: APK/AAB 파일 생성
6. **서명**: 디지털 서명 추가
7. **최적화**: ProGuard/R8을 통한 코드 난독화 및 최적화

### 2.3 예시

```kotlin
// build.gradle.kts
android {
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
    
    buildTypes {
        release {
            isMinifyEnabled = true // ProGuard 최적화
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

**Android에서의 빌드**:
```bash
# Gradle을 통한 빌드
./gradlew assembleDebug

# 전체 과정:
# 1. Kotlin/Java 컴파일 (.kt → .class)
# 2. 리소스 컴파일 (XML → compiled XML)
# 3. DEX 변환 (.class → .dex)
# 4. APK 패키징 (코드 + 리소스 + 라이브러리)
# 5. APK 서명
# 결과: app-debug.apk
```

---

## 3. Compile vs Build 비교

| 구분 | Compile (컴파일) | Build (빌드) |
|------|-----------------|-------------|
| **범위** | 소스 코드 → 바이트코드/기계어 | 전체 프로젝트 → 실행 가능한 파일 |
| **작업** | 코드 변환만 수행 | 컴파일 + 리소스 처리 + 패키징 + 서명 등 |
| **결과물** | .class, .o 등 중간 파일 | .apk, .aab, .exe 등 실행 파일 |
| **포함 관계** | Build의 일부 | Compile을 포함하는 상위 개념 |
| **도구** | kotlinc, javac, gcc 등 | Gradle, Maven, Make 등 |
| **시간** | 상대적으로 빠름 | 상대적으로 느림 (여러 단계 포함) |

---

## 4. Android 빌드 프로세스

### 4.1 전체 빌드 흐름

```kotlin
/**
 * Android 빌드 프로세스
 * 
 * 1. 소스 컴파일
 *    - Kotlin/Java → .class (JVM 바이트코드)
 *    - AIDL → .java
 *    
 * 2. 리소스 처리
 *    - AAPT2: XML 리소스 → Compiled XML
 *    - R.java 생성
 *    
 * 3. DEX 변환
 *    - D8/R8: .class → .dex (Dalvik Executable)
 *    
 * 4. 패키징
 *    - APK Packager: 모든 파일을 APK로 압축
 *    
 * 5. 서명
 *    - APK Signer: 디지털 서명 추가
 *    
 * 6. 정렬
 *    - zipalign: APK 최적화
 */
```

### 4.2 빌드 최적화

```kotlin
// build.gradle.kts
android {
    // 증분 컴파일 활성화 (기본값)
    kotlinOptions {
        incremental = true
    }
    
    // 병렬 빌드 활성화
    // gradle.properties에 설정
    // org.gradle.parallel=true
    // org.gradle.caching=true
    
    // Build Cache 활성화
    buildCache {
        local {
            isEnabled = true
        }
    }
}
```

---
