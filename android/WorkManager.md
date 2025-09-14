# WorkManager

> WorkManager는 Android에서 백그라운드 작업을 안전하고 효율적으로 실행하기 위한 Jetpack 라이브러리입니다.  
> 앱이 종료되거나 기기가 재부팅되어도 백그라운드 작업을 보장하며, 배터리 최적화와 함께 안정적인 작업 실행을 제공합니다.  
> 이 글에서는 WorkManager의 핵심 개념과 실제 안드로이드 앱에서의 활용법을 정리합니다.

## 1. WorkManager란?

### 1-1. 정의

**WorkManager**는 Android Jetpack의 일부로, 백그라운드에서 실행해야 하는 작업을 관리하는 라이브러리입니다.  
앱이 종료되거나 기기가 재부팅되어도 작업을 보장하며, 배터리 최적화 정책을 준수합니다.

### 1-2. 왜 WorkManager를 사용하나요?

**기존 백그라운드 작업의 문제점:**
- `Service`는 Android 8.0(API 26)부터 제한됨
- `AlarmManager`는 Doze Mode에서 동작하지 않을 수 있음
- `JobScheduler`는 API 21 이상에서만 사용 가능

**WorkManager의 장점:**
- **호환성**: API 16부터 지원 (Android 4.1+)
- **배터리 최적화**: Doze Mode, App Standby 정책 준수
- **안정성**: 앱 종료/재부팅 후에도 작업 보장
- **유연성**: 일회성/주기적 작업 모두 지원

### 1-3. WorkManager vs 다른 백그라운드 작업

| 방식 | API 레벨 | 배터리 최적화 | 앱 종료 후 실행 | 사용 시기 |
|------|----------|---------------|-----------------|-----------|
| Service | 모든 버전 | 제한적 | 보장 안됨 | 실시간 작업 |
| JobScheduler | API 21+ | 지원 | 보장됨 | 백그라운드 작업 |
| AlarmManager | 모든 버전 | 제한적 | 보장됨 | 정확한 시간 작업 |
| **WorkManager** | **API 16+** | **완벽 지원** | **보장됨** | **모든 백그라운드 작업** |

---

## 2. WorkManager 핵심 구성요소

### 2-1. WorkRequest

작업을 정의하는 클래스로, **OneTimeWorkRequest**와 **PeriodicWorkRequest** 두 가지가 있습니다.

```kotlin
// 일회성 작업
val oneTimeWorkRequest = OneTimeWorkRequest.Builder(MyWorker::class.java)
    .setInputData(workDataOf("key" to "value"))
    .build()

// 주기적 작업 (최소 15분 간격)
val periodicWorkRequest = PeriodicWorkRequest.Builder(
    MyWorker::class.java, 
    15, TimeUnit.MINUTES
).build()
```

### 2-2. Worker

실제 작업을 수행하는 클래스입니다.

```kotlin
class MyWorker(
    context: Context,
    workerParams: WorkerParameters
) : Worker(context, workerParams) {
    
    override fun doWork(): Result {
        return try {
            // 실제 작업 수행
            performBackgroundTask()
            Result.success()
        } catch (e: Exception) {
            Result.failure()
        }
    }
    
    private fun performBackgroundTask() {
        // 백그라운드 작업 로직
        Log.d("WorkManager", "백그라운드 작업 실행")
    }
}
```

### 2-3. WorkManager

작업을 스케줄링하고 관리하는 클래스입니다.

```kotlin
class MainActivity : AppCompatActivity() {
    
    private fun scheduleWork() {
        val workRequest = OneTimeWorkRequest.Builder(MyWorker::class.java)
            .setInputData(workDataOf("message" to "Hello WorkManager"))
            .build()
        
        WorkManager.getInstance(this).enqueue(workRequest)
    }
}
```

---

## 3. 실제 안드로이드 앱에서의 활용

### 3-1. 의존성 추가

```gradle
// build.gradle (Module: app)
dependencies {
    implementation "androidx.work:work-runtime-ktx:2.9.0"
    
    // Room과 함께 사용할 경우
    implementation "androidx.work:work-runtime-ktx:2.9.0"
    implementation "androidx.room:room-ktx:2.6.1"
}
```

### 3-2. 기본 사용법

