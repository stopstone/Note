# Kotlin companion object

> Kotlin의 companion object는 Java의 static 키워드를 대체하며, 클래스 수준의 멤버를 정의할 때 사용합니다.  
> 이 글에서는 companion object의 정의, 사용 시점, 예제, 그리고 Java에서의 접근 방식을 정리합니다.  

## 1. companion object란?

Kotlin은 클래스 내부에 `static` 멤버를 직접 선언할 수 없습니다.    
대신 companion object를 통해 클래스에 종속된 싱글턴 객체(singleton object)로서, 정적 멤버처럼 사용할 수 있는 구조를 제공합니다.   
이 객체는 컴파일 시 내부 `static final` 클래스로 생성되며, `클래스명.Companion`을 통해 접근합니다.  

※ companion object는 **객체이며 런타임에 존재**하지만, Java의 static은 **클래스 로딩 시점에 메모리에 존재**한다는 차이가 있습니다.

## 2. 언제 companion object를 사용할까?

### 2-1. 팩토리 메서드 구현

복잡한 객체 생성을 캡슐화하기 위해 companion object에 팩토리 메서드를 정의합니다.

```kotlin
class User private constructor(val name: String) {
    companion object {
        fun create(name: String): User = User(name)
    }
}

val user = User.create("신짱구")
```

### 2-2. 상수 정의

클래스와 관련된 상수를 정의할 때 companion object를 사용하면 좋습니다.

```kotlin
class Constants {
    companion object {
        const val MAX_AGE = 100

        @JvmField
        val DEFAULT_NAME = "신짱구"
    }
}
```

```kotlin
val ageLimit = Constants.MAX_AGE
```

Java에서는 `const val`은 `getMAX_AGE()`로 접근하지만, `@JvmField`가 붙은 프로퍼티는 바로 필드처럼 접근할 수 있습니다.

```java
int maxAge = Constants.Companion.getMAX_AGE(); // const val
String name = Constants.DEFAULT_NAME; // @JvmField 사용 시
```

### 2-3. 인터페이스 구현

companion object는 객체이므로 인터페이스를 구현할 수도 있습니다.

```kotlin
interface Factory<T> {
    fun create(): T
}

class Product {
    companion object : Factory<Product> {
        override fun create(): Product = Product()
    }
}

val product = Product.create()
```

## 3. Java에서 companion object 사용하기

Kotlin의 companion object는 Java 코드에서 다음과 같이 접근할 수 있습니다.

```java
User.Companion.create("신짱구");
```

`@JvmStatic` 어노테이션을 사용하면 Java에서 더 간결하게 호출할 수 있습니다.

```kotlin
class User {
    companion object {
        @JvmStatic
        fun greet() = println("안녕하세요!")
    }
}
```

```java
User.greet();
```

## 4. Kotlin vs Java companion 호출 비교

| 목적              | Kotlin 호출                | Java 호출                            |
| --------------- | ------------------------ | ---------------------------------- |
| 팩토리 메서드 호출      | `User.create("신짱구")`     | `User.Companion.create("신짱구")`     |
| @JvmStatic 사용 시 | `User.greet()`           | `User.greet()`                     |
| 상수 접근           | `Constants.MAX_AGE`      | `Constants.Companion.getMAX_AGE()` |
| @JvmField 접근    | `Constants.DEFAULT_NAME` | `Constants.DEFAULT_NAME`           |

## 참고 자료

* Kotlin 공식 문서: [https://kotlinlang.org/docs/object-declarations.html](https://kotlinlang.org/docs/object-declarations.html)
* Kotlin 공식 문서: [https://kotlinlang.org/docs/classes.html](https://kotlinlang.org/docs/classes.html)
* Kotlin in Action 4장: 클래스, 객체, 인터페이스
* Medium 블로그: [https://medium.com/@appdevinsights/companion-object-in-kotlin-c3a1203cd63c](https://medium.com/@appdevinsights/companion-object-in-kotlin-c3a1203cd63c)
* Medium 블로그: [https://medium.com/softaai-blogs/kotlin-companion-object-explained-the-ultimate-guide-for-beginners-bbb26dcd0273](https://medium.com/softaai-blogs/kotlin-companion-object-explained-the-ultimate-guide-for-beginners-bbb26dcd0273)
