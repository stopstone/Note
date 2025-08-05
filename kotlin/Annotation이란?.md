# Annotation이란?

> Kotlin 개발에서 `@JvmStatic`, `@JvmOverloads`, `@Parcelize` 등 다양한 어노테이션을 사용합니다.  
> 어노테이션은 코드에 메타데이터를 추가하여 컴파일 타임 검증, 코드 생성, 런타임 동작 제어 등의 기능을 제공합니다.

## 1. Annotation이란?

`Annotation`은 소스코드에 `@어노테이션명`의 형태로 표현하며 클래스, 필드, 메소드의 선언부에 적용할 수 있는 **특정 기능이 부여된 표현법**을 말합니다.

### 주요 특징

* **메타데이터 제공**: 코드에 추가 정보를 담아 컴파일러나 런타임에 특정 동작을 수행
* **코드 가독성 향상**: 의도를 명확하게 표현하여 코드 이해도 증가
* **컴파일 타임 검증**: 잘못된 사용을 미리 방지
* **코드 생성**: 어노테이션 프로세서를 통해 반복적인 코드 자동 생성

### 언제 사용하는가?

* **컴파일 타임 검증**이 필요한 경우 (`@Override`, `@Nullable`)
* **코드 생성**이 필요한 경우 (`@Parcelize`, `@Keep`)
* **런타임 동작 제어**가 필요한 경우 (`@Keep`, `@SuppressLint`)
* **의존성 주입**을 위한 경우 (`@Inject`, `@Module`)



## 2. 커스텀 Annotation 만들기

### 2.1 기본 어노테이션 정의
```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class DebugLog

@Target(AnnotationTarget.FIELD)
@Retention(AnnotationRetention.SOURCE)
annotation class AutoBind
```

### 2.2 어노테이션 프로세서 활용
```kotlin
@AutoBind
class MainActivity : AppCompatActivity() {
    @AutoBind
    lateinit var textView: TextView
    
    @AutoBind
    lateinit var button: Button
}
```

## 3. Annotation 사용 시 주의사항

### 3.1 성능 고려사항
* **런타임 어노테이션**: 리플렉션을 사용하므로 성능 오버헤드 발생 가능
* **컴파일 타임 어노테이션**: 성능 영향 없음, 권장

### 3.2 적절한 사용
* **과도한 사용 금지**: 코드 복잡성 증가
* **명확한 의도**: 어노테이션의 목적을 명확히 이해하고 사용
* **문서화**: 커스텀 어노테이션은 반드시 문서화

## 4. Annotation vs XML 설정

| 항목 | Annotation | XML 설정 |
|------|------------|----------|
| 타입 안전성 | 높음 (컴파일 타임 검증) | 낮음 (런타임 오류 가능) |
| 가독성 | 높음 (코드와 함께) | 낮음 (분리된 파일) |
| 유지보수 | 쉬움 | 어려움 |
| 성능 | 영향 없음 | 파싱 오버헤드 |
| 사용 예 | 안드로이드 개발 | 레거시 시스템 |
