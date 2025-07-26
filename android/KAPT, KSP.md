# KAPT, KSP

> Kotlin에서 어노테이션 처리를 담당하는 **KAPT**와 **KSP**에 대해 알아보고      
> Annotation Processing의 개념부터 KAPT의 한계, KSP의 등장 배경과 장점, 그리고 실무에서의 활용 방법까지 정리해보겠습니다.  

---

## 1. Annotation Processing이란?

Annotation Processing은 컴파일 타임에 어노테이션을 분석하고, 새로운 소스 코드를 생성하는 기술입니다.

### 1-1. 기본 개념

개발자가 작성한 어노테이션이 컴파일 타임에 실제 코드로 변환되는 과정을 의미합니다.

```kotlin
// 개발자가 작성하는 코드
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    val name: String,
    val email: String
)

// 컴파일 타임에 자동 생성되는 코드 (개발자는 보지 못함)
class UserDao_Impl : UserDao {
    override fun insertUser(user: User) {
        // Room이 자동으로 생성한 SQL 쿼리 실행 코드
    }
}
```

### 1-2. Annotation Processing이 필요한 이유

- **보일러플레이트 코드 제거**: 반복적인 코드 작성 방지
- **타입 안전성**: 컴파일 타임에 오류 검출
- **성능 최적화**: 런타임 리플렉션 대신 컴파일 타임 코드 생성

## 2. KAPT (Kotlin Annotation Processing Tool)

### 2-1. KAPT란?

KAPT는 Kotlin에서 Java의 Annotation Processing Tool(APT)을 사용할 수 있게 해주는 도구입니다.

### 2-2. KAPT의 동작 원리

```kotlin
// build.gradle.kts
plugins {
    id("kotlin-kapt")
}

dependencies {
    kapt("androidx.room:room-compiler:2.6.0")
    implementation("androidx.room:room-runtime:2.6.0")
}
```

**KAPT의 처리 과정:**
1. Kotlin 소스 → Java 스텁 생성
2. Java 스텁 → Java APT 실행
3. Java APT → 새로운 Java 코드 생성
4. 생성된 Java 코드 → Kotlin으로 변환

### 2-3. KAPT의 한계점

KAPT는 Kotlin과 Java 간의 호환성을 위해 여러 단계의 변환 과정을 거치기 때문에 성능상 문제가 있습니다.

1. **빌드 성능 문제**
   - Kotlin → Java 스텁 생성 오버헤드
   - Java APT → Kotlin 변환 오버헤드
   - 빌드 시간이 2-3배 증가

2. **메모리 사용량**
   - 스텁 파일 생성으로 인한 메모리 낭비
   - 대규모 프로젝트에서 OOM 발생 가능

3. **Kotlin 특성 미지원**
   - Kotlin의 고급 타입 시스템 활용 불가
   - Nullable 타입 정보 손실

## 3. KSP (Kotlin Symbol Processing)

### 3-1. KSP란?

KSP는 Kotlin 전용으로 설계된 Symbol Processing API입니다. KAPT의 한계를 극복하기 위해 등장했습니다.

### 3-2. KSP의 등장 배경

KAPT의 성능 문제를 해결하기 위해 Google에서 개발한 새로운 어노테이션 처리 도구입니다.

- 2019년 Google에서 KSP 프로젝트 시작
- 2021년 정식 릴리즈
- Room, Hilt, Moshi 등 주요 라이브러리들이 KSP 지원

### 3-3. KSP의 장점

#### 3-3-1. 빌드 성능 향상

```kotlin
// build.gradle.kts
plugins {
    id("com.google.devtools.ksp")
}

dependencies {
    ksp("androidx.room:room-compiler:2.6.0")
    implementation("androidx.room:room-runtime:2.6.0")
}
```

**성능 비교:**
- KAPT: 빌드 시간 100% (기준)
- KSP: 빌드 시간 20-40% 감소 (프로젝트 크기에 따라 다름)

#### 3-3-2. Kotlin 네이티브 지원

```kotlin
// KSP에서는 Kotlin 타입 정보를 완벽하게 활용
class MyProcessor : SymbolProcessor {
    override fun process(resolver: Resolver): List<KSAnnotated> {
        val symbols = resolver.getSymbolsWithAnnotation("MyAnnotation")
        
        symbols.forEach { symbol ->
            when (symbol) {
                is KSClassDeclaration -> {
                    // Kotlin 클래스 정보 완벽 활용
                    symbol.primaryConstructor?.parameters?.forEach { param ->
                        // Nullable 타입 정보도 정확히 파악 가능
                        val isNullable = param.type.resolve().isMarkedNullable
                    }
                }
            }
        }
        return emptyList()
    }
}
```

#### 3-3-3. 메모리 효율성

- 스텁 파일 생성 없음
- 직접 Kotlin AST 처리
- 메모리 사용량 30-50% 감소 (스텁 파일 생성 오버헤드 제거)

## 4. KAPT vs KSP 비교

| 구분 | KAPT | KSP |
|------|------|-----|
| **빌드 성능** | 느림 | 빠름 |
| **메모리 사용량** | 높음 | 낮음 |
| **Kotlin 지원** | 제한적 | 완벽 |
| **학습 곡선** | 쉬움 | 보통 |
| **라이브러리 지원** | 모든 Java APT | 점진적 확대 |

## 5. 실제 사용 예시

### 5-1. Room + KSP

```kotlin
// build.gradle.kts
plugins {
    id("com.google.devtools.ksp")
}

dependencies {
    ksp("androidx.room:room-compiler:2.6.0")
    implementation("androidx.room:room-runtime:2.6.0")
    implementation("androidx.room:room-ktx:2.6.0")
}
```

### 5-2. Hilt + KSP

```kotlin
// build.gradle.kts
plugins {
    id("com.google.devtools.ksp")
    id("dagger.hilt.android.plugin")
}

dependencies {
    ksp("com.google.dagger:hilt-compiler:2.48")
    implementation("com.google.dagger:hilt-android:2.48")
}
```

### 5-3. Moshi + KSP

```kotlin
// build.gradle.kts
dependencies {
    ksp("com.squareup.moshi:moshi-kotlin-codegen:1.14.0")
    implementation("com.squareup.moshi:moshi:1.14.0")
}
```

## 6. KAPT에서 KSP로 마이그레이션

### 6-1. 1단계: 플러그인 변경

```kotlin
// 기존 (KAPT)
plugins {
    id("kotlin-kapt")
}

// 변경 후 (KSP)
plugins {
    id("com.google.devtools.ksp")
}
```

### 6-2. 2단계: 의존성 변경

```kotlin
// 기존 (KAPT)
dependencies {
    kapt("androidx.room:room-compiler:2.6.0")
    kapt("com.google.dagger:hilt-compiler:2.48")
}

// 변경 후 (KSP)
dependencies {
    ksp("androidx.room:room-compiler:2.6.0")
    ksp("com.google.dagger:hilt-compiler:2.48")
}
```

### 6-3. 3단계: 설정 파일 정리

```kotlin
// kapt 관련 설정 제거
android {
    // kaptOptions 제거
    // kapt {
    //     correctErrorTypes = true
    //     useBuildCache = true
    // }
}
```

## 참고 자료

- [공식 문서 - KSP Overview](https://kotlinlang.org/docs/ksp-overview.html)
- [Room KSP 가이드](https://developer.android.com/jetpack/compose/libraries#room)
- [Hilt KSP 가이드](https://dagger.dev/hilt/gradle-setup#ksp)
- [Moshi KSP 가이드](https://github.com/square/moshi#codegen) 
