# AlarmManager란?

> 안드로이드에서 정확한 시간에 작업을 실행하거나 알림을 보내야 할 때 `AlarmManager`를 사용합니다.  
> 하지만 Android 6.0(API 23) 이후부터는 배터리 최적화와 Doze 모드로 인해 정확한 시간 보장이 어려워졌고,  
> 대안으로 `WorkManager`가 권장되고 있긴 하지만, 차이를 알기 위해 정리해보겠습니다.  

## 1. AlarmManager란?

`AlarmManager`는 안드로이드 시스템에서 제공하는 서비스로, **정확한 시간에 작업을 실행**할 수 있게 해주는 API입니다.

### 주요 특징
- 앱이 종료되어도 시스템 레벨에서 알람을 관리
- 정확한 시간에 작업 실행 가능
- 배터리 최적화와 Doze 모드의 영향을 받음
- Android 6.0 이후부터는 정확성 보장이 어려움

## 2. AlarmManager vs WorkManager 비교

| 항목 | AlarmManager | WorkManager |
|------|-------------|-------------|
| 정확성 | 높음 (Android 6.0 이전) | 중간 (배터리 최적화 고려) |
| 배터리 최적화 | 영향 받음 | 영향 받지 않음 |
| Doze 모드 | 영향 받음 | 영향 받지 않음 |
| 사용 편의성 | 복잡 | 간단 |
| 권장 여부 | 비권장 (Android 6.0+) | 권장 |

## 3. AlarmManager 사용 방법

### 3.1 기본 설정

```kotlin
class AlarmActivity : AppCompatActivity() {
    private lateinit var alarmManager: AlarmManager
    private lateinit var pendingIntent: PendingIntent
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // AlarmManager 인스턴스 생성
        alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
    }
}
```

### 3.2 알람 설정하기

```kotlin
private fun setAlarm() {
    val intent = Intent(this, AlarmReceiver::class.java)
    intent.putExtra("message", "알람이 울렸습니다!")
    
    pendingIntent = PendingIntent.getBroadcast(
        this,
        0,
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    // 5초 후에 알람 실행
    val triggerTime = System.currentTimeMillis() + 5000
    
    alarmManager.setExactAndAllowWhileIdle(
        AlarmManager.RTC_WAKEUP,
        triggerTime,
        pendingIntent
    )
}
```

### 3.3 알람 수신자 (BroadcastReceiver)

```kotlin
class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val message = intent.getStringExtra("message") ?: "알람"
        
        // 알림 표시
        showNotification(context, message)
        
        // 또는 다른 작업 수행
        Log.d("AlarmReceiver", "알람 실행: $message")
    }
    
    private fun showNotification(context: Context, message: String) {
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "alarm_channel",
                "알람 채널",
                NotificationManager.IMPORTANCE_HIGH
            )
            notificationManager.createNotificationChannel(channel)
        }
        
        val notification = NotificationCompat.Builder(context, "alarm_channel")
            .setContentTitle("알람")
            .setContentText(message)
            .setSmallIcon(R.drawable.ic_alarm)
            .setAutoCancel(true)
            .build()
        
        notificationManager.notify(1, notification)
    }
}
```

### 3.4 반복 알람 설정

```kotlin
private fun setRepeatingAlarm() {
    val intent = Intent(this, AlarmReceiver::class.java)
    pendingIntent = PendingIntent.getBroadcast(
        this,
        1,
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    // 매일 오전 9시에 알람 실행
    val calendar = Calendar.getInstance().apply {
        set(Calendar.HOUR_OF_DAY, 9)
        set(Calendar.MINUTE, 0)
        set(Calendar.SECOND, 0)
        set(Calendar.MILLISECOND, 0)
        
        // 오늘 9시가 지났으면 내일 9시로 설정
        if (timeInMillis <= System.currentTimeMillis()) {
            add(Calendar.DAY_OF_MONTH, 1)
        }
    }
    
    alarmManager.setRepeating(
        AlarmManager.RTC_WAKEUP,
        calendar.timeInMillis,
        AlarmManager.INTERVAL_DAY,
        pendingIntent
    )
}
```

## 4. AlarmManager 타입

### 4.1 RTC vs RTC_WAKEUP

| 타입 | 설명 | 사용 시점 |
|------|------|-----------|
| `RTC` | 절대 시간 (UTC) | 기기가 깨어있을 때만 실행 |
| `RTC_WAKEUP` | 절대 시간 (UTC) | 기기가 꺼져있어도 실행 |

### 4.2 ELAPSED_REALTIME vs ELAPSED_REALTIME_WAKEUP

| 타입 | 설명 | 사용 시점 |
|------|------|-----------|
| `ELAPSED_REALTIME` | 부팅 후 경과 시간 | 기기가 깨어있을 때만 실행 |
| `ELAPSED_REALTIME_WAKEUP` | 부팅 후 경과 시간 | 기기가 꺼져있어도 실행 |

## 5. Android 6.0 이후 주의사항

### 5.1 Doze 모드와 App Standby

Android 6.0부터 도입된 배터리 최적화 기능으로 인해:
- **Doze 모드**: 기기가 오랫동안 사용되지 않으면 알람이 지연될 수 있음
- **App Standby**: 앱이 오랫동안 사용되지 않으면 알람이 제한될 수 있음

### 5.2 해결 방법

```kotlin
// setExactAndAllowWhileIdle 사용 (Android 6.0+)
alarmManager.setExactAndAllowWhileIdle(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent
)

// 또는 setAlarmClock 사용 (시계 앱에서 표시됨)
alarmManager.setAlarmClock(
    AlarmManager.AlarmClockInfo(triggerTime, pendingIntent),
    pendingIntent
)
```

## 6. 권한 설정

### 6.1 AndroidManifest.xml

```xml
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
<uses-permission android:name="android.permission.WAKE_LOCK" />

<receiver android:name=".AlarmReceiver" />
```

### 6.2 런타임 권한 확인 (Android 12+)

```kotlin
private fun checkAlarmPermission() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
        if (!alarmManager.canScheduleExactAlarms()) {
            // 정확한 알람 권한 요청
            val intent = Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM)
            startActivity(intent)
        }
    }
}
```

## 7. 현대적인 대안: WorkManager

AlarmManager의 한계를 극복하기 위해 **WorkManager**를 사용하는 것을 권장합니다.

### 7.1 WorkManager의 장점

- 배터리 최적화와 Doze 모드에 영향받지 않음
- 더 간단한 API
- 백그라운드 작업 제한을 자동으로 처리
- 다양한 제약 조건 지원

### 7.2 WorkManager 예시

```kotlin
// 의존성 추가
implementation("androidx.work:work-runtime-ktx:2.8.1")

// Worker 클래스
class AlarmWorker(
    context: Context,
    params: WorkerParameters
) : Worker(context, params) {
    
    override fun doWork(): Result {
        // 알람 작업 수행
        showNotification(applicationContext, "WorkManager 알람")
        return Result.success()
    }
}

// WorkManager로 알람 설정
private fun setWorkManagerAlarm() {
    val alarmRequest = OneTimeWorkRequestBuilder<AlarmWorker>()
        .setInitialDelay(5, TimeUnit.SECONDS)
        .build()
    
    WorkManager.getInstance(this).enqueue(alarmRequest)
}
```
