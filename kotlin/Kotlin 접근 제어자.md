# Kotlin 접근 제어자

> Kotlin에서도 Java처럼 접근 제어자를 통해 클래스, 함수, 프로퍼티의 접근 범위를 제어할 수 있습니다.  
> 실무에서 코드의 모듈화와 캡슐화를 고려할 때 접근 제어자는 중요한 요소이며,  
> 이 글에서는 Kotlin의 접근 제어자 종류와 특징, 기본값, 그리고 Java와의 차이를 함께 정리합니다.  

## 1. Kotlin 접근 제어자 종류

Kotlin에서 제공하는 접근 제어자는 총 4가지입니다.

| 접근 제어자      | 클래스 외부 접근 | 같은 모듈 | 같은 패키지       | 서브클래스 접근 |
| ----------- | --------- | ----- | ------------ | -------- |
| `public`    | O         | O     | O            | O        |
| `internal`  | X         | O     | O            | O        |
| `protected` | X         | X     | O (서브클래스에서만) | O        |
| `private`   | X         | X     | X            | X        |

### 1-1. public

* 어디에서든 접근 가능 (기본값)
* Java의 public과 동일

### 1-2. internal

* 같은 모듈 내에서만 접근 가능
* Java에는 없는 Kotlin 고유의 제어자

### 1-3. protected

* 선언된 클래스 또는 서브클래스에서만 접근 가능
* Kotlin에서는 파일 수준에서는 사용할 수 없음 (클래스 멤버 전용)

### 1-4. private

* 같은 파일 또는 클래스 내에서만 접근 가능
* top-level 선언의 경우 파일 내에서만 유효함

## 2. Kotlin의 기본 접근 제어자

Kotlin은 다음과 같은 기본 접근 제어자를 가집니다:

* 클래스나 함수, 프로퍼티를 선언할 때 접근 제어자를 생략하면 `public`이 기본
* top-level 함수/프로퍼티의 경우, `public`이 기본값

예시:

```kotlin
fun greet() {
    println("Hello")
}
// 위 함수는 public fun greet() 와 동일함
```

## 3. Java와의 차이점

| 구분        | Java              | Kotlin        |
| --------- | ----------------- | ------------- |
| 기본값       | default (package) | public        |
| internal  | 없음                | 있음            |
| protected | 패키지 + 상속          | 상속만 (패키지 무관)  |
| top-level | 클래스 안에서만 정의 가능    | 파일 최상단에 선언 가능 |

### 핵심 차이 요약

* **internal**은 Kotlin만의 키워드로, Java로 컴파일 시 public으로 변환됨
* **protected**는 Kotlin에서는 클래스/서브클래스 전용으로, 같은 패키지에 있어도 외부 클래스는 접근 불가
* Kotlin은 top-level에 함수나 프로퍼티 선언이 가능하며, Java와 달리 객체 수준이 아닌 파일 단위의 캡슐화가 가능함

## 참고자료

* [Kotlin 공식 문서 - Visibility Modifiers](https://kotlinlang.org/docs/visibility-modifiers.html)
* [Baeldung - Kotlin Visibility](https://www.baeldung.com/kotlin/visibility-modifiers)
