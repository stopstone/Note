# ActivityResultLauncher로 액티비티 결과 받기

> 안드로이드에서 화면 전환 후 결과를 받아야 하는 경우, 기존의 `startActivityForResult()`와 `onActivityResult()` 방식은 많은 한계가 있었습니다.
> AndroidX에서는 이를 대체하기 위해 `ActivityResultLauncher`와 `registerForActivityResult()`를 도입했습니다.
> 이 글에서는 그 차이점과 사용법을 정리합니다.

## 1. 기존 방식의 한계점

### 1-1. 복잡한 requestCode 관리

* 여러 요청을 처리할 때 직접 `requestCode`를 비교하고 분기해야 함

### 1-2. 프래그먼트에서의 불편함

* 프래그먼트에서 결과를 받기 위해 액티비티로 전달하고 다시 받아야 함

### 1-3. 라이프사이클 문제

* 액티비티나 프래그먼트가 recreate될 경우 결과 처리가 제대로 되지 않을 수 있음

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val intent = Intent(this, SomeActivity::class.java)
    startActivityForResult(intent, REQUEST_CODE)
}

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        val result = data?.getStringExtra("key")
        // 결과 처리
    }
}
```

## 2. 새로운 방식: ActivityResultLauncher

### 2-1. 장점

* requestCode 없이 요청 구분 가능
* 프래그먼트에서도 직관적으로 사용 가능
* 라이프사이클을 고려한 안전한 처리

### 2-2. 사용법 (Activity)

```kotlin
private val launcher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == RESULT_OK) {
        val data = result.data?.getStringExtra("key")
        // 결과 처리
    }
}

val intent = Intent(this, SomeActivity::class.java)
launcher.launch(intent)
```

### 2-3. 사용법 (Fragment)

```kotlin
private val launcher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == RESULT_OK) {
        val data = result.data?.getStringExtra("key")
        // 결과 처리
    }
}

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    someButton.setOnClickListener {
        val intent = Intent(requireContext(), SomeActivity::class.java)
        launcher.launch(intent)
    }
}
```

### 2-4. 커스텀 ActivityResultContract 예시

```kotlin
class GetStringContract : ActivityResultContract<Unit, String?>() {
    override fun createIntent(context: Context, input: Unit): Intent {
        return Intent(context, SomeActivity::class.java)
    }

    override fun parseResult(resultCode: Int, intent: Intent?): String? {
        return if (resultCode == RESULT_OK) intent?.getStringExtra("key") else null
    }
}
```

## 3. 주의할 점

* `registerForActivityResult()`는 액티비티나 프래그먼트의 **생명주기 전에 등록**되어야 하며, **프래그먼트의 경우 onAttach() 이전에 호출되어야 함**
* `launch()`는 `onStart()` 이후에 호출하는 것이 안전함

## 4. 요약 비교

| 항목                | 기존 방식 (`startActivityForResult`) | 새로운 방식 (`ActivityResultLauncher`) |
| ----------------- | -------------------------------- | --------------------------------- |
| requestCode 필요 여부 | O                                | X                                 |
| 프래그먼트 사용          | 불편함                              | 직관적임                              |
| 라이프사이클 안전성        | 낮음                               | 높음                                |
| 코드 구조             | 분산됨                              | 일관되고 모듈화됨                         |

## 참고자료

* [Android Developers - registerForActivityResult](https://developer.android.com/training/basics/intents/result)
