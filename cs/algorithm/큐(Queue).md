# 큐(Queue)

> 큐는 **선입선출(FIFO: First In First Out)** 원칙을 따르는 선형 자료구조입니다.  
> 먼저 들어온 데이터가 먼저 나가는 구조로, 일상생활에서 줄을 서는 것과 같은 개념입니다.

## 1. 큐란?

`큐(Queue)`는 **한쪽 끝에서 삽입하고 다른 한쪽 끝에서 삭제하는 선형 자료구조**입니다.

### 큐의 특징

* **선입선출(FIFO)**: 먼저 들어온 데이터가 먼저 나감
* **한쪽 방향**: 삽입과 삭제가 서로 다른 끝에서 이루어짐
* **순서 보장**: 데이터의 순서가 유지됨
* **동적 크기**: 필요에 따라 크기가 변할 수 있음

### 큐의 활용 예시

* **프린터 대기열**: 인쇄 작업을 순서대로 처리
* **메시지 큐**: 비동기 메시지 처리
* **BFS 알고리즘**: 그래프 탐색에서 사용
* **버퍼링**: 데이터 스트림 처리
* **작업 스케줄링**: CPU 작업 스케줄링

## 2. 큐의 기본 연산

### 2.1 주요 연산

```kotlin
// 큐의 기본 연산들
interface Queue<T> {
    fun enqueue(element: T)    // 삽입 (rear)
    fun dequeue(): T?          // 삭제 (front)
    fun peek(): T?             // 맨 앞 요소 확인
    fun isEmpty(): Boolean     // 비어있는지 확인
    fun isFull(): Boolean      // 가득 찬지 확인
    fun size(): Int            // 크기 반환
}
```

### 2.2 큐 포인터

큐는 두 개의 포인터를 사용합니다:

* **Front**: 삭제할 위치 (맨 앞)
* **Rear**: 삽입할 위치 (맨 뒤)

```kotlin
class ArrayQueue<T>(private val maxSize: Int) {
    private val queue: Array<Any?> = Array(maxSize) { null }
    private var front: Int = 0  // 삭제 위치
    private var rear: Int = -1  // 삽입 위치
    private var size: Int = 0   // 현재 크기
}
```

## 3. 배열을 이용한 큐 구현

### 3.1 기본 큐 구현

```kotlin
class ArrayQueue<T>(private val maxSize: Int) {
    private val queue: Array<Any?> = Array(maxSize) { null }
    private var front: Int = 0
    private var rear: Int = -1
    private var size: Int = 0

    fun enqueue(element: T): Boolean {
        if (isFull()) return false
        
        rear = (rear + 1) % maxSize  // 원형 큐 구현
        queue[rear] = element
        size++
        return true
    }

    fun dequeue(): T? {
        if (isEmpty()) return null
        
        val element = queue[front] as T
        front = (front + 1) % maxSize
        size--
        return element
    }

    fun peek(): T? {
        return if (isEmpty()) null else queue[front] as T
    }

    fun isEmpty(): Boolean = size == 0
    fun isFull(): Boolean = size == maxSize
    fun size(): Int = size
}
```

### 3.2 순환큐 (Circular Queue) 상세 설명

순환큐는 **배열의 끝과 시작을 연결하여 메모리를 효율적으로 사용하는 큐**입니다.

#### 순환큐의 필요성

일반 배열 큐의 문제점:
```kotlin
// 문제가 있는 일반 큐
class ProblematicQueue<T>(private val maxSize: Int) {
    private val queue: Array<Any?> = Array(maxSize) { null }
    private var front: Int = 0
    private var rear: Int = -1

    fun enqueue(element: T): Boolean {
        if (rear >= maxSize - 1) return false  // 배열 끝에 도달하면 실패
        
        queue[++rear] = element
        return true
    }

    fun dequeue(): T? {
        if (front > rear) return null
        
        return queue[front++] as T
    }
}
```

**문제점:**
* 배열 앞부분이 비어있어도 새로운 요소를 추가할 수 없음
* 메모리 낭비 발생

#### 순환큐의 해결책

```kotlin
class CircularQueue<T>(private val maxSize: Int) {
    private val queue: Array<Any?> = Array(maxSize) { null }
    private var front: Int = 0
    private var rear: Int = -1
    private var size: Int = 0

    fun enqueue(element: T): Boolean {
        if (isFull()) return false
        
        // 순환: 배열 끝에 도달하면 처음으로 돌아감
        rear = (rear + 1) % maxSize
        queue[rear] = element
        size++
        return true
    }

    fun dequeue(): T? {
        if (isEmpty()) return null
        
        val element = queue[front] as T
        // 순환: 배열 끝에 도달하면 처음으로 돌아감
        front = (front + 1) % maxSize
        size--
        return element
    }

    fun peek(): T? = if (isEmpty()) null else queue[front] as T
    fun isEmpty(): Boolean = size == 0
    fun isFull(): Boolean = size == maxSize
    fun size(): Int = size
}
```

#### 순환큐 동작 예시

