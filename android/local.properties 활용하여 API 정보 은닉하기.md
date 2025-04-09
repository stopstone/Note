# local.properties 활용하여 API 정보 은닉하기

> API 키 같은 민감 정보는 절대 하드코딩하지 말고, 항상 안전하게 관리해야 합니다.   
> 팀 프로젝트나 오픈소스 활동을 할 때 이런 기본적인 보안 수칙을 지키는 것이 개발자의 기본 소양입니다. (개인프로젝트에서도 필요합니다.)    
> 하지만, `github`이나 `블로그` 등에서 유출하는 경우도.. 가끔 있습니다.  
> API키를 노출하면 보안, API 과다 사용으로 비용이 발생할 수 있습니다.
> 이 문서에서 설명할 방법을 적용하여, 보다 안전하게 KEY를 관리하는 것을 목표로 합니다.  

> 따라서, local.properties 파일을 활용해서 민감한 정보를 안전하게 관리하고,  
> 코드에서는 이를 불러와 사용하는 방법을 소개하겠습니다.  

## 1. 왜 local.properties를 사용하는가?
- local.properties 파일은 로컬 개발 환경에만 존재하고, 보통 .gitignore에 기본으로 포함되어 있어 깃에 커밋되지 않습니다.
- API 키를 소스 코드에서 분리하면, 오픈소스 프로젝트를 운영할 때도 민감 정보 유출 위험을 줄일 수 있습니다.
- 빌드 시점에 필요한 값만 주입하기 때문에, 코드 관리가 더 깔끔해집니다.

## 2. 코드로 알아보기

### 2.1 local.properties 파일 작성
프로젝트 루트(local.properties)에 다음과 같이 민감 정보를 추가합니다.

```
APP_KEY=API키를 입력해주세요.
```

### 2.2 build.gradle.kts 수정
앱 모듈(app/build.gradle.kts)에 다음 코드를 추가하여 local.properties 파일을 읽어옵니다.

manifestPlaceholders는 메니페스트에 API키를 활용하기 위해 사용합니다.  
ex) GCP, KAKAO, NAVER

```kotlin
import java.util.Properties

val localPropertiesFile = project.rootProject.file("local.properties")
val properties = Properties()

if (localPropertiesFile.exists()) {
    properties.load(localPropertiesFile.inputStream())
} else {
    throw GradleException("local.properties 파일이 없습니다. 민감 정보를 추가해 주세요.")
}

android {
    ...

    defaultConfig {
        ...

        // Manifest에 값을 주입할 때 사용
        manifestPlaceholders += mapOf(
            "APP_KEY" to properties.getProperty("APP_KEY")
        )

        // Kotlin 코드에서 직접 사용할 수 있도록 BuildConfig에 추가
        buildConfigField(
            "String",
            "APP_KEY",
            "\"${properties.getProperty("APP_KEY")}\""
        )
    }

    buildFeatures {
        compose = true
        buildConfig = true // buildConfig로 키를 가져오기 위해 등록
    }
}
```

### 2.3 AndroidManifest.xml에서 사용하는 방법
Manifest 파일에서는 아래처럼 사용할 수 있습니다.

이는, API에 따라 상이할 수 있으므로 활용하는 API의 공식문서를 참조하는 것이 좋습니다.

```xml
<meta-data
    android:name="com.example.APP_KEY"
    android:value="${APP_KEY}" />
```

> ${} 안에 manifestPlaceholders에서 매핑한 키를 그대로 사용합니다.

### 2.4 Kotlin 코드에서 사용하는 방법
코틀린 코드에서는 BuildConfig 클래스를 통해 값을 불러올 수 있습니다.

```kotlin
val appKey = BuildConfig.APP_KEY
```

## 4. 주의사항 및 개선 아이디어
- local.properties가 없는 환경(예: CI 서버)에서 빌드 실패할 수 있으므로, 필요하면 기본값을 설정하는 것도 고려해야 합니다.
- local.properties 대신 gradle.properties를 사용하는 방법도 있는데, 이는 모듈 간 공유가 필요할 때 유용합니다.
