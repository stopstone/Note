# Kotlin Nothing 타입

## 1. Nothing 타입이란?

> Kotlin에서 `Nothing`은 "절대 값이 존재할 수 없는 타입"을 의미합니다. 프로그램의 흐름이 이 지점에서 **끝나야 함을 나타내는 특수한 타입**입니다.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

이 함수는 항상 예외를 던지기 때문에 **정상적인 값은 절대 반환하지 않습니다**.  
따라서 반환 타입으로 `Nothing`을 사용합니다.

---

## 2. 언제 사용하나요?

### 2.1 예외를 던지는 함수

* 예외를 발생시켜 더 이상 코드를 진행하지 않는 경우

```kotlin
fun requirePositive(x: Int): Int {
    if (x <= 0) error("양수만 허용됩니다")
    return x
}
```

이때 `error()` 함수는 내부적으로 `Nothing`을 반환하는 함수이므로 `requirePositive` 함수는 Int를 그대로 반환할 수 있습니다.

### 2.2 무한 루프

* `while(true)`와 같은 무한 루프 함수에도 적용할 수 있습니다.

```kotlin
fun loopForever(): Nothing {
    while (true) {}
}
```

### 2.3 스마트 캐스트 보조

* 조건문 분기에서 `Nothing`을 반환하는 함수가 있으면, 이후 코드는 컴파일러가 "도달 불가"로 판단하여 스마트 캐스트가 가능해집니다.

```kotlin
fun checkNotNull(value: String?) {
    if (value == null) error("null 불가")
    println(value.length) // value는 String으로 스마트 캐스트됨
}
```

---

## 3. 자바와의 차이점은?

| 항목           | Kotlin (`Nothing`)       | Java (`void`, `Exception`) |
| ------------ | ------------------------ | -------------------------- |
| 반환 타입 없음 표현  | `Nothing`                | `void`                     |
| 예외만 던지는 함수   | `Nothing` 반환 타입 사용       | `void` 반환, 주석으로 의미 표현      |
| 타입 시스템 통합 여부 | 타입 시스템의 일부 (모든 타입의 서브타입) | 타입 아님 (primitive)          |
| 표현력          | 강력한 타입 추론 및 스마트 캐스트      | 수동 예외 처리 필요                |

Kotlin에서는 `Nothing`이 **모든 타입의 하위 타입**이기 때문에, 어떤 타입 자리에도 들어갈 수 있습니다.  
반면 자바에서는 void는 타입이 아니며, 예외를 던지는 함수와의 타입 추론 연계는 불가능합니다.

---

## 4. Unit과의 차이

| 항목       | `Nothing`              | `Unit`                            |
| -------- | ---------------------- | --------------------------------- |
| 의미       | 절대 값을 반환하지 않음          | 아무 값도 의미하지 않는 단일 객체 반환            |
| 반환 가능 여부 | 절대 반환되지 않음             | 항상 반환됨 (예: 함수의 기본 반환형)            |
| 사용 예     | 예외 던짐, 무한 루프, 도달 불가 코드 | 반환 값이 불필요한 함수 (`fun log(): Unit`) |
| 타입 역할    | 모든 타입의 하위 타입           | 단일 인스턴스 타입 (싱글톤)                  |

즉, `Nothing`은 "여기서 절대 돌아오지 않아!"를 의미하고, `Unit`은 "돌아오긴 하지만 특별한 값은 없어요"라고 볼 수 있습니다.

---

## 5. 왜 쓰는 걸까?

* **명확한 의도 표현**: 이 함수는 절대 값을 반환하지 않는다는 의도 전달
* **스마트 캐스트 활용**: 컴파일러가 더 정확하게 타입 추론 가능
* **에러 핸들링 일관성**: 실패 시 예외를 던지는 함수들의 타입을 통일시켜 가독성과 유지보수 향상

---

## 참고자료

* [Kotlin 공식 Nothing 문서](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-nothing/)
* [Effective Kotlin - Item 44: Use Nothing to mark unreachable code]
* [Kotlin 공식 Unit 문서](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-unit/)
