# Android permissions 종류

> 안드로이드 앱을 개발할 때 특정 기능을 사용하려면 `AndroidManifest.xml`에 `<uses-permission>` 태그를 추가해야 합니다.  
> 권한은 사용자의 개인정보와 시스템 리소스를 보호하기 위한 안드로이드의 보안 메커니즘입니다.  
> 권한을 요청하지 않으면 해당 기능을 사용할 수 없고, 런타임 권한의 경우 사용자가 거부하면 앱이 정상적으로 동작하지 않을 수 있습니다.  
> 
> 이 문서에서는 안드로이드 권한의 종류와 각 권한이 언제 필요한지, 권한을 요청하지 않으면 어떻게 되는지에 대해 정리하겠습니다.

---

## 1. 권한의 종류

안드로이드 권한은 크게 **일반 권한(Normal Permissions)**과 **위험 권한(Dangerous Permissions)**으로 나뉩니다.

### 일반 권한 (Normal Permissions)
- 사용자의 개인정보에 직접적으로 접근하지 않는 권한
- 앱 설치 시 자동으로 승인됨
- 별도의 런타임 권한 요청 불필요

### 위험 권한 (Dangerous Permissions)
- 사용자의 개인정보나 기기 기능에 직접적으로 접근하는 권한
- Android 6.0(API 23) 이상에서는 런타임에 사용자에게 명시적으로 요청해야 함
- 사용자가 거부할 수 있음

### 권한 그룹의 특징
- 같은 그룹의 권한 중 하나를 승인하면 나머지도 자동으로 승인됨
- 예: `READ_CONTACTS`를 승인하면 `WRITE_CONTACTS`도 자동으로 승인
- 권한 그룹별로 한 번에 요청할 수 있어 사용자 경험이 개선됨

---

## 2. 주요 권한 종류와 사용 시점

### 네트워크 관련 권한

#### `INTERNET`
- **언제 사용**: 웹 API 호출, 이미지 다운로드, 실시간 통신
- **권한 없을 때**: 네트워크 연결 불가, 앱이 인터넷에 접근할 수 없음

#### `ACCESS_NETWORK_STATE`
- **언제 사용**: 네트워크 상태 확인, 온라인/오프라인 상태 체크
- **권한 없을 때**: 네트워크 상태 정보를 얻을 수 없음

### 저장소 관련 권한

#### `READ_EXTERNAL_STORAGE`
- **언제 사용**: 갤러리에서 이미지 선택, 파일 읽기
- **권한 없을 때**: 외부 저장소의 파일에 접근할 수 없음
- **Android 11+**: Scoped Storage로 인해 MediaStore API 사용 시 권한 불필요

#### `WRITE_EXTERNAL_STORAGE`
- **언제 사용**: 파일 저장, 이미지 다운로드, 캐시 저장
- **권한 없을 때**: 외부 저장소에 파일을 저장할 수 없음
- **Android 11+**: 앱 전용 디렉토리는 권한 없이 사용 가능

### 카메라 관련 권한

#### `CAMERA`
- **언제 사용**: 사진 촬영, QR 코드 스캔, 비디오 녹화
- **권한 없을 때**: 카메라 기능 사용 불가, 앱 크래시 발생 가능

### 위치 관련 권한

#### `ACCESS_FINE_LOCATION`
- **언제 사용**: 정확한 GPS 위치 정보 필요 (지도, 네비게이션)
- **권한 없을 때**: 정확한 위치 정보를 얻을 수 없음

#### `ACCESS_COARSE_LOCATION`
- **언제 사용**: 대략적인 위치 정보만 필요 (날씨 앱, 근처 매장 검색)
- **권한 없을 때**: 위치 기반 서비스 사용 불가

### 연락처 관련 권한

#### `READ_CONTACTS`
- **언제 사용**: 연락처 목록 조회, 친구 찾기
- **권한 없을 때**: 연락처 정보에 접근할 수 없음

#### `WRITE_CONTACTS`
- **언제 사용**: 새 연락처 추가, 기존 연락처 수정
- **권한 없을 때**: 연락처 정보를 수정할 수 없음

### 전화 관련 권한

#### `READ_PHONE_STATE`
- **언제 사용**: 전화번호 조회, IMEI 확인
- **권한 없을 때**: 기기 정보에 접근할 수 없음

#### `CALL_PHONE`
- **언제 사용**: 앱에서 직접 전화 걸기
- **권한 없을 때**: 전화 기능 사용 불가

### 마이크 관련 권한

#### `RECORD_AUDIO`
- **언제 사용**: 음성 녹음, 음성 인식, 음성 메시지
- **권한 없을 때**: 마이크 사용 불가, 녹음 기능 동작하지 않음

### 알림 관련 권한

