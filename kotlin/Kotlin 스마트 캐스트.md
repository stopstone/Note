# Kotlin 스마트 캐스트

> Kotlin의 스마트 캐스트(Smart Cast)는 타입 안정성을 높이고 코드의 가독성을 향상시키기 위한 기능입니다.  
> 명시적 캐스팅 없이도 조건문 내에서 안전하다고 판단된 경우, 컴파일러가 자동으로 타입을 변환해줍니다.  
> 본 문서에서는 스마트 캐스트의 개념과 동작 방식, 제한 사항, 그리고 관련된 `is`와 `as` 키워드의 차이점을 알아보는 것을 목표로합니다.  

## 1. 스마트 캐스트(Smart Cast)란?

스마트 캐스트는 `if`, `when` 등의 조건문을 통해 특정 타입으로 확인된 변수를 컴파일러가 자동으로 그 타입으로 간주하는 기능입니다.  
명시적 캐스팅 없이도 해당 타입의 속성이나 메서드를 사용할 수 있습니다.  

### 1-1. 사용 예시

```kotlin
fun printLength(obj: Any) {
    if (obj is String) {
        println(obj.length) // obj는 String으로 스마트 캐스트됨
    }
}
```

### 1-2. 사용 조건

* `is`, `!is` 연산자 사용 시
* `null` 체크 후
* `when` 표현식 사용 시
* 논리 연산자와 함께 사용 시

### 1-3. 제한 사항

* `var` 변수는 값이 변경될 수 있기 때문에 스마트 캐스트가 적용되지 않음
* 커스텀 getter가 있는 프로퍼티는 컴파일러가 값을 예측할 수 없어서 적용 불가
* 멀티스레드 환경에서는 스마트 캐스트가 적용되지 않을 수 있음

## 2. Kotlin 2.0의 스마트 캐스트 향상

* `||` 조건에서도 공통 상위 타입으로 캐스트 가능
* `val` 변수는 조건문 외부에서도 스마트 캐스트 정보 유지 가능
* 인라인 함수 내부에서도 스마트 캐스트 동작

## 3. `is`와 `as`의 차이점

### 3-1. `is` 연산자

* 객체가 특정 타입인지 검사할 때 사용
* 조건문 내에서는 스마트 캐스트가 자동으로 적용됨

```kotlin
val obj: Any = "Hello"
if (obj is String) {
    println(obj.length) // 스마트 캐스트
}
```

### 3-2. `as` 연산자

* 객체를 명시적으로 특정 타입으로 캐스팅
* 실패 시 `ClassCastException` 발생

```kotlin
val obj: Any = "Hello"
val str: String = obj as String
```

### 3-3. `as?` 연산자

* 안전한 캐스팅
* 실패 시 null을 반환

```kotlin
val obj: Any = 123
val str: String? = obj as? String // null 반환
```

### 3-4. 비교 정리

| 연산자   | 용도      | 캐스팅 방식 | 실패 시 처리                      |
| ----- | ------- | ------ | ---------------------------- |
| `is`  | 타입 검사   | 자동     | 스마트 캐스트 적용                   |
| `as`  | 명시적 캐스팅 | 수동     | 예외 발생 (`ClassCastException`) |
| `as?` | 안전한 캐스팅 | 수동     | `null` 반환                    |

## 참고 자료

Kotlin 공식 문서: [Smart Cast](https://kotlinlang.org/docs/typecasts.html#smart-casts)  
Kotlin in Action: 3장 - 클래스와 객체 다루기
