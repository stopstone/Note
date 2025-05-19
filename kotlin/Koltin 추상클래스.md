# Kotlin 추상 클래스

> Kotlin의 추상 클래스에 대해 개념을 명확히 정리하고, Java의 추상 클래스와 어떤 차이가 있는지 함께 비교해보겠습니다.  
> 또한 interface와의 차이점도 함께 알아보며, 어떤 상황에서 어떤 방식을 사용하는 것이 적절한지 알아봅니다.  

## 1. 추상 클래스란?

추상 클래스는 **자체적으로 인스턴스화될 수 없는 클래스**로,  
공통 기능을 정의하고 일부 구현을 자식 클래스에 위임할 때 사용합니다.

추상 클래스는 다음과 같은 특징을 가집니다:

* `abstract` 키워드를 사용해 선언
* **추상 함수**(body가 없는 함수)를 포함 가능
* **구현된 함수나 프로퍼티도 가질 수 있음**
* **생성자 가질 수 있음**
* 상속을 위해 `open`이 아닌 `abstract`를 사용

```kotlin
abstract class Animal(val name: String) {
    abstract fun makeSound()
    fun printName() {
        println("My name is $name")
    }
}

class Dog(name: String) : Animal(name) {
    override fun makeSound() {
        println("Woof")
    }
}
```

## 2. 언제 사용하나요?

다음과 같은 경우 추상 클래스를 사용하는 것이 적절합니다:

* **기본 동작을 정의하고 일부만 하위 클래스에서 구현하고 싶을 때**
* **상태(프로퍼티)를 공통적으로 갖고 있어야 할 때**
* **생성자 로직이 필요한 경우**

## 3. Kotlin vs Java 추상 클래스 차이점

| 항목     | Kotlin 추상 클래스            | Java 추상 클래스           |
| ------ | ------------------------ | --------------------- |
| 생성자 선언 | `constructor` 구문 가능      | 일반 생성자 사용             |
| 프로퍼티   | 선언 시 `val`/`var` 키워드 사용  | 필드 + getter/setter 방식 |
| 접근 제한자 | 프로퍼티, 생성자에 다양한 접근 제한자 사용 | 메서드/필드 접근제한자 중심       |
| 기본 구현  | 함수/프로퍼티에 기본 구현 제공 가능     | 함수에만 기본 구현 가능         |

## 4. interface와의 차이점

| 구분     | 추상 클래스           | 인터페이스          |
| ------ | ---------------- | -------------- |
| 키워드    | `abstract class` | `interface`    |
| 다중 상속  | 불가능              | 가능             |
| 생성자    | 있음               | 없음 (구현 불가)     |
| 상태 보유  | O (프로퍼티 가짐)      | X (state 없음)   |
| 기본 구현  | O                | O (default 사용) |
| 접근 제한자 | 가능               | 기본적으로 `public` |
| 용도     | 공통 상태와 기능 전달     | 기능 계약 정의       |

## 5. 잘못된 예제 (SOLID 위반)

다음은 클릭 처리에서 모든 동작을 하나의 추상 클래스에 몰아넣은 잘못된 예시입니다.

```kotlin
abstract class ClickHandler {
    abstract fun onClick()
    abstract fun onLongClick()
}

class ButtonHandler : ClickHandler() {
    override fun onClick() {
        println("Click")
    }

    override fun onLongClick() {
        // 필요 없는 기능도 구현 강제됨
        println("No-op")
    }
}
```

**문제점:** ButtonHandler는 `onLongClick()`을 사용하지 않지만, 추상 클래스라 반드시 구현해야 합니다.  
이는 **인터페이스 분리 원칙(ISP)** 위반입니다.

## 6. 개선된 설계 (인터페이스로 분리)

```kotlin
interface Clickable {
    fun onClick()
}

interface LongClickable {
    fun onLongClick()
}

class ButtonHandler : Clickable {
    override fun onClick() {
        println("Click")
    }
}
```

## 참고 자료

* [Kotlin Language Reference - Abstract Classes](https://kotlinlang.org/docs/inheritance.html#abstract-classes)