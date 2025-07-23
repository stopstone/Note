# Async/Await 패턴의 내부 동작

> Async/Await는 비동기 프로그래밍에서 가장 직관적이고 강력한 패턴 중 하나입니다.    
> 코루틴에서 `async`와 `await`가 어떻게 내부적으로 동작하는지,   
> 그리고 왜 이 패턴이 비동기 코드를 동기 코드처럼 깔끔하게 만들어주는지 알아보겠습니다.  

---

## 1. Async/Await 패턴이란?

Async/Await는 **비동기 작업을 동기 코드처럼 작성**할 수 있게 해주는 패턴입니다.  
복잡한 콜백 지옥을 피하고, Promise 체이닝보다 더 읽기 쉬운 코드를 만들 수 있어요.

### 전통적인 콜백 방식
```kotlin
fun fetchUserData(userId: String, callback: (User) -> Unit) {
    // 비동기 작업
    callback(user)
}
```

### Async/Await 방식
```kotlin
suspend fun fetchUserData(userId: String): User {
    // 비동기 작업을 동기처럼 작성
    return user
}
```

---

## 3. 코루틴에서 Async/Await의 내부 동작

### async의 내부 동작

`async`는 **Deferred 객체를 반환**하는 코루틴 빌더입니다. 내부적으로는 다음과 같이 동작해요:

```kotlin
val deferred = async {
    delay(1000) // 일시 중단
    "결과 값"
}
```

**내부 동작 과정:**

1. **새로운 코루틴 생성**: `async`는 새로운 코루틴을 생성합니다
2. **Deferred 객체 반환**: 즉시 `Deferred<T>` 객체를 반환합니다
3. **백그라운드 실행**: 코루틴은 백그라운드에서 비동기적으로 실행됩니다
4. **상태 관리**: `Deferred`는 코루틴의 상태를 추적합니다

### await의 내부 동작

`await()`는 **결과가 준비될 때까지 일시 중단**하는 함수입니다:

```kotlin
val result = deferred.await() // 여기서 일시 중단
```

**내부 동작 과정:**

1. **상태 확인**: `Deferred`의 상태를 확인합니다
2. **완료 대기**: 아직 완료되지 않았다면 현재 코루틴을 일시 중단합니다
3. **재개**: 결과가 준비되면 자동으로 코루틴을 재개합니다
4. **결과 반환**: 완성된 결과를 반환합니다

---

## 3. 상태 머신(State Machine)의 역할

코루틴의 Async/Await는 **상태 머신**을 사용해서 내부적으로 동작합니다.

### 상태 머신의 동작 원리

```kotlin
suspend fun fetchUserData(userId: String): User {
    val user = api.getUser(userId) // 일시 중단 지점 1
    val posts = api.getUserPosts(userId) // 일시 중단 지점 2
    return User(user, posts)
}
```

**내부적으로 컴파일러가 생성하는 코드:**

```kotlin
fun fetchUserData(userId: String, continuation: Continuation<User>): Any {
    when (continuation.label) {
        0 -> {
            // 첫 번째 일시 중단 지점
            continuation.label = 1
            return api.getUser(userId, continuation)
        }
        1 -> {
            // 두 번째 일시 중단 지점
            val user = continuation.result as User
            continuation.label = 2
            return api.getUserPosts(userId, continuation)
        }
        2 -> {
            // 최종 결과 반환
            val posts = continuation.result as List<Post>
            return User(user, posts)
        }
    }
}
```

### Continuation의 역할

`Continuation`은 **코루틴의 실행 상태를 저장**하는 객체입니다:

- **label**: 현재 실행 중인 지점을 나타냅니다
- **result**: 이전 단계의 결과를 저장합니다
- **context**: 코루틴의 컨텍스트 정보를 담습니다

---

## 4. 실제 동작 예시

### 단순한 Async/Await 예시

```kotlin
suspend fun fetchData(): String {
    delay(1000) // 네트워크 요청 시뮬레이션
    return "데이터"
}

fun main() = runBlocking {
    val deferred = async { fetchData() }
    val result = deferred.await() // 여기서 일시 중단
    println(result)
}
```

**실행 과정:**