#### `POST_NOTIFICATIONS` (Android 13+)
- **언제 사용**: 푸시 알림, 로컬 알림 발송
- **권한 없을 때**: 알림이 표시되지 않음

### 시스템 관련 권한

#### `SYSTEM_ALERT_WINDOW`
- **언제 사용**: 다른 앱 위에 오버레이 표시 (채팅 헤드, 플로팅 버튼)
- **권한 없을 때**: 오버레이 창을 표시할 수 없음

#### `REQUEST_INSTALL_PACKAGES`
- **언제 사용**: APK 파일 설치 (앱 업데이트, 외부 앱 설치)
- **권한 없을 때**: 앱 설치 기능 사용 불가

---

## 3. 권한 요청 방법

### AndroidManifest.xml에 권한 선언
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

### 런타임 권한 요청 방법

#### 1. 전통적인 방식 (Android 6.0+)
```kotlin
class MainActivity : AppCompatActivity() {
    private val CAMERA_PERMISSION_REQUEST = 100
    
    private fun checkCameraPermission() {
        when {
            ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.CAMERA
            ) == PackageManager.PERMISSION_GRANTED -> {
                // 권한이 이미 승인됨
                openCamera()
            }
            shouldShowRequestPermissionRationale(Manifest.permission.CAMERA) -> {
                // 권한 거부 이력이 있음 - 사용자에게 설명 필요
                showPermissionRationaleDialog()
            }
            else -> {
                // 권한 요청
                requestPermissions(
                    arrayOf(Manifest.permission.CAMERA),
                    CAMERA_PERMISSION_REQUEST
                )
            }
        }
    }
    
    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        
        when (requestCode) {
            CAMERA_PERMISSION_REQUEST -> {
                if (grantResults.isNotEmpty() && 
                    grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // 권한 승인됨
                    openCamera()
                } else {
                    // 권한 거부됨
                    showPermissionDeniedMessage()
                }
            }
        }
    }
}
```

#### 2. 최신 방식 - ActivityResultLauncher (권장)
```kotlin
class MainActivity : AppCompatActivity() {
    private val cameraPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            openCamera()
        } else {
            showPermissionDeniedMessage()
        }
    }
    
    private val multiplePermissionsLauncher = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        when {
            permissions.getOrDefault(Manifest.permission.CAMERA, false) -> {
                openCamera()
            }
            permissions.getOrDefault(Manifest.permission.READ_EXTERNAL_STORAGE, false) -> {
                openGallery()
            }
            else -> {
                showPermissionDeniedMessage()
            }
        }
    }
    
    private fun requestCameraPermission() {
        cameraPermissionLauncher.launch(Manifest.permission.CAMERA)
    }
    
    private fun requestMultiplePermissions() {
        multiplePermissionsLauncher.launch(
            arrayOf(
                Manifest.permission.CAMERA,
                Manifest.permission.READ_EXTERNAL_STORAGE
            )
        )
    }
}
```

---

## 4. 권한 거부 시 대응 방법

### 1. 대체 기능 제공
```kotlin
private fun handleCameraPermissionDenied() {
    // 카메라 대신 갤러리에서 이미지 선택
    val intent = Intent(Intent.ACTION_PICK)
    intent.type = "image/*"
    startActivityForResult(intent, GALLERY_REQUEST_CODE)
}
```

### 2. 설정 화면으로 안내
```kotlin
private fun showPermissionSettingsDialog() {
    AlertDialog.Builder(this)
        .setTitle("권한 필요")
        .setMessage("이 기능을 사용하려면 설정에서 권한을 허용해주세요.")
        .setPositiveButton("설정으로 이동") { _, _ ->
            val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
            intent.data = Uri.fromParts("package", packageName, null)
            startActivity(intent)
        }
        .setNegativeButton("취소", null)
        .show()
}
```

---

## 5. 권한 관련 모범 사례

### 1. 필요한 권한만 요청
- 사용자 경험을 위해 꼭 필요한 권한만 요청
- 불필요한 권한은 앱 심사에서 거부될 수 있음

### 2. 권한 요청 시점 최적화
- 앱 시작 시 모든 권한을 요청하지 말고, 해당 기능 사용 시점에 요청
- 사용자가 기능을 사용하려고 할 때 권한 요청

### 3. 권한 거부 시 대안 제공
- 권한이 거부되어도 앱의 핵심 기능은 동작할 수 있도록 설계
- 대체 방법을 제공하여 사용자 경험 개선

### 4. 권한 설명 제공
- 권한 요청 전에 왜 해당 권한이 필요한지 명확히 설명
- 사용자가 권한의 필요성을 이해할 수 있도록 안내

### 5. 최신 권한 요청 방식 사용
- `ActivityResultLauncher`를 사용하여 더 안전하고 간결한 권한 요청
- 콜백 기반으로 권한 결과 처리 가능
- 여러 권한을 한 번에 요청할 때 더 효율적

