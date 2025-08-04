# 스택(Stack)

> 스택은 **후입선출(LIFO: Last In First Out)** 원칙을 따르는 선형 자료구조입니다.  
> 데이터의 삽입과 삭제가 한쪽 끝에서만 이루어지며,  
> 가장 나중에 들어온 데이터가 가장 먼저 나오는 특성을 가집니다.

## 1. 스택이란?

`스택(Stack)`은 **입력과 출력이 한 곳(방향)으로 제한된 선형 자료구조**입니다.  
마치 접시를 쌓아놓은 것처럼, 가장 위에 있는 접시부터 사용하는 방식과 동일합니다.

<img width="292" height="173" alt="image" src="https://github.com/user-attachments/assets/bf8cc378-b0a6-4608-b888-ee63a72e6ae2" />

### 스택의 특징

* **LIFO (Last In First Out, 후입선출)**: 가장 나중에 들어온 것이 가장 먼저 나옴
* **한쪽 끝에서만 접근**: 데이터의 삽입과 삭제가 같은 끝에서 이루어짐
* **순차적 접근**: 중간에 있는 데이터에 직접 접근 불가능
* **동적 크기**: 필요에 따라 크기가 변할 수 있음

### 언제 사용하는가?

* **함수의 콜스택**: 함수 호출 시 복귀 주소 저장
* **문자열 역순 출력**: 문자열을 스택에 넣고 꺼내면 역순 출력
* **연산자 후위표기법**: 중위표기법을 후위표기법으로 변환
* **괄호 검사**: 괄호의 짝이 맞는지 확인
* **브라우저 뒤로가기**: 방문한 페이지들을 스택에 저장
* **실행 취소(Undo)**: 작업 내역을 스택에 저장

## 2. 스택의 기본 연산

### 2.1 핵심 연산

```kotlin
// 데이터 삽입
fun push(element: T): Unit

// 데이터 삭제 및 반환
fun pop(): T?

// 최상위 데이터 확인 (삭제하지 않음)
fun peek(): T?

// 스택이 비어있는지 확인
fun isEmpty(): Boolean

// 스택이 가득 찼는지 확인
fun isFull(): Boolean

// 스택의 크기 반환
fun size(): Int
```

### 2.2 스택 포인터(SP)

스택 포인터는 **다음 값이 들어갈 위치를 가리키는 인덱스**입니다.

```kotlin
private var sp: Int = -1  // 스택 포인터 초기값
```

> **스택 포인터의 역할**  
> * 다음 데이터가 들어갈 위치를 기억
> * 현재 스택의 크기를 추적
> * push/pop 연산 시 위치 정보 제공

## 3. 배열을 이용한 스택 구현

### 3.1 기본 스택 구현

```kotlin
class ArrayStack<T>(private val maxSize: Int) {
    private val stack: Array<Any?> = Array(maxSize) { null }
    private var sp: Int = -1  // 스택 포인터
    
    fun push(element: T): Boolean {
        if (isFull()) {
            return false
        }
        
        stack[++sp] = element
        return true
    }
    
    fun pop(): T? {
        if (isEmpty()) {
            return null
        }
        
        @Suppress("UNCHECKED_CAST")
        return stack[sp--] as T
    }
    
    fun peek(): T? {
        if (isEmpty()) {
            return null
        }
        
        @Suppress("UNCHECKED_CAST")
        return stack[sp] as T
    }
    
    fun isEmpty(): Boolean {
        return sp == -1
    }
    
    fun isFull(): Boolean {
        return sp + 1 == maxSize
    }
    
    fun size(): Int {
        return sp + 1
    }
}
```

### 3.2 동적 배열 스택 구현

고정 크기 제한을 없애기 위해 동적 배열을 사용합니다.

```kotlin
class DynamicArrayStack<T> {
    private var stack: Array<Any?> = Array(INITIAL_SIZE) { null }
    private var sp: Int = -1
    private var capacity: Int = INITIAL_SIZE
    
    companion object {
        private const val INITIAL_SIZE = 10
        private const val GROWTH_FACTOR = 2
    }
    
    fun push(element: T) {
        if (isFull()) {
            expandCapacity()
        }
        
        stack[++sp] = element
    }
    
    fun pop(): T? {
        if (isEmpty()) {
            return null
        }
        
        @Suppress("UNCHECKED_CAST")
        return stack[sp--] as T
    }
    
    fun peek(): T? {
        if (isEmpty()) {
            return null
        }
        
        @Suppress("UNCHECKED_CAST")
        return stack[sp] as T
    }
    
    fun isEmpty(): Boolean {
        return sp == -1
    }
    
    fun isFull(): Boolean {
        return sp + 1 == capacity
    }
    
    fun size(): Int {
        return sp + 1
    }
    
    private fun expandCapacity() {
        val newCapacity = capacity * GROWTH_FACTOR
        val newStack = Array(newCapacity) { null }
        
        // 기존 데이터 복사
        stack.copyInto(newStack, 0, 0, capacity)
        
        stack = newStack
        capacity = newCapacity
    }
}
```