```kotlin
// Worker 클래스
class ImageUploadWorker(
    context: Context,
    workerParams: WorkerParameters
) : Worker(context, workerParams) {
    
    override fun doWork(): Result {
        return try {
            // 입력 데이터 받기
            val imagePath = inputData.getString("image_path")
            val userId = inputData.getString("user_id")
            
            // 이미지 업로드 작업
            uploadImageToServer(imagePath, userId)
            
            // 성공 결과 반환
            Result.success()
        } catch (e: Exception) {
            Log.e("ImageUploadWorker", "업로드 실패", e)
            Result.failure()
        }
    }
    
    private fun uploadImageToServer(imagePath: String?, userId: String?) {
        // 실제 업로드 로직
        Thread.sleep(3000) // 업로드 시뮬레이션
        Log.d("ImageUploadWorker", "이미지 업로드 완료: $imagePath")
    }
}
```

```kotlin
// MainActivity에서 사용
class MainActivity : AppCompatActivity() {
    
    private fun uploadImage() {
        val uploadRequest = OneTimeWorkRequest.Builder(ImageUploadWorker::class.java)
            .setInputData(workDataOf(
                "image_path" to "/storage/image.jpg",
                "user_id" to "user123"
            ))
            .build()
        
        WorkManager.getInstance(this).enqueue(uploadRequest)
    }
}
```

### 3-3. 주기적 작업 (Periodic Work)

```kotlin
// Worker 클래스
class DataSyncWorker(
    context: Context,
    workerParams: WorkerParameters
) : Worker(context, workerParams) {
    
    override fun doWork(): Result {
        return try {
            // 서버와 데이터 동기화
            syncDataWithServer()
            Result.success()
        } catch (e: Exception) {
            Log.e("DataSyncWorker", "동기화 실패", e)
            Result.retry()
        }
    }
    
    private fun syncDataWithServer() {
        // 데이터 동기화 로직
        Log.d("DataSyncWorker", "데이터 동기화 실행")
    }
}
```

```kotlin
// MainActivity에서 주기적 작업 스케줄링
class MainActivity : AppCompatActivity() {
    
    private fun schedulePeriodicSync() {
        val syncRequest = PeriodicWorkRequest.Builder(
            DataSyncWorker::class.java,
            1, TimeUnit.HOURS // 1시간마다 실행
        )
            .setInitialDelay(30, TimeUnit.MINUTES) // 30분 후 첫 실행
            .build()
        
        WorkManager.getInstance(this)
            .enqueueUniquePeriodicWork(
                "data_sync",
                ExistingPeriodicWorkPolicy.KEEP,
                syncRequest
            )
    }
}
```

### 3-4. 작업 체인 (Work Chain)

```kotlin
class ImageProcessingWorker(
    context: Context,
    workerParams: WorkerParameters
) : Worker(context, workerParams) {
    
    override fun doWork(): Result {
        return try {
            val imagePath = inputData.getString("image_path")
            
            // 이미지 압축
            val compressedPath = compressImage(imagePath)
            
            // 결과를 다음 작업으로 전달
            val outputData = workDataOf("compressed_path" to compressedPath)
            Result.success(outputData)
        } catch (e: Exception) {
            Result.failure()
        }
    }
    
    private fun compressImage(imagePath: String?): String {
        // 이미지 압축 로직
        return "/compressed/$imagePath"
    }
}

class ImageUploadWorker(
    context: Context,
    workerParams: WorkerParameters
) : Worker(context, workerParams) {
    
    override fun doWork(): Result {
        return try {
            val compressedPath = inputData.getString("compressed_path")
            
            // 압축된 이미지 업로드
            uploadImage(compressedPath)
            Result.success()
        } catch (e: Exception) {
            Result.failure()
        }
    }
    
    private fun uploadImage(imagePath: String?) {
        // 업로드 로직
        Log.d("ImageUploadWorker", "업로드 완료: $imagePath")
    }
}

// 작업 체인 실행
class MainActivity : AppCompatActivity() {
    
    private fun processAndUploadImage() {
        val compressRequest = OneTimeWorkRequest.Builder(ImageProcessingWorker::class.java)
            .setInputData(workDataOf("image_path" to "/original/image.jpg"))
            .build()
        
        val uploadRequest = OneTimeWorkRequest.Builder(ImageUploadWorker::class.java)
            .build()
        
        WorkManager.getInstance(this)
            .beginWith(compressRequest)
            .then(uploadRequest)
            .enqueue()
    }
}
```

