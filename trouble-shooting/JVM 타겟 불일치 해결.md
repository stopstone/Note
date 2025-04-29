# [트러블슈팅] JVM-target compatibility 오류 해결

> Android Studio 프로젝트를 클론하거나, 이전에 작업하던 프로젝트를 다시 실행할 때 다음과 같은 JVM-target 관련 에러를 마주칠 수 있습니다.
> 이 문서에서는 "Inconsistent JVM-target compatibility" 오류가 발생한 이유와 해결 방법을 트러블슈팅 형식으로 정리합니다.

## 1. 문제 상황

Android 프로젝트 빌드 시 다음과 같은 에러가 발생했습니다.

```
Execution failed for task ':core:navigation:compileKotlin'.
> Inconsistent JVM-target compatibility detected for tasks 'compileJava' (17)와 'compileKotlin' (21).
```

```
Execution failed for task ':domain:kspKotlin'.
> Inconsistent JVM-target compatibility detected for tasks 'compileJava' (17)와 'kspKotlin' (21).
```

※ 에러에 등장하는 모듈명(:core:navigation, :domain)이나 작업명(compileKotlin, kspKotlin)은 프로젝트에 따라 다를 수 있습니다.

## 2. 발생 원인

- Java 컴파일러(`compileJava`)는 JVM 17을 사용하고 있었고,
- Kotlin 컴파일러(`compileKotlin`, `kspKotlin` 등)는 JVM 21을 사용하고 있었습니다.

Android Gradle Plugin(AGP)이나 Kotlin 버전을 업그레이드하면 기본 JVM Target이 변경될 수 있습니다.
- AGP 8.0+, Kotlin 2.0+에서는 JVM 21 이상을 요구합니다.

Gradle 빌드 시 컴파일러 간 JVM 버전이 일치하지 않으면, 호환성 문제를 방지하기 위해 빌드가 중단됩니다.

## 4. 해결 방법

### 4.1. Gradle JDK 버전 수정

**Android Studio > File > Project Structure > SDK Location** 메뉴로 이동하여 Gradle JDK를 **21**로 변경합니다.

[여기서 Gradle JDK를 21로 설정하는 화면을 보여준다] 말한다

변경 후 반드시 "Apply"와 "OK"를 눌러야 적용됩니다.

### 4.2. build.gradle 파일에 JVM Target 명시 (선택사항)

추가로, build.gradle 또는 build.gradle.kts 파일에 명시적으로 JVM Target을 설정해주는 방법도 있습니다.

```kotlin
kotlin {
    jvmToolchain(21)
}
```

또는,

```kotlin
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
    kotlinOptions {
        jvmTarget = "21"
    }
}
```

`jvmToolchain(21)` 방식을 사용하는 것이 더욱 권장됩니다.

## 5. 최종 상태 확인

- Gradle JDK: 21
- compileJava JVM Target: 21
- Kotlin 관련 작업 JVM Target: 21

빌드가 정상적으로 완료됩니다.

[빌드 성공 후 Android Studio Build 창 화면을 보여준다] 말한다

