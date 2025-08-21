# String equals()와 == 연산자의 차이

> Java에서 문자열 비교는 단순해 보이지만, String Pool이라는 특별한 메모리 관리 방식 때문에 **equals()와 == 연산자**가 서로 다른 결과를 보여줍니다.  
> 이 글에서는 자바의 equals()에 대해 알아보고, 코틀린과의 차이도 살짝 정리해보겠습니다.  

---

## 1. equals()와 == 연산자의 차이점

### equals() 메서드
```java
// 문자열 내용 자체를 비교
String s1 = "신짱구";
String s2 = "신짱구";
s1.equals(s2);  // true - 내용이 같음
```

### == 연산자
```java
// 객체의 참조(주소값)를 비교
String s1 = "신짱구";
String s2 = new String("신짱구");
s1 == s2;  // false - 서로 다른 객체를 참조
```

---

## 2. String Pool이란?

### String Pool의 개념
```java
// String 리터럴로 생성 - String Pool에 저장
String s1 = "떡잎마을";
String s2 = "떡잎마을";

// 같은 문자열이므로 같은 객체를 참조
System.out.println(s1 == s2);  // true
```

String Pool은 **JVM Heap 메모리 내의 특별한 영역**으로, 동일한 문자열 리터럴을 재사용하여 메모리를 절약합니다.

### new 키워드 사용 시
```java
// new 키워드로 생성 - Heap에 새로운 객체 생성
String s1 = "액션가면";
String s2 = new String("액션가면");

System.out.println(s1 == s2);       // false - 다른 객체 참조
System.out.println(s1.equals(s2));  // true - 내용은 같음
```

---

## 3. 실제 예시로 이해하기

### 예시 1: 기본적인 차이점
```java
String s1 = "신짱구";
String s2 = new String("신짱구");

// == 연산자 비교
System.out.println(s1 == s2);       // false
System.out.println("이유: 서로 다른 메모리 위치를 참조");

// equals() 메서드 비교
System.out.println(s1.equals(s2));  // true
System.out.println("이유: 문자열 내용이 동일");
```

### 예시 2: String Pool 동작 확인
```java
String pool1 = "흰둥이";
String pool2 = "흰둥이";
String heap1 = new String("흰둥이");

System.out.println("String Pool 비교:");
System.out.println(pool1 == pool2);      // true - 같은 객체 참조

System.out.println("Pool vs Heap 비교:");
System.out.println(pool1 == heap1);      // false - 다른 객체 참조
System.out.println(pool1.equals(heap1)); // true - 내용은 같음
```

---

## 4. intern() 메서드 활용

### intern() 메서드란?
```java
String s1 = "짱아";
String s2 = new String("짱아");
String s3 = s2.intern();  // String Pool에 등록

System.out.println(s1 == s2);  // false
System.out.println(s1 == s3);  // true - intern()으로 String Pool 참조
```

### intern() 동작 원리
```java
// intern() 메서드는 String Pool을 확인하여:
// 1. 동일한 문자열이 있으면 해당 참조 반환
// 2. 없으면 String Pool에 추가 후 참조 반환

String name = new String("신형만");
String internedName = name.intern();

String literalName = "신형만";
System.out.println(literalName == internedName);  // true
```

---

## 5. Kotlin에서는 어떻게 다를까?

### Kotlin의 문자열 비교
```kotlin
// Kotlin에서는 == 연산자가 자동으로 equals() 호출
val s1 = "신짱구"
val s2 = "신짱구"
val s3 = String("신짱구".toCharArray())  // 새로운 객체 생성

println(s1 == s2)   // true (내용 비교)
println(s1 === s2)  // true (참조 비교, String Pool 때문)
println(s1 === s3)  // false (다른 참조)
println(s1 == s3)   // true (내용 비교)
```

### Java vs Kotlin 차이점
```kotlin
// Java 방식 (Kotlin에서도 가능)
if ("신짱구".equals(userName)) {
    // 안전한 비교
}

// Kotlin 방식 (더 간단)
if (userName == "신짱구") {
    // Kotlin에서는 이것도 안전 (자동으로 equals() 호출)
}

// 참조 비교가 필요한 경우
if (userName === "신짱구") {
    // 객체 참조 비교
}
```

Kotlin에서는 `==` 연산자가 내부적으로 `equals()`를 호출하므로 Java보다 더 직관적으로 문자열 비교가 가능합니다.
