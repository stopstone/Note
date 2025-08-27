# ktlint

> **ktlint** 는 Kotlin 코드의 스타일을 자동으로 검사하고 수정하는 도구입니다.  
> 개발팀의 코드 일관성을 유지하고, 코드 리뷰 시간을 단축하며, 깔끔한 코드베이스를 유지하는 데 도움을 줍니다.  
> 이 글에서는 ktlint의 설치 방법, 설정, 사용법, 그리고 Android 프로젝트에서의 활용 방법을 정리합니다.  

---

## 1. ktlint란?

### 1-1. ktlint의 정의

- **정의**: Kotlin 코드 스타일 검사 및 자동 수정 도구
- **목적**: 일관된 코드 스타일 유지 및 자동 포맷팅
- **특징**: 공식 Kotlin 스타일 가이드 준수, 플러그인 확장 가능

### 1-2. ktlint의 장점

```kotlin
// ktlint 적용 전 - 일관성 없는 코드 스타일
class UserManager{
    private val userRepository:UserRepository
    fun getUser(id:Int):User?{
        return userRepository.findById(id)
    }
}

// ktlint 적용 후 - 일관된 코드 스타일
class UserManager {
    private val userRepository: UserRepository
    
    fun getUser(id: Int): User? {
        return userRepository.findById(id)
    }
}
```

**주요 장점:**
- **일관된 코드 스타일**: 팀 전체의 코드 스타일 통일
- **자동 수정**: 대부분의 스타일 문제를 자동으로 수정
- **CI/CD 통합**: 빌드 과정에서 자동 검사
- **시간 절약**: 수동 코드 포맷팅 시간 단축
- **코드 품질 향상**: 가독성과 유지보수성 개선

---

## 2. ktlint 설치 및 설정

### 2-1. Gradle 플러그인 설치

```gradle
// 프로젝트 레벨 build.gradle
buildscript {
    dependencies {
        classpath "org.jlleitschuh.gradle:ktlint-gradle:11.3.1"
    }
}

// 앱 레벨 build.gradle
plugins {
    id "org.jlleitschuh.gradle.ktlint" version "11.3.1"
}

// 또는 최신 방식 (Gradle 7.0+)
plugins {
    id "org.jlleitschuh.gradle.ktlint" version "11.3.1"
}
```

### 2-2. 기본 설정

```gradle
ktlint {
    // ktlint 버전 설정
    version.set("0.48.2")
    
    // 안드로이드 스타일 적용
    android.set(true)
    
    // 출력 형식 설정
    verbose.set(true)
    
    // 색상 출력 활성화
    coloredOutput.set(true)
    
    // 추가 규칙 설정
    additionalEditorconfigFile.set(file(".editorconfig"))
    
    // 제외할 파일 설정
    filter {
        exclude { element -> element.file.path.contains("build/") }
        exclude { element -> element.file.path.contains("generated/") }
    }
}
```

### 2-3. .editorconfig 파일 설정

```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{kt,kts}]
# ktlint 규칙 설정
ktlint_code_style = android_studio
ktlint_ignore_back_ticked_identifier = true
ktlint_standard_no-wildcard-imports = disabled
ktlint_standard_filename = disabled

# 들여쓰기 설정
indent_style = space
indent_size = 4

# 최대 줄 길이
max_line_length = 100
```

---

## 3. ktlint 사용법

### 3-1. 기본 명령어

```bash
# 코드 스타일 검사
./gradlew ktlintCheck

# 코드 스타일 자동 수정
./gradlew ktlintFormat

# 특정 파일만 검사
./gradlew ktlintCheck --continue

# 특정 모듈만 검사
./gradlew :app:ktlintCheck
```

### 3-2. IDE 플러그인 설치

**Android Studio / IntelliJ IDEA:**
1. Settings → Plugins → Marketplace
2. "ktlint" 검색
3. "ktlint (unofficial)" 플러그인 설치
4. 재시작 후 자동 포맷팅 활성화

**VS Code:**
```json
// settings.json
{
    "kotlin.formatOnSave": true,
    "kotlin.linting.ktlint.enable": true
}
```

### 3-3. Git Hooks 설정

```bash
# pre-commit hook 설정
./gradlew addKtlintCheckGitPreCommitHook

# pre-push hook 설정
./gradlew addKtlintCheckGitPrePushHook
```

---

## 4. ktlint 규칙 및 설정

### 4-1. 주요 규칙

```kotlin
// 1. 들여쓰기 규칙
class Example {
    fun method() {
        if (condition) {
            // 4칸 들여쓰기
        }
    }
}

// 2. 공백 규칙
val list = listOf(1, 2, 3) // 콤마 뒤 공백
fun method(param: String) { } // 콜론 뒤 공백

// 3. 줄바꿈 규칙
val longString = "매우 긴 문자열을 " +
    "여러 줄로 나누어 작성"

// 4. 임포트 규칙
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
// 와일드카드 임포트 금지: import androidx.lifecycle.*
```

### 4-2. 규칙 비활성화

```kotlin
// 특정 라인에서 규칙 비활성화
@Suppress("ktlint:standard:no-wildcard-imports")
import androidx.lifecycle.*

// 특정 블록에서 규칙 비활성화
@Suppress("ktlint:standard:no-wildcard-imports")
class Example {
    // 이 블록 내에서는 와일드카드 임포트 허용
}

// 파일 전체에서 규칙 비활성화
@file:Suppress("ktlint:standard:no-wildcard-imports")
package com.example.app
```

### 4-3. 커스텀 규칙 설정

```gradle
ktlint {
    // 커스텀 규칙 추가
    customRuleset {
        dependencies {
            customRuleset("com.github.username:custom-ktlint-rules:1.0.0")
        }
    }
    
    // 특정 규칙 비활성화
    filter {
        exclude { element ->
            element.file.path.contains("generated/")
        }
    }
}
```

---

## 5. Android 프로젝트에서의 활용

### 5-1. 멀티 모듈 설정

```gradle
// 프로젝트 레벨 build.gradle
subprojects {
    apply plugin: "org.jlleitschuh.gradle.ktlint"
    
    ktlint {
        android.set(true)
        verbose.set(true)
        
        // 모듈별 설정
        if (project.name == "app") {
            filter {
                exclude { element ->
                    element.file.path.contains("generated/")
                }
            }
        }
    }
}
```

### 5-2. CI/CD 통합

```yaml
# GitHub Actions 예시
name: ktlint

on: [push, pull_request]

jobs:
  ktlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Run ktlint
        run: ./gradlew ktlintCheck
```

### 5-3. 빌드 스크립트 통합

```gradle
// 앱 레벨 build.gradle
android {
    // 기존 설정...
}

// ktlint를 빌드 과정에 통합
tasks.withType(Test) {
    dependsOn 'ktlintCheck'
}

tasks.withType(JavaCompile) {
    dependsOn 'ktlintCheck'
}
```

---

**참고 자료:**
- [ktlint 공식 GitHub](https://github.com/pinterest/ktlint)
- [ktlint Gradle 플러그인](https://github.com/JLLeitschuh/ktlint-gradle)
- [Kotlin 공식 코딩 컨벤션](https://kotlinlang.org/docs/coding-conventions.html)
