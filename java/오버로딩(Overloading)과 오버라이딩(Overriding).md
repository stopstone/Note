# 오버로딩(Overloading)과 오버라이딩(Overriding)

> 객체지향 프로그래밍에서 **다형성(Polymorphism)**을 구현하는 두 가지 핵심 기법입니다.  
> 비슷해 보이지만 완전히 다른 개념으로, 각각의 특징과 사용 시점을 명확히 구분해서 이해해야 합니다.

---

## 1. 오버로딩(Overloading)이란?

**같은 이름의 메서드**를 **매개변수만 다르게** 하여 여러 개 정의하는 것입니다.

### 특징
- 메서드명: **동일**
- 매개변수: **다름** (타입, 개수, 순서)
- 반환타입: **관계없음** (같을 수도, 다를 수도 있음)
- **컴파일 타임**에 어떤 메서드를 호출할지 결정됨

### 예시

```kotlin
class Calculator {
    // 정수 덧셈
    fun add(a: Int, b: Int): Int {
        return a + b
    }
    
    // 실수 덧셈  
    fun add(a: Double, b: Double): Double {
        return a + b
    }
    
    // 세 개 정수 덧셈
    fun add(a: Int, b: Int, c: Int): Int {
        return a + b + c
    }
    
    // 문자열 연결
    fun add(a: String, b: String): String {
        return a + b
    }
}
```

### 생성자 오버로딩

```kotlin
data class User(
    val id: Long,
    val name: String,
    val email: String?,
    val age: Int?
) {
    // 기본 생성자 외에 추가 생성자들
    constructor(id: Long, name: String) : this(id, name, null, null)
    
    constructor(id: Long, name: String, email: String) : this(id, name, email, null)
}

// 사용 예시
val user1 = User(1L, "김개발")
val user2 = User(2L, "박코틀린", "park@example.com")  
val user3 = User(3L, "이안드로이드", "lee@example.com", 25)
```

---

## 2. 오버라이딩(Overriding)이란?

부모 클래스에서 정의된 메서드를 **자식 클래스에서 재정의**하는 것입니다.

### 특징
- 메서드명: **동일**
- 매개변수: **동일** (타입, 개수, 순서)
- 반환타입: **동일** (또는 하위 타입)
- **런타임**에 실제 객체 타입에 따라 어떤 메서드를 호출할지 결정됨
- `open` 키워드로 오버라이드 허용, `override` 키워드로 재정의

### 예시

```kotlin
// 부모 클래스
open class Animal {
    open fun makeSound(): String {
        return "동물이 소리를 냅니다"
    }
    
    open fun move(): String {
        return "동물이 움직입니다"
    }
}

// 자식 클래스들
class Dog : Animal() {
    override fun makeSound(): String {
        return "멍멍!"
    }
    
    override fun move(): String {
        return "개가 뛰어다닙니다"
    }
}

class Cat : Animal() {
    override fun makeSound(): String {
        return "야옹~"
    }
    
    override fun move(): String {
        return "고양이가 조용히 걸어갑니다"
    }
}

// 사용 예시
fun main() {
    val animals: List<Animal> = listOf(Dog(), Cat())
    
    animals.forEach { animal ->
        println(animal.makeSound()) // 런타임에 실제 타입에 따라 결정
        println(animal.move())
    }
}
```

---

## 3. 주요 차이점 비교

| 구분 | 오버로딩(Overloading) | 오버라이딩(Overriding) |
|------|---------------------|----------------------|
| **메서드명** | 동일 | 동일 |
| **매개변수** | 다름 (타입/개수/순서) | 동일 |
| **반환타입** | 관계없음 | 동일 (또는 하위 타입) |
| **결정 시점** | 컴파일 타임 | 런타임 |
| **관계** | 같은 클래스 내 | 상속 관계 |
| **목적** | 편의성, 유연성 | 다형성 구현 |
| **키워드** | 없음 | `open`, `override` |