1. `async`가 새로운 코루틴을 생성하고 `Deferred` 반환
2. `await()` 호출 시 코루틴이 일시 중단
3. 백그라운드에서 `fetchData()` 실행
4. `fetchData()` 완료 시 코루틴 재개
5. 결과 반환

### 고급 Async/Await 예시 (면접용)

```kotlin
suspend fun complexAsyncOperation(): Result<Data> {
    return try {
        // 여러 비동기 작업을 병렬로 실행
        val userDeferred = async(Dispatchers.IO) { 
            api.getUser().also { 
                Log.d("Async", "User loaded") 
            }
        }
        
        val postsDeferred = async(Dispatchers.IO) { 
            api.getPosts().also { 
                Log.d("Async", "Posts loaded") 
            }
        }
        
        val settingsDeferred = async(Dispatchers.Default) { 
            processSettings().also { 
                Log.d("Async", "Settings processed") 
            }
        }
        
        // 모든 작업이 완료될 때까지 대기
        val user = userDeferred.await()
        val posts = postsDeferred.await()
        val settings = settingsDeferred.await()
        
        Result.success(Data(user, posts, settings))
    } catch (e: Exception) {
        Log.e("Async", "Error in async operation", e)
        Result.failure(e)
    }
}
```

### 병렬 실행 예시

```kotlin
suspend fun fetchUserAndPosts(userId: String): UserWithPosts {
    val userDeferred = async { api.getUser(userId) }
    val postsDeferred = async { api.getUserPosts(userId) }
    
    val user = userDeferred.await()
    val posts = postsDeferred.await()
    
    return UserWithPosts(user, posts)
}
```

**병렬 실행의 장점:**

- 두 API 호출이 **동시에 실행**됩니다
- 총 시간은 **가장 긴 요청의 시간**만큼만 걸립니다
- 순차 실행보다 **훨씬 빠릅니다**

### Deferred 상태 모니터링

```kotlin
suspend fun monitoredAsyncOperation(): String {
    val deferred = async {
        delay(2000)
        "완료된 결과"
    }
    
    // 상태 모니터링
    deferred.invokeOnCompletion { throwable ->
        when {
            throwable == null -> Log.d("Deferred", "정상 완료")
            deferred.isCancelled -> Log.d("Deferred", "취소됨")
            else -> Log.e("Deferred", "예외 발생: ${throwable.message}")
        }
    }
    
    return deferred.await()
}
```

---

## 5. 에러 처리와 Async/Await

### try-catch를 사용한 에러 처리

```kotlin
suspend fun fetchDataSafely(): Result<String> {
    return try {
        val result = api.fetchData()
        Result.success(result)
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

### 코루틴 예외 처리

```kotlin
val deferred = async {
    throw RuntimeException("에러 발생")
}

try {
    val result = deferred.await() // 여기서 예외 발생
} catch (e: Exception) {
    println("에러 처리: ${e.message}")
}
```

### 고급 에러 처리 (면접용)

```kotlin
suspend fun robustAsyncOperation(): Result<Data> {
    return withTimeoutOrNull(5000L) { // 5초 타임아웃
        try {
            val deferred = async {
                // 복잡한 비동기 작업
                delay(3000)
                if (Random.nextBoolean()) {
                    throw RuntimeException("랜덤 에러")
                }
                "성공 결과"
            }
            
            // 취소 가능한 작업
            deferred.await()
            Result.success(Data(deferred.await()))
        } catch (e: TimeoutCancellationException) {
            Log.w("Async", "작업 타임아웃")
            Result.failure(e)
        } catch (e: CancellationException) {
            Log.w("Async", "작업 취소됨")
            Result.failure(e)
        } catch (e: Exception) {
            Log.e("Async", "예외 발생", e)
            Result.failure(e)
        }
    } ?: Result.failure(TimeoutCancellationException("타임아웃"))
}
```

### Deferred 예외 전파 메커니즘

```kotlin
suspend fun exceptionPropagationExample() {
    val deferred = async {
        delay(1000)
        throw RuntimeException("Deferred에서 발생한 예외")
    }
    
    try {
        deferred.await() // 여기서 예외가 전파됨
    } catch (e: Exception) {
        // Deferred에서 발생한 예외가 여기서 잡힘
        Log.e("Async", "예외 전파됨: ${e.message}")
    }
}
```

---

## 6. 성능 최적화와 주의사항

### 메모리 사용량

Async/Await는 **상태 머신**을 사용하므로 약간의 메모리 오버헤드가 있습니다:

- **Continuation 객체**: 코루틴 상태 저장
- **스택 프레임**: 일시 중단 지점 정보
- **Deferred 객체**: 비동기 작업 상태 관리

### 스레드 풀 활용

```kotlin
// IO 작업용 스레드 풀 사용
val ioDeferred = async(Dispatchers.IO) {
    // 파일 읽기, 네트워크 요청 등
}

