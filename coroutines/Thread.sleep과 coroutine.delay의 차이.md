# Thread.sleep과 coroutine.delay의 차이

> 안드로이드 개발을 하다 보면 비동기 처리와 지연 실행이 필요할 때가 있습니다.  
> Java의 Thread.sleep()과 Kotlin 코루틴의 delay() 모두 시간 지연을 위한 함수이지만,  
> 내부 동작 방식에 차이가 있습니다.    
> 이번 글에서는 두 함수의 차이점을 자세히 알아보겠습니다.

---

## 1. 기본 개념 비교

### 1.1 Thread.sleep()

**Java의 전통적인 스레드 블로킹 방식**

```kotlin
// Thread.sleep() 사용 예시
fun traditionalDelay() {
    println("시작: ${System.currentTimeMillis()}")
    
    Thread.sleep(1000) // 1초 동안 현재 스레드를 블로킹
    
    println("완료: ${System.currentTimeMillis()}")
}
```

### 1.2 코루틴 delay()

**Kotlin 코루틴의 논블로킹 지연 방식**

```kotlin
// 코루틴 delay() 사용 예시
suspend fun coroutineDelay() {
    println("시작: ${System.currentTimeMillis()}")
    
    delay(1000) // 1초 동안 논블로킹 지연
    
    println("완료: ${System.currentTimeMillis()}")
}
```

---

## 2. 핵심 차이점

### 2.1 스레드 블로킹 vs 논블로킹

**Thread.sleep()의 문제점:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    private fun demonstrateThreadSleep() {
        // 메인 스레드에서 Thread.sleep() 사용 - ANR 발생!
        Thread.sleep(5000) // 5초 동안 UI가 완전히 멈춤
        
        // 사용자는 5초 동안 앱이 멈춘 것처럼 보임
        // 시스템이 ANR(Application Not Responding) 다이얼로그 표시
    }
    
    private fun demonstrateThreadSleepInBackground() {
        // 백그라운드 스레드에서 사용은 가능하지만 비효율적
        Thread {
            Thread.sleep(5000) // 스레드가 5초 동안 아무것도 하지 않음
            runOnUiThread {
                // UI 업데이트
            }
        }.start()
    }
}
```

**코루틴 delay()의 장점:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    private fun demonstrateCoroutineDelay() {
        // 메인 스레드에서도 안전하게 사용 가능
        lifecycleScope.launch {
            delay(5000) // 5초 동안 지연하지만 UI는 계속 반응함
            
            // UI 업데이트
            updateUI()
        }
        
        // 사용자는 다른 UI 요소들과 계속 상호작용 가능
    }
    
    private fun updateUI() {
        // UI 업데이트 로직
    }
}
```

### 2.2 리소스 효율성

**Thread.sleep()의 리소스 사용:**

```kotlin
// 비효율적인 스레드 사용
fun inefficientThreadUsage() {
    repeat(1000) { index ->
        Thread {
            Thread.sleep(1000) // 각 스레드가 1초 동안 대기
            println("작업 $index 완료")
        }.start()
    }
    // 1000개의 스레드가 생성되어 시스템 리소스 낭비
}
```

**코루틴 delay()의 효율성:**

```kotlin
// 효율적인 코루틴 사용
suspend fun efficientCoroutineUsage() {
    repeat(1000) { index ->
        launch {
            delay(1000) // 논블로킹 지연
            println("작업 $index 완료")
        }
    }
    // 소수의 스레드로 1000개의 작업 처리 가능
}
```

## 3. 마무리

### 3.1 핵심 정리

| 구분 | Thread.sleep() | 코루틴 delay() |
|------|----------------|----------------|
| **스레드 블로킹** | 블로킹 | 논블로킹 |
| **UI 스레드 사용** | ANR 발생 | 안전함 |
| **리소스 효율성** | 비효율적 | 효율적 |
| **메모리 사용량** | 높음 | 낮음 |
| **사용 복잡도** | 간단 | 코루틴 이해 필요 |
| **안드로이드 권장** | 비권장 | 권장 |
