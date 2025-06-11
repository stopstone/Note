# withContext

## 1. withContext란?

`withContext`는 Kotlin의 코루틴에서 **다른 CoroutineContext로 일시적으로 전환**하여 블록을 실행할 수 있게 해주는 suspend 함수입니다.  
기존 컨텍스트를 변경하고, 결과를 반환하며, 종료 시 원래 컨텍스트로 돌아옵니다.

```kotlin
val result = withContext(Dispatchers.IO) {
    // 여기 블록은 IO 스레드에서 실행됨
    fetchDataFromNetwork()
}
```

> `launch`나 `async`와 달리 새로운 Job을 생성하지 않으며, **현재 코루틴의 Job 안에서 컨텍스트만 바꿔** 실행합니다.

---

## 2. 언제 사용하나요?

### 2-1. **무거운 작업을 다른 스레드에서 처리하고 싶을 때**

* 예: 데이터베이스 쿼리, 네트워크 요청, 파일 입출력 등

```kotlin
suspend fun loadUserData(): User = withContext(Dispatchers.IO) {
    userDao.getUser()
}
```

### 2-2. **Main 스레드에서 UI 처리를 보장하고 싶을 때**

* 예: IO → Main으로 전환 후 결과 표시

```kotlin
val user = withContext(Dispatchers.IO) { getUserFromRemote() }
withContext(Dispatchers.Main) { showUser(user) }
```

### 2-3. **컨텍스트 전환이 필요하지만, 병렬성이 필요 없을 때**

* `launch`나 `async`는 새로운 코루틴을 만들어 병렬 실행하지만, `withContext`는 단순 컨텍스트 변경이 목적일 때 사용합니다.

---

## 3. launch/async와의 차이점

| 항목     | withContext | launch / async       |
| ------ | ----------- | -------------------- |
| Job 생성 | 없음          | 새 코루틴 생성             |
| 반환값    | 있음          | launch 없음 / async 있음 |
| 병렬성    | 없음          | 병렬 처리 가능             |
| 목적     | 컨텍스트 전환     | 비동기 병렬 작업 실행         |

---

## 참고 자료

* [공식 문서: kotlinx.coroutines - withContext](https://kotlinlang.org/docs/coroutines-basics.html#withcontext)
* [Google Android 개발자 문서: Coroutine Best Practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
