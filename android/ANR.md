# ANR

> ANR(Application Not Responding)은 안드로이드 앱에서 사용자 입력에 장시간 반응하지 않을 때 발생하는 오류입니다.  
> 이번 글에서는 ANR의 정의, 발생 원인, 그리고 방지 방법에 대해 실무 중심으로 정리합니다.  

---

## 1. ANR이란?

ANR은 **Application Not Responding**의 약자로, 앱이 일정 시간 동안 사용자 입력(터치, 버튼 클릭 등)에 응답하지 않으면 시스템에서 강제로 띄우는 팝업입니다.  
대표적으로 다음과 같은 메시지가 나타납니다:

> 앱이 응답하지 않습니다. 종료하시겠습니까?

ANR은 주로 **메인(UI) 스레드**가 차단(block)되어 있을 때 발생합니다.

---

## 2. ANR 발생 조건

안드로이드 시스템은 아래 조건에 해당할 경우 ANR을 감지합니다:

* **UI 이벤트 처리 지연**: 5초 이상 입력 이벤트를 처리하지 못할 때
* **BroadcastReceiver 실행 지연**: onReceive() 메서드가 10초 이내에 종료되지 않을 때
* **Service 실행 지연**:

  * 일반 서비스: 20초 이내에 시작하지 못할 때
  * 포그라운드 서비스: startForegroundService() 호출 후 5초 이내에 startForeground() 호출하지 않을 때

이런 경우 시스템은 traces.txt 로그를 기록하며, 사용자는 앱이 멈춘 것으로 인식합니다.

---

## 3. ANR 방지 방법

### 3-1. 메인 스레드에서 무거운 작업 금지

네트워크 호출, DB 쿼리, 파일 입출력 등은 반드시 백그라운드 스레드나 Coroutine에서 처리해야 합니다.

```kotlin
viewModelScope.launch {
    val result = withContext(Dispatchers.IO) {
        repository.fetchData()
    }
    _uiState.value = result
}
```

### 3-2. UI 생명주기 메서드 최소화

onCreate(), onResume() 등에서는 무거운 작업을 피하고, 필요한 초기 작업만 수행합니다.

### 3-3. 사용자에게 진행 상황 제공

진행 중인 작업이 있다면 ProgressBar, Loading 메시지를 통해 사용자에게 피드백을 주어야 합니다.

### 3-4. StrictMode 활용

개발 중 StrictMode를 활성화하면 메인 스레드에서의 위험한 작업을 감지할 수 있습니다.

```kotlin
StrictMode.setThreadPolicy(
    StrictMode.ThreadPolicy.Builder()
        .detectAll()
        .penaltyLog()
        .build()
)
```

---

## 참고 자료

* [Android Developers - ANR](https://developer.android.com/topic/performance/vitals/anr)
* [StrictMode](https://developer.android.com/reference/android/os/StrictMode)
