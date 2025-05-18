# Kotlin의 주요 컬렉션 타입 정리

> Kotlin에서는 데이터를 효율적으로 다루기 위해 다양한 컬렉션 타입을 제공합니다.  
> 본 문서에서는 `Array`, `List`, `Set`, `Map`의 정의, 특성, 사용 방법을 정리하고, 각 타입의 차이점까지 비교합니다.  
> 특히 이 네 가지 컬렉션은 **코틀린 면접에서도 자주 출제되는 핵심 주제**이므로, 개념과 사용법을 정확히 이해하는 것이 중요합니다.  

## 1. Array

### 1-1. 정의

* 고정된 크기의 배열입니다.
* 생성 시 크기를 지정해야 하며, 이후 크기 변경은 불가능합니다.

### 1-2. 특징

* 인덱스를 사용해 요소에 접근합니다.
* 값은 변경 가능하지만, 구조(크기)는 변경할 수 없습니다.

### 1-3. 예제

```kotlin
val numbers = arrayOf(1, 2, 3)
numbers[0] = 10
println(numbers.joinToString()) // 10, 2, 3
```

### 1-4. 주의사항

* `==`는 참조 비교이므로 배열 내용 비교는 `contentEquals()` 사용

```kotlin
val a = arrayOf(1, 2)
val b = arrayOf(1, 2)
println(a == b) // false
println(a.contentEquals(b)) // true
```

## 2. List

### 2-1. 정의

* 순서를 유지하는 컬렉션이며, 중복 요소를 허용합니다.
* `List`는 읽기 전용, `MutableList`는 변경 가능합니다.

### 2-2. 예제

```kotlin
val readOnly = listOf("a", "b")
val mutable = mutableListOf(1, 2)
mutable.add(3)
println(mutable) // [1, 2, 3]
```

## 3. Set

### 3-1. 정의

* 중복을 허용하지 않는 컬렉션입니다.
* 기본적으로 순서를 보장하지 않으며, `LinkedHashSet`을 사용하면 삽입 순서를 유지할 수 있습니다.

### 3-2. 예제

```kotlin
val readOnly = setOf(1, 2, 2, 3)
println(readOnly) // [1, 2, 3]

val mutable = mutableSetOf("a", "b")
mutable.add("c")
println(mutable) // [a, b, c]
```

## 4. Map

### 4-1. 정의

* 키-값 쌍의 컬렉션입니다.
* 각 키는 고유하며, 값을 빠르게 검색할 수 있습니다.

### 4-2. 예제

```kotlin
val readOnly = mapOf("a" to 1, "b" to 2)
println(readOnly["a"]) // 1

val mutable = mutableMapOf("x" to 10)
mutable["y"] = 20
println(mutable) // {x=10, y=20}
```

## 5. 요약 비교

| 타입    | 순서 유지 | 중복 허용    | 크기 변경 가능      | 키-값 구조 |
| ----- | ----- | -------- | ------------- | ------ |
| Array | O     | O        | X             | X      |
| List  | O     | O        | O (`Mutable`) | X      |
| Set   | X     | X        | O (`Mutable`) | X      |
| Map   | X     | 키 X, 값 O | O (`Mutable`) | O      |
