# Dispatchers

> Kotlin Coroutines에서 코루틴이 어느 스레드에서 실행될지를 지정해주는 핵심 요소가 바로 `Dispatcher`입니다.  
> 어떤 작업을 어떤 스레드에서 실행할지는 성능과 안정성에 큰 영향을 주기 때문에 Dispatcher를 올바르게 선택하는 것은 매우 중요합니다.  
> 이 글에서는 코루틴에서 제공하는 기본 Dispatcher들의 종류와 특징, 언제 어떤 Dispatcher를 사용하는지에 대해 정리해보겠습니다.  

---

## 1. Dispatchers.Default

### 정의

기본 CoroutineDispatcher이며, CPU 연산에 최적화되어 있다.  
공유 스레드 풀에서 코어 수에 따라 자동으로 스레드 수를 조절한다.  
기본적으로 제한된 스레드 수로 인해, 동시에 많은 작업을 수행하면 대기 상태가 발생할 수 있다.  

### 사용 예시

```kotlin
launch(Dispatchers.Default) {
    // 무거운 계산 로직 처리
}
```

### 사용 시점

* 정렬, 파싱, 압축 등 CPU 중심 연산
* 복잡한 데이터 변환, 이미지 처리 등
* 동시에 실행되는 작업 수가 적은 경우에 적합

---

## 2. Dispatchers.IO

### 정의

입출력(I/O) 작업에 최적화된 Dispatcher.  
많은 수의 스레드를 사용하는 스레드 풀로 구성되어 있다.  
`Dispatchers.Default`보다 훨씬 더 많은 작업을 동시에 처리할 수 있도록 설계되어 있으며, 필요에 따라 자동 확장된다.  

### 사용 예시

```kotlin
launch(Dispatchers.IO) {
    // 파일 읽기, 네트워크 요청, DB 접근 등
}
```

### 사용 시점

* 파일 시스템 작업
* Retrofit 등 네트워크 호출
* Room, DataStore 등 데이터 저장소 접근
* 대량의 비동기 작업을 동시에 처리해야 하는 경우

---

## 3. Dispatchers.Main

### 정의

Main/UI 스레드에서 코루틴을 실행하는 Dispatcher.  
UI를 업데이트하거나 사용자 상호작용을 처리하는 데 사용된다.  

### 사용 예시

```kotlin
launch(Dispatchers.Main) {
    // UI 변경
}
```

### 사용 시점

* UI 요소 변경
* 사용자 입력 처리
* LiveData 관찰 결과 처리

> Android에서 사용하려면 `kotlinx-coroutines-android` 의존성 필요

---

## 4. Dispatchers.Unconfined

### 정의

코루틴을 호출한 현재 스레드에서 실행하고, `suspend` 후에는 중단점 다음 위치의 스레드에서 재개된다.  
명시적인 스레드 전환이 없기 때문에 성능 면에서는 이점이 있으나, 예측 불가능한 동작이 발생할 수 있다.  

### 사용 예시

```kotlin
launch(Dispatchers.Unconfined) {
    // 현재 스레드에서 시작됨
}
```

### 사용 시점

* 코루틴이 매우 짧고 suspend 되지 않을 때
* 테스트 환경에서 디버깅 용도
* 일반적인 실무에서는 사용 지양 (suspend 이후 재개 스레드가 달라질 수 있어, UI 코드에서 사용 시 앱 크래시 가능성 있음)

---

## 5. Dispatcher 선택 기준

| 상황                | 권장 Dispatcher                 |
| ----------------- | ----------------------------- |
| UI 변경             | `Dispatchers.Main`            |
| 네트워크, DB, 파일 작업   | `Dispatchers.IO`              |
| CPU 바운드 작업        | `Dispatchers.Default`         |
| 특별한 제어 흐름, 테스트 용도 | `Dispatchers.Unconfined` (지양) |

---

## 참고 자료

* [공식 문서 - Coroutine Dispatchers](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)
* [Android Developers - Coroutines](https://developer.android.com/kotlin/coroutines)
