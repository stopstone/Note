# equals()와 hashCode()는 왜 항상 같이 override 해야 할까?

> Java와 Kotlin에서 객체의 동등성 비교는 단순한 문법이 아니라, 컬렉션의 동작과 깊게 연관된 중요한 개념입니다.  
> 특히 `HashMap`, `HashSet` 같은 해시 기반 자료구조를 사용할 때는 `equals()`와 `hashCode()`를 반드시 함께 override 해야 예기치 않은 오류를 막을 수 있습니다.  

---

## 1. equals()와 hashCode()는 어떤 역할을 할까요?

### equals()

* 두 객체가 **논리적으로 같은지** 비교하는 메서드입니다.
* 기본 구현은 참조 주소 비교(`==`)로 동작합니다.

### hashCode()

* 객체를 해시 기반 자료구조에 넣기 위한 **정수값**을 반환합니다.
* 동일 객체라면 항상 같은 해시값을 반환해야 합니다.

> 이 둘은 독립적인 것처럼 보이지만, 해시 기반 컬렉션에서는 반드시 **같이** 사용됩니다.

---

## 2. 왜 항상 같이 override 해야 할까요?

### HashMap, HashSet 내부 구조

1. `hashCode()`를 통해 **버킷(bucket)** 위치를 결정합니다.
2. 같은 버킷 안에서 `equals()`로 실제 동일 객체인지 확인합니다.

```kotlin
val map = HashMap<Person, String>()
map[Person("신짱구", 1)] = "Android Developer"
println(map[Person("신짱구", 1)]) // null? "Android Developer"?
```

* 위 코드에서 `equals()`만 override하고 `hashCode()`를 override하지 않으면, 같은 논리적 객체라도 **다른 버킷에 저장**되기 때문에 값을 찾을 수 없습니다.
* 반대로 `hashCode()`만 override하면, 같은 해시값을 가진 **다른 객체**와 잘못 비교되는 문제가 생길 수 있습니다.

> 따라서 두 메서드는 **계약(contract)** 에 따라 함께 오버라이드해야 합니다.

---

## 3. Kotlin vs Java 차이점

### Java

* `equals()`와 `hashCode()` 모두 수동으로 override 해야 합니다.
* 실수로 하나만 override할 경우, IDE가 경고해 주는 정도지만 **런타임 오류**가 발생할 가능성이 큽니다.

### Kotlin

* **`data class`는 자동으로 equals(), hashCode()를 구현**해줍니다.
* 클래스의 모든 `property`가 비교/해시 대상이 됩니다.

```kotlin
data class User(val name: String, val id: Int)
```

* 위 클래스는 아래처럼 자동 구현됩니다:

```kotlin
override fun equals(other: Any?): Boolean = ...
override fun hashCode(): Int = ...
```

### 주의할 점

* `data class`가 아닌 일반 클래스에서는 Java처럼 직접 override 해줘야 합니다.
* 특히 `equals()` 구현 시에는 `hashCode()`도 반드시 함께 구현해야 합니다.

---

## 4. mutable 객체는 주의가 필요합니다

* `hashCode()`는 객체의 상태가 바뀌지 않을 것을 전제로 합니다.
* 하지만 필드가 변경 가능한(mutable) 객체는 내부 값이 바뀌면 해시값도 바뀔 수 있습니다.

```kotlin
val user = User("신짱구", 1)
val set = hashSetOf(user)
user.id = 2 // 내부 상태 변경
println(set.contains(user)) // false?!
```

* 이 때문에 컬렉션에서는 **immutable 객체**를 key로 사용하는 것이 좋습니다.

---

## 5. 정리하며

| 구분             | equals()만 구현 | hashCode()만 구현 | 둘 다 구현 |
| -------------- | ------------ | -------------- | ------ |
| HashMap/Set 동작 | O작동 X        | 오작동 가능         | 정상 작동  |

* `equals()`와 `hashCode()`는 반드시 **같이 override** 해야 합니다.
* Kotlin의 `data class`는 이를 자동으로 처리해주어 실수를 줄여줍니다.
* 해시 기반 컬렉션에서는 mutable 객체보다 **immutable 객체**를 사용하는 습관이 중요합니다.

---

## 참고 자료

* [Effective Java - Item 10, 11](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
* [Kotlin 공식 문서: Data classes](https://kotlinlang.org/docs/data-classes.html)
* [Java 공식 문서: Object.equals() / hashCode()](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html)