## 4. 연결리스트를 이용한 스택 구현

### 4.1 노드 클래스

```kotlin
data class Node<T>(
    val data: T,
    var next: Node<T>? = null
)
```

### 4.2 연결리스트 스택

```kotlin
class LinkedListStack<T> {
    private var top: Node<T>? = null
    private var size: Int = 0
    
    fun push(element: T) {
        val newNode = Node(element)
        newNode.next = top
        top = newNode
        size++
    }
    
    fun pop(): T? {
        if (isEmpty()) {
            return null
        }
        
        val poppedData = top?.data
        top = top?.next
        size--
        
        return poppedData
    }
    
    fun peek(): T? {
        return top?.data
    }
    
    fun isEmpty(): Boolean {
        return top == null
    }
    
    fun getSize(): Int {
        return size
    }
}
```

## 5. 실제 활용 예제

### 5.1 괄호 검사

```kotlin
fun isValidParentheses(s: String): Boolean {
    val stack = ArrayDeque<Char>()
    
    for (char in s) {
        when (char) {
            '(', '{', '[' -> stack.addLast(char)
            ')' -> if (stack.isEmpty() || stack.removeLast() != '(') return false
            '}' -> if (stack.isEmpty() || stack.removeLast() != '{') return false
            ']' -> if (stack.isEmpty() || stack.removeLast() != '[') return false
        }
    }
    
    return stack.isEmpty()
}
```

### 5.2 문자열 역순 출력

```kotlin
fun reverseString(s: String): String {
    val stack = ArrayDeque<Char>()
    
    // 문자열을 스택에 push
    for (char in s) {
        stack.addLast(char)
    }
    
    // 스택에서 pop하여 역순으로 출력
    val result = StringBuilder()
    while (stack.isNotEmpty()) {
        result.append(stack.removeLast())
    }
    
    return result.toString()
}
```

### 5.3 후위표기법 계산

```kotlin
fun evaluatePostfix(expression: String): Int {
    val stack = ArrayDeque<Int>()
    
    for (char in expression) {
        when {
            char.isDigit() -> stack.addLast(char.toString().toInt())
            char == '+' -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a + b)
            }
            char == '-' -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a - b)
            }
            char == '*' -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a * b)
            }
            char == '/' -> {
                val b = stack.removeLast()
                val a = stack.removeLast()
                stack.addLast(a / b)
            }
        }
    }
    
    return stack.removeLast()
}
```

## 6. 시간 복잡도

| 연산 | 배열 스택 | 연결리스트 스택 |
|------|-----------|-----------------|
| push() | O(1) | O(1) |
| pop() | O(1) | O(1) |
| peek() | O(1) | O(1) |
| isEmpty() | O(1) | O(1) |
| size() | O(1) | O(1) |

> **참고**: 동적 배열의 경우 확장 시 O(n) 시간이 소요되지만,  
> 분할 상환 분석(Amortized Analysis)에 의해 평균적으로 O(1)로 볼 수 있습니다.

## 7. 공간 복잡도

* **배열 스택**: O(n) - 최대 크기만큼 메모리 할당
* **연결리스트 스택**: O(n) - 실제 데이터 개수만큼 메모리 사용
* **동적 배열 스택**: O(n) - 필요에 따라 메모리 확장

## 8. 장단점

### 8.1 장점

* **빠른 연산**: push, pop 연산이 O(1) 시간 복잡도
* **메모리 효율성**: 필요한 만큼만 메모리 사용
* **구현 간단**: 기본적인 자료구조로 구현이 쉬움
* **LIFO 특성**: 특정 상황에서 유용한 후입선출 특성

### 8.2 단점

* **제한된 접근**: 중간 데이터에 직접 접근 불가능
* **오버플로우**: 고정 크기 배열의 경우 스택 오버플로우 가능
* **메모리 낭비**: 배열 스택의 경우 사용하지 않는 공간 존재
