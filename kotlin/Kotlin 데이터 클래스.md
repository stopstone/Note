# Kotlin Data Class

> Kotlin의 `data class`는 데이터를 담는 데 최적화된 클래스입니다.  
> 일반 클래스와의 차이점, 사용 시점, 그리고 실무에서의 활용 예를 중심으로 정리해보겠습니다.    

---

## 1. 언제 사용하는가?

`data class`는 다음과 같은 상황에서 유용하게 사용할 수 있습니다:

- 사용자 정보, 설정 값 등 **순수 데이터를 보관하는 용도**  
- 객체의 상태가 불변이어야 하는 **불변 객체**  
- 객체의 참조가 아닌 **값으로 동등성 비교가 필요한 경우**  
- 기존 객체를 기반으로 일부 속성만 변경한 **복사 객체를 생성할 때**

예시:

```kotlin  
data class User(val name: String, val age: Int)  
```

이렇게 정의하면 `equals()`, `hashCode()`, `toString()`, `copy()` 등의 메서드가 자동 생성됩니다.

---

## 2. 일반 클래스와의 차이점

| 항목                   | 일반 클래스 (`class`) | 데이터 클래스 (`data class`) |  
| -------------------- | ---------------- | ---------------------- |  
| equals(), hashCode() | 수동 구현 필요         | 자동 생성                  |  
| toString()           | 기본 구현 제공         | 속성 정보를 포함한 문자열 반환      |  
| copy()               | 수동 구현 필요         | 자동 생성                  |  
| componentN()         | 없음               | 자동 생성                  |  
| 주 사용 목적              | 로직 및 상태 관리       | 데이터 보관 및 전달            |

즉, `data class`는 데이터를 표현하고 전달하는 데에 특화된 설계라고 할 수 있습니다.

### 2-1. 일반 클래스에서 equals(), hashCode(), toString() 오버라이드 예시

```kotlin  
class User(val name: String, val age: Int) {  
    override fun equals(other: Any?): Boolean {  
        if (this === other) return true  
        if (other !is User) return false

        return name == other.name && age == other.age  
    }

    override fun hashCode(): Int {  
        var result = name.hashCode()  
        result = 31 * result + age  
        return result  
    }

    override fun toString(): String {  
        return "User(name=$name, age=$age)"  
    }  
}  
```

---

## 3. 사용 시 주의할 점

- 반드시 하나 이상의 `val` 또는 `var` 속성이 주 생성자에 선언되어야 합니다.  
- `data class`는 `abstract`, `open`, `sealed`, `inner`로 선언할 수 없습니다.  
- 클래스 본문에 정의된 프로퍼티는 자동 생성되는 함수(`equals()`, `hashCode()` 등)에 포함되지 않습니다.

---

## 참고 자료

- [공식 문서 - Data Classes](https://kotlinlang.org/docs/data-classes.html)  
- 『Kotlin in Action』 Chapter 4. 클래스, 객체, 인터페이스

