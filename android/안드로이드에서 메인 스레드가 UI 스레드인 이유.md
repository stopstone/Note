# 안드로이드에서 메인 스레드가 UI 스레드인 이유

> 안드로이드 개발에서 UI 처리를 제대로 이해하기 위해서는 메인 스레드와 백그라운드 스레드의 차이를 정확히 알고 있어야 합니다.    
> 본 글에서는 각각의 스레드가 무엇을 의미하는지, 어떤 역할을 하는지, 그리고 UI 업데이트 시 주의해야 할 사항들을 정리하였습니다.   

## 1. 왜 메인 스레드가 UI 스레드인가?

안드로이드는 앱의 UI 요소를 단일 스레드에서 처리하도록 설계되어 있습니다. 이를 단일 스레드 모델(Single Thread Model)이라 하며, 주요 이유는 다음과 같습니다.

* **스레드 안전성 확보**: UI 요소는 상태가 자주 바뀌고 사용자와의 상호작용이 많습니다. 여러 스레드에서 동시에 UI 요소를 수정하게 되면 충돌이나 Race Condition이 발생할 수 있습니다.
* **단순한 설계**: 모든 UI 관련 코드를 한 스레드(즉, 메인 스레드)에서 실행하도록 강제함으로써 코드의 예측 가능성과 안정성을 높입니다.
* **일관된 사용자 경험**: UI 처리가 분산되어 있으면 동기화 문제가 발생하고, 사용자에게 불안정한 인터페이스를 제공할 수 있습니다. 이를 방지하기 위해 UI는 항상 메인 스레드에서만 처리됩니다.

이러한 이유로, 안드로이드에서는 **메인 스레드 = UI 스레드**라는 기본 규칙이 적용됩니다.  
따라서 **UI 변경은 반드시 메인 스레드에서만 가능**하며, 긴 작업은 백그라운드 스레드로 분리하는 것이 필수적입니다.

## 2. 메인 스레드 (Main Thread)

### 2.1 정의

* 메인 스레드는 앱의 UI를 담당하는 스레드입니다.
* 화면 출력, 사용자 이벤트 처리 등 대부분의 UI 작업이 메인 스레드에서 수행됩니다.

### 2.2 특징

* UI 요소를 직접 수정할 수 있습니다.
* 네트워크 요청이나 파일 입출력 등 시간이 오래 걸리는 작업을 수행할 경우, ANR(Application Not Responding)이 발생할 수 있습니다.

## 3. 백그라운드 스레드 (Background Thread)

### 3.1 정의

* 백그라운드 작업을 수행하는 스레드입니다.
* 네트워크 통신, 데이터베이스 쿼리, 이미지 처리와 같은 무거운 작업을 수행할 때 사용됩니다.

### 3.2 특징

* UI 요소를 직접 수정할 수 없습니다.
* 작업이 끝난 후에는 메인 스레드로 결과를 전달하여 UI를 업데이트해야 합니다.

## 4. UI 업데이트 시 주의사항

* UI는 반드시 메인 스레드에서만 수정해야 합니다.
* 백그라운드 스레드에서 UI를 직접 수정하려고 하면 예외가 발생하거나 앱이 비정상적으로 종료될 수 있습니다.

## 5. 백그라운드 스레드에서 메인 스레드로 UI 업데이트하는 방법

### 5.1 runOnUiThread 사용

```kotlin
Thread {
    val result = "작업 완료"
    runOnUiThread {
        textView.text = result
    }
}.start()
```

### 5.2 Handler 사용

```kotlin
val handler = Handler(Looper.getMainLooper())
Thread {
    val result = "작업 완료"
    handler.post {
        textView.text = result
    }
}.start()
```

### 5.3 LiveData 사용

```kotlin
val resultLiveData = MutableLiveData<String>()

resultLiveData.observe(this) { result ->
    textView.text = result
}

GlobalScope.launch(Dispatchers.IO) {
    val result = "작업 완료"
    resultLiveData.postValue(result)
}
```