### 3-5. 작업 제약 조건 (Constraints)

```kotlin
// MainActivity에서 제약 조건과 함께 작업 스케줄링
class MainActivity : AppCompatActivity() {
    
    private fun scheduleSyncWithConstraints() {
        // 제약 조건 설정
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED) // 네트워크 연결 필요
            .setRequiresCharging(true) // 충전 중일 때만 실행
            .setRequiresBatteryNotLow(true) // 배터리 부족하지 않을 때만 실행
            .build()
        
        val syncRequest = OneTimeWorkRequest.Builder(DataSyncWorker::class.java)
            .setConstraints(constraints)
            .build()
        
        WorkManager.getInstance(this).enqueue(syncRequest)
    }
}
```

### 3-6. 작업 상태 모니터링

```kotlin
// MainActivity에서 작업 상태 모니터링
class MainActivity : AppCompatActivity() {
    
    private fun monitorWorkStatus() {
        val workRequest = OneTimeWorkRequest.Builder(MyWorker::class.java)
            .build()
        
        // 작업 상태 관찰
        WorkManager.getInstance(this)
            .getWorkInfoByIdLiveData(workRequest.id)
            .observe(this) { workInfo ->
                when (workInfo?.state) {
                    WorkInfo.State.ENQUEUED -> {
                        Log.d("WorkManager", "작업 대기 중")
                    }
                    WorkInfo.State.RUNNING -> {
                        Log.d("WorkManager", "작업 실행 중")
                    }
                    WorkInfo.State.SUCCEEDED -> {
                        Log.d("WorkManager", "작업 성공")
                        val outputData = workInfo.outputData
                        // 결과 데이터 처리
                    }
                    WorkInfo.State.FAILED -> {
                        Log.d("WorkManager", "작업 실패")
                    }
                    WorkInfo.State.CANCELLED -> {
                        Log.d("WorkManager", "작업 취소됨")
                    }
                    null -> {
                        Log.d("WorkManager", "작업 정보 없음")
                    }
                }
            }
        
        WorkManager.getInstance(this).enqueue(workRequest)
    }
}
```

---

## 4. 면접에서 나올 수 있는 질문들

**Q: WorkManager란 무엇인가요?**
**A:** WorkManager는 Android Jetpack의 일부로, 백그라운드 작업을 안전하고 효율적으로 실행하기 위한 라이브러리입니다. 앱이 종료되거나 기기가 재부팅되어도 작업을 보장하며, 배터리 최적화 정책을 준수합니다.

**Q: WorkManager와 Service의 차이점은?**
**A:** Service는 실시간 작업에 적합하지만 Android 8.0부터 제한이 있고, 앱 종료 시 보장되지 않습니다. WorkManager는 백그라운드 작업에 최적화되어 있으며, 앱 종료/재부팅 후에도 작업을 보장합니다.

**Q: WorkManager의 작업 재시도 정책에 대해 설명해주세요.**
**A:** WorkManager는 `Result.retry()`를 통해 작업 재시도를 지원하며, 지수 백오프(Exponential Backoff)와 선형 백오프(Linear Backoff) 정책을 제공합니다. 네트워크 오류 등 일시적인 문제에 대해 자동으로 재시도합니다.

**Q: WorkManager의 제약 조건(Constraints)에 대해 설명해주세요.**
**A:** Constraints는 작업 실행 조건을 정의합니다. 네트워크 연결 상태, 충전 상태, 배터리 레벨, 저장 공간 등을 조건으로 설정할 수 있어 배터리 최적화와 사용자 경험을 향상시킵니다.

**Q: WorkManager 2.9.0에서 추가된 WorkQuery 기능에 대해 설명해주세요.**
**A:** WorkQuery는 여러 조건을 조합하여 WorkInfo를 쿼리할 수 있는 기능입니다. 작업 상태, 태그, 고유 이름 등을 조건으로 설정하여 특정 작업들을 효율적으로 조회할 수 있습니다.