// CPU 집약적 작업용 스레드 풀 사용
val cpuDeferred = async(Dispatchers.Default) {
    // 복잡한 계산 등
}
```

### 고급 성능 최적화 (면접용)

#### **1. 구조화된 동시성 활용**

```kotlin
suspend fun structuredConcurrencyExample(): List<Data> {
    return coroutineScope { // 구조화된 동시성
        val deferred1 = async { fetchData1() }
        val deferred2 = async { fetchData2() }
        val deferred3 = async { fetchData3() }
        
        // 모든 작업이 완료될 때까지 대기
        listOf(deferred1.await(), deferred2.await(), deferred3.await())
    }
}
```

#### **2. 메모리 누수 방지**

```kotlin
class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<Data>()
    
    fun loadData() {
        viewModelScope.launch {
            try {
                val deferred = async(Dispatchers.IO) {
                    api.fetchData()
                }
                
                // ViewModel이 소멸되면 자동으로 취소됨
                val result = deferred.await()
                _data.value = result
            } catch (e: CancellationException) {
                Log.d("ViewModel", "작업 취소됨")
            }
        }
    }
}
```

#### **3. 디버깅과 모니터링**

```kotlin
suspend fun monitoredAsyncOperation(): String {
    val startTime = System.currentTimeMillis()
    
    val deferred = async {
        delay(1000)
        "결과"
    }.also { deferred ->
        // 완료 시점 모니터링
        deferred.invokeOnCompletion { throwable ->
            val duration = System.currentTimeMillis() - startTime
            Log.d("Performance", "작업 완료: ${duration}ms")
            
            if (throwable != null) {
                Log.e("Performance", "작업 실패", throwable)
            }
        }
    }
    
    return deferred.await()
}
```

#### **4. 배치 처리 최적화**

```kotlin
suspend fun batchProcessing(items: List<String>): List<Result> {
    return items.chunked(10) // 10개씩 배치 처리
        .map { batch ->
            async {
                batch.map { item ->
                    try {
                        processItem(item)
                    } catch (e: Exception) {
                        Log.e("Batch", "항목 처리 실패: $item", e)
                        null
                    }
                }.filterNotNull()
            }
        }
        .awaitAll() // 모든 배치 완료 대기
        .flatten()
}
```

### 주의사항

#### **1. 취소 처리**
```kotlin
// 잘못된 예시 - 취소되지 않음
val deferred = async { /* 긴 작업 */ }
// deferred.cancel() 호출하지 않음

// 올바른 예시 - 적절한 취소 처리
val deferred = async { /* 긴 작업 */ }
try {
    deferred.await()
} catch (e: CancellationException) {
    // 취소 처리
}
```

#### **2. 스레드 안전성**
```kotlin
// 잘못된 예시 - 스레드 안전하지 않음
var sharedData = ""

suspend fun unsafeAsync() {
    async {
        sharedData = "값" // 여러 스레드에서 동시 접근 가능
    }
}

// 올바른 예시 - 스레드 안전
@Volatile
var sharedData = ""

suspend fun safeAsync() {
    async {
        synchronized(this) {
            sharedData = "값"
        }
    }
}
```

---

## 참고자료

* [Kotlin 공식 문서 - Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
* [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-basics.html)
* [Coroutines Design Document](https://github.com/Kotlin/KEEP/blob/master/proposals/kotlin-coroutines.md)
* [Deferred Interface Documentation](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/)
* [Async/Await Pattern in Kotlin](https://kotlinlang.org/docs/coroutines-composing-suspending-functions.html)