---

## 6. Android 11+ Scoped Storage

### Scoped Storage란?
Android 11부터 도입된 새로운 저장소 접근 방식으로, 앱이 자신의 전용 디렉토리와 공유 미디어 파일에만 접근할 수 있도록 제한합니다.

### 주요 변경사항

#### 1. 앱 전용 디렉토리
```kotlin
// 앱 전용 디렉토리는 권한 없이 자유롭게 사용 가능
val appSpecificDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES)
val file = File(appSpecificDir, "my_image.jpg")
```

#### 2. MediaStore API 사용
```kotlin
// READ_EXTERNAL_STORAGE 권한 없이도 MediaStore로 미디어 파일 접근 가능
val projection = arrayOf(
    MediaStore.Images.Media._ID,
    MediaStore.Images.Media.DISPLAY_NAME
)
val selection = "${MediaStore.Images.Media.DATE_ADDED} >= ?"
val selectionArgs = arrayOf(
    DateUtils.addDays(Date(), -7).time.toString()
)
val sortOrder = "${MediaStore.Images.Media.DATE_ADDED} DESC"

contentResolver.query(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    projection,
    selection,
    selectionArgs,
    sortOrder
)?.use { cursor ->
    // 이미지 파일 처리
}
```

#### 3. 권한 요청 최적화
- **Android 11+**: `READ_EXTERNAL_STORAGE` 권한 없이도 MediaStore API 사용 가능
- **Android 10 이하**: 여전히 `READ_EXTERNAL_STORAGE` 권한 필요
- **모든 버전**: `WRITE_EXTERNAL_STORAGE`는 앱 전용 디렉토리 외부 저장 시 필요

---

## 7. 권한 요청 라이브러리

### 권한 요청 라이브러리 사용의 장점
- 복잡한 권한 요청 로직을 간소화
- 권한 거부 시 대응 로직 내장
- 다양한 권한 요청 시나리오 지원

### 주요 라이브러리

#### 1. PermissionsDispatcher
```kotlin
@RuntimePermissions
class MainActivity : AppCompatActivity() {
    
    @NeedsPermission(Manifest.permission.CAMERA)
    fun openCamera() {
        // 카메라 기능 구현
    }
    
    @OnPermissionDenied(Manifest.permission.CAMERA)
    fun onCameraDenied() {
        showPermissionDeniedMessage()
    }
    
    @OnShowRationale(Manifest.permission.CAMERA)
    fun showRationaleForCamera(request: PermissionRequest) {
        showRationaleDialog(request)
    }
}
```

#### 2. Dexter
```kotlin
Dexter.withContext(this)
    .withPermission(Manifest.permission.CAMERA)
    .withListener(object : PermissionListener {
        override fun onPermissionGranted(response: PermissionGrantedResponse) {
            openCamera()
        }
        
        override fun onPermissionDenied(response: PermissionDeniedResponse) {
            showPermissionDeniedMessage()
        }
        
        override fun onPermissionRationaleShouldBeShown(
            permission: PermissionRequest?,
            token: PermissionToken?
        ) {
            token?.continuePermissionRequest()
        }
    }).check()
```

---

## 8. 권한 그룹별 정리

| 권한 그룹 | 주요 권한 | 사용 시점 |
|-----------|-----------|-----------|
| **네트워크** | `INTERNET`, `ACCESS_NETWORK_STATE` | API 호출, 온라인 기능 |
| **저장소** | `READ_EXTERNAL_STORAGE`, `WRITE_EXTERNAL_STORAGE` | 파일 읽기/쓰기 |
| **카메라** | `CAMERA` | 사진 촬영, QR 스캔 |
| **위치** | `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION` | 위치 기반 서비스 |
| **연락처** | `READ_CONTACTS`, `WRITE_CONTACTS` | 연락처 관리 |
| **전화** | `READ_PHONE_STATE`, `CALL_PHONE` | 전화 기능 |
| **마이크** | `RECORD_AUDIO` | 음성 녹음, 음성 인식 |
| **알림** | `POST_NOTIFICATIONS` | 푸시 알림 |

---

## 참고자료

* [Android 공식 문서 - 권한 개요](https://developer.android.com/guide/topics/permissions/overview)
* [Android 공식 문서 - 권한 요청](https://developer.android.com/training/permissions/requesting)
* [Android 공식 문서 - Scoped Storage](https://developer.android.com/about/versions/11/privacy/storage)
* [Android 권한 모범 사례](https://developer.android.com/topic/security/best-practices#permissions)
* [PermissionsDispatcher GitHub](https://github.com/permissions-dispatcher/PermissionsDispatcher)
* [Dexter GitHub](https://github.com/Karumi/Dexter)
