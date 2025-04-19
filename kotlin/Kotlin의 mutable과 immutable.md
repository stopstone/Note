# Kotlin의 mutable과 immutable

> 코틀린은 컬렉션을 `mutable`과 `immutable`로 구분하여, 코드의 안정성과 예측 가능성을 높입니다.  
> 기본적으로 `immutable` 컬렉션을 권장하며,  
> 이는 함수형 프로그래밍과의 조화, 스레드 안전성, API 설계의 명확성 등 여러 측면에서 이점을 제공합니다.  

---

## 1. `mutable`과 `immutable` 컬렉션의 차이점

코틀린에서 컬렉션은 크게 두 가지로 나뉩니다:

- **`immutable` 컬렉션**: 한 번 생성되면 변경할 수 없는 컬렉션입니다.  
예를 들어, `listOf()`를 사용하여 생성한 리스트는 요소를 추가하거나 제거할 수 없습니다.

  ```kotlin
  val fruits = listOf("Apple", "Banana", "Cherry")
  // fruits.add("Durian") // 오류 발생: add 함수는 사용 불가
  ```

- **`mutable` 컬렉션**: 요소를 추가, 수정, 삭제할 수 있는 컬렉션입니다.  
예를 들어, `mutableListOf()`를 사용하여 생성한 리스트는 자유롭게 변경할 수 있습니다.

  ```kotlin
  val fruits = mutableListOf("Apple", "Banana", "Cherry")
  fruits.add("Durian") // 정상 작동
  ```

이러한 구분은 개발자가 컬렉션의 사용 목적에 따라 적절한 타입을 선택하도록 유도합니다.

---

## 2. 왜 기본적으로 `immutable` 컬렉션을 권장할까요?

코틀린이 `immutable` 컬렉션을 기본으로 권장하는 이유는 다음과 같습니다:

1. **안정성과 예측 가능성**: `immutable` 컬렉션은 상태 변경이 없기 때문에, 코드의 동작을 예측하기 쉬워집니다.  
이는 디버깅과 유지보수를 용이하게 합니다.

2. **스레드 안전성**: `immutable` 컬렉션은 여러 스레드에서 동시에 접근하더라도 상태 변경이 없으므로,  
동기화에 대한 걱정 없이 안전하게 사용할 수 있습니다.

3. **함수형 프로그래밍과의 조화**: 코틀린은 함수형 프로그래밍 패러다임을 지원합니다.  
`immutable` 컬렉션은 이러한 패러다임과 잘 맞아떨어지며,  
`map`, `filter`, `fold` 등의 함수를 활용한 선언적 프로그래밍을 가능하게 합니다.

4. **API 설계의 명확성**: 외부에 `immutable` 컬렉션을 제공함으로써, 내부 구현을 보호하고,  
외부에서의 불필요한 상태 변경을 방지할 수 있습니다. 이는 캡슐화와 모듈화에 기여합니다.

---

## 3. `immutable` 컬렉션의 활용 사례

`immutable` 컬렉션은 다음과 같은 상황에서 유용하게 사용됩니다:

- **상태 공유**: 여러 컴포넌트나 스레드 간에 상태를 공유할 때, `immutable` 컬렉션을 사용하면 예기치 않은 변경을 방지할 수 있습니다.

- **함수형 프로그래밍**: `map`, `filter`, `reduce` 등의 함수를 활용하여 데이터를 변형할 때,  
원본 데이터를 변경하지 않고 새로운 컬렉션을 생성할 수 있습니다.

  ```kotlin
  val numbers = listOf(1, 2, 3)
  val doubled = numbers.map { it * 2 } // [2, 4, 6]
  ```

- **API 설계**: 외부에 데이터를 제공할 때, `immutable` 컬렉션을 반환함으로써, 외부에서의 불필요한 상태 변경을 방지할 수 있습니다.

  ```kotlin
  private val _items = mutableListOf<String>()
  val items: List<String> get() = _items // 외부에는 불변 리스트 제공
  ```

- **내부적으로 `mutable`을 활용하는 불변 구조 만들기**:
  `toMutableList()`나 `copy()` 등을 활용하면 외부에는 불변처럼 보이지만, 내부에서는 필요한 로직에 따라 유연하게 조작할 수 있습니다.

  ```kotlin
  val original = listOf("A", "B")
  val modified = original.toMutableList().apply { add("C") }.toList()
  println(modified) // [A, B, C]
  ```

  이처럼 원본 데이터를 보존하면서 새로운 리스트를 구성할 수 있어, 안정성과 유연성을 동시에 확보할 수 있습니다.

---

## 4. 참고 자료

- [Kotlin 공식 문서: 컬렉션 개요](https://kotlinlang.org/docs/collections-overview.html)