```kotlin
// 순환큐 사용 예시
val queue = CircularQueue<Int>(5)

queue.enqueue(1)  // [1, _, _, _, _] front=0, rear=0
queue.enqueue(2)  // [1, 2, _, _, _] front=0, rear=1
queue.enqueue(3)  // [1, 2, 3, _, _] front=0, rear=2
queue.enqueue(4)  // [1, 2, 3, 4, _] front=0, rear=3
queue.enqueue(5)  // [1, 2, 3, 4, 5] front=0, rear=4

queue.dequeue()   // [_, 2, 3, 4, 5] front=1, rear=4
queue.dequeue()   // [_, _, 3, 4, 5] front=2, rear=4

queue.enqueue(6)  // [6, _, 3, 4, 5] front=2, rear=0 (순환!)
queue.enqueue(7)  // [6, 7, 3, 4, 5] front=2, rear=1 (순환!)
```

#### 순환큐의 장점

* **메모리 효율성**: 배열의 모든 공간을 활용
* **연속적인 삽입/삭제**: 배열 끝에 도달해도 계속 사용 가능
* **일정한 성능**: O(1) 시간복잡도 유지

#### 순환큐의 주의사항

```kotlin
// 주의: size 변수 없이 front와 rear만으로는 빈 큐와 가득 찬 큐를 구분하기 어려움
class CircularQueueWithoutSize<T>(private val maxSize: Int) {
    private val queue: Array<Any?> = Array(maxSize) { null }
    private var front: Int = 0
    private var rear: Int = -1

    fun isEmpty(): Boolean {
        return front == (rear + 1) % maxSize && queue[front] == null
    }

    fun isFull(): Boolean {
        return front == (rear + 1) % maxSize && queue[front] != null
    }
}
```

### 3.3 동적 배열 큐 구현

```kotlin
class DynamicArrayQueue<T> {
    private var queue: Array<Any?> = Array(INITIAL_SIZE) { null }
    private var front: Int = 0
    private var rear: Int = -1
    private var size: Int = 0

    companion object {
        private const val INITIAL_SIZE = 10
    }

    fun enqueue(element: T) {
        if (isFull()) {
            resize()
        }
        
        rear = (rear + 1) % queue.size
        queue[rear] = element
        size++
    }

    fun dequeue(): T? {
        if (isEmpty()) return null
        
        val element = queue[front] as T
        front = (front + 1) % queue.size
        size--
        return element
    }

    private fun resize() {
        val newCapacity = queue.size * 2
        val newQueue = Array(newCapacity) { null }
        
        for (i in 0 until size) {
            newQueue[i] = queue[(front + i) % queue.size]
        }
        
        queue = newQueue
        front = 0
        rear = size - 1
    }

    fun peek(): T? = if (isEmpty()) null else queue[front] as T
    fun isEmpty(): Boolean = size == 0
    fun isFull(): Boolean = size == queue.size
    fun size(): Int = size
}
```

## 4. 연결리스트를 이용한 큐 구현

### 4.1 노드 클래스

```kotlin
data class Node<T>(
    val data: T,
    var next: Node<T>? = null
)
```

### 4.2 연결리스트 큐

```kotlin
class LinkedListQueue<T> {
    private var front: Node<T>? = null
    private var rear: Node<T>? = null
    private var size: Int = 0

    fun enqueue(element: T) {
        val newNode = Node(element)
        
        if (isEmpty()) {
            front = newNode
            rear = newNode
        } else {
            rear?.next = newNode
            rear = newNode
        }
        size++
    }

    fun dequeue(): T? {
        if (isEmpty()) return null
        
        val element = front?.data
        front = front?.next
        
        if (front == null) {
            rear = null
        }
        size--
        return element
    }

    fun peek(): T? = front?.data
    fun isEmpty(): Boolean = front == null
    fun size(): Int = size
}
```

## 5. 큐의 종류

### 5.1 일반 큐 (Queue)

* 가장 기본적인 큐
* 선입선출 원칙

### 5.2 우선순위 큐 (Priority Queue)

```kotlin
class PriorityQueue<T : Comparable<T>> {
    private val heap = mutableListOf<T>()
    
    fun enqueue(element: T) {
        heap.add(element)
        heapifyUp(heap.size - 1)
    }
    
    fun dequeue(): T? {
        if (heap.isEmpty()) return null
        
        val result = heap[0]
        heap[0] = heap.last()
        heap.removeAt(heap.size - 1)
        
        if (heap.isNotEmpty()) {
            heapifyDown(0)
        }
        
        return result
    }
    
    private fun heapifyUp(index: Int) {
        var current = index
        while (current > 0) {
            val parent = (current - 1) / 2
            if (heap[current] < heap[parent]) {
                heap.swap(current, parent)
                current = parent
            } else break
        }
    }
    
    private fun heapifyDown(index: Int) {
        var current = index
        while (true) {
            val left = 2 * current + 1
            val right = 2 * current + 2
            var smallest = current
            
            if (left < heap.size && heap[left] < heap[smallest]) {
                smallest = left
            }
            if (right < heap.size && heap[right] < heap[smallest]) {
                smallest = right
            }
            
            if (smallest == current) break
            
            heap.swap(current, smallest)
            current = smallest
        }
    }
    
    private fun MutableList<T>.swap(i: Int, j: Int) {
        val temp = this[i]
        this[i] = this[j]
        this[j] = temp
    }
}
```
* [ArrayDeque 문서](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-array-deque/) 
