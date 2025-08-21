# Firebase Firestore 필드명 매핑 이슈 해결

> 자동 로그인 상태에서 프로필이 DB에 존재함에도 불구하고 프로필 생성 화면이 계속 나타나는 문제와 해결 과정을 정리한 문서입니다.

---

## 문제 상황

### 증상
- **예상 동작**: 자동 로그인 → 프로필 존재 확인 → 홈 화면 이동
- **실제 동작**: 자동 로그인 → 프로필 없음으로 판단 → 프로필 생성 화면 이동
- **DB 상태**: 로그인 성공, 프로필 데이터 존재

### 관련 로그
```
[AuthRepositoryImpl::hasUserProfile-IoAF18A (AuthRepositoryImpl.kt:238)]사용자 프로필 존재 여부: false
[SplashViewModel::checkUserProfileAndNavigate (SplashViewModel.kt:84)]새 사용자 - 프로필 생성 화면으로 이동
[SplashScreen::invokeSuspend (SplashScreen.kt:46)]스플래시 화면 종료, 프로필 생성 화면으로 이동
[ProfileCreateViewModel::<init> (ProfileCreateViewModel.kt:37)]ProfileCreateViewModel 초기화
```

---

## 원인 분석

### Firebase Firestore 필드명 자동 변환

Firebase Firestore는 Java Bean 규칙을 따라 Boolean 필드명에서 `is` 접두사를 자동으로 제거하는 특성이 있음

**왜 `is` 접두사가 제거되는가?**

1. **Java Bean 명명 규칙**: Firebase는 Java 기반 플랫폼으로, Java Bean의 getter/setter 명명 규칙을 따릅니다. Boolean 필드의 경우:
   - 필드명: `isActive`
   - Getter: `isActive()` 또는 `getActive()`
   - Setter: `setActive(boolean)`
   - 직렬화 시 필드명: `active` (is 접두사 제거)

2. **Jackson/Gson 직렬화 라이브러리 동작**: Firebase가 내부적으로 사용하는 직렬화 라이브러리는 Boolean 필드의 `is` 접두사를 제거하여 JSON 필드명을 생성합니다.

3. **크로스 플랫폼 호환성**: 다양한 언어(JavaScript, Swift, Dart 등)에서 `is` 접두사가 없는 것이 더 자연스럽고 일관된 네이밍을 제공합니다.

4. **데이터베이스 네이밍 컨벤션**: NoSQL 데이터베이스에서는 일반적으로 간결하고 명확한 필드명을 선호하며, `is` 접두사 없이 `active`, `enabled`, `visible` 등의 형태가 더 직관적입니다.

**문제가 된 코드:**
```kotlin
// 엔티티 정의
data class ProfileFirebaseEntity(
    val isActive: Boolean = true,
    val isDefault: Boolean = false,
    // ...
)

// 쿼리 코드
val querySnapshot = firestore.collection(PROFILES_COLLECTION)
    .whereEqualTo("userId", userId)
    .whereEqualTo("isActive", true)  // 실제로는 "active" 필드를 찾음
    .limit(1)
    .get()
    .await()
```

**실제 Firebase에 저장된 데이터:**
```json
{
  "userId": "abc***def",
  "active": null,
  "default": false,
  "profileName": "사용자프로필"
}
```

---

## 디버깅 과정

### 상세 로그 추가

실제 저장된 데이터 구조를 확인하기 위한 디버그 로그 추가:

```kotlin
override suspend fun hasUserProfile(): Result<Boolean> {
    return try {
        val currentUser = firebaseAuth.currentUser
            ?: return Result.failure(Exception("로그인이 필요합니다."))
        
        val userId = currentUser.uid
        logd("프로필 존재 여부 확인 시작 - userId: ${userId.take(3)}***${userId.takeLast(3)}")
        
        // 전체 프로필 확인
        val allProfilesSnapshot = firestore.collection(PROFILES_COLLECTION)
            .whereEqualTo("user_id", userId)
            .get()
            .await()
        
        logd("전체 프로필 문서 개수: ${allProfilesSnapshot.size()}")
        allProfilesSnapshot.documents.forEachIndexed { index, document ->
            val data = document.data
            logd("프로필 $index: user_id=${data?.get("user_id")?.toString()?.let { "${it.take(3)}***${it.takeLast(3)}" }}, is_active=${data?.get("is_active")}, profile_name=${data?.get("profile_name")}")
        }
    }
}
```

### 디버그 결과
```
[AuthRepositoryImpl::hasUserProfile-IoAF18A]전체 프로필 문서 개수: 2
[AuthRepositoryImpl::hasUserProfile-IoAF18A]프로필 0: user_id=abc***def, isActive=null, profileName=돌멩1
[AuthRepositoryImpl::hasUserProfile-IoAF18A]프로필 1: user_id=abc***def, isActive=null, profileName=돌멩2
```

**발견사항:**
- 프로필 데이터는 존재함
- `isActive` 필드가 `null`로 조회됨 → 실제로는 `active` 필드로 저장되어 있음

---

## 해결 방안

### 1. Firebase Entity 수정

`@PropertyName` 어노테이션을 사용하여 snake_case 필드명으로 매핑:

```kotlin
@Keep
data class ProfileFirebaseEntity(
    @DocumentId
    val profileId: String = "",
    
    @get:PropertyName("user_id")
    @set:PropertyName("user_id")
    var userId: String = "",
    
    @get:PropertyName("profile_name")
    @set:PropertyName("profile_name")
    var profileName: String = "",
    
    @get:PropertyName("profile_image_url")
    @set:PropertyName("profile_image_url")
    var profileImageUrl: String = "",
    
    @get:PropertyName("is_default")
    @set:PropertyName("is_default")
    var isDefault: Boolean = false,
    
    @get:PropertyName("is_active")
    @set:PropertyName("is_active")
    var isActive: Boolean = true,
    
    @get:PropertyName("created_at")
    @set:PropertyName("created_at")
    var createdAt: Long = System.currentTimeMillis(),
    
    @get:PropertyName("updated_at")
    @set:PropertyName("updated_at")
    var updatedAt: Long = System.currentTimeMillis(),
)
```

### 2. 쿼리 조건 수정

모든 Firestore 쿼리를 snake_case로 변경:

```kotlin
// Before
val querySnapshot = firestore.collection(PROFILES_COLLECTION)
    .whereEqualTo("userId", userId)
    .whereEqualTo("isActive", true)
    .whereEqualTo("isDefault", true)
    .limit(1)

// After  
val querySnapshot = firestore.collection(PROFILES_COLLECTION)
    .whereEqualTo("user_id", userId)
    .whereEqualTo("is_active", true)
    .whereEqualTo("is_default", true)
    .limit(1)
```

### 3. User Entity 동일하게 수정

```kotlin
@Keep
data class UserFirebaseEntity(
    @DocumentId
    val userId: String = "",
    
    @get:PropertyName("email")
    @set:PropertyName("email") 
    var email: String = "",
    
    @get:PropertyName("display_name")
    @set:PropertyName("display_name")
    var displayName: String = "",
    
    @get:PropertyName("profile_image_url")
    @set:PropertyName("profile_image_url")
    var profileImageUrl: String = "",
    
    @get:PropertyName("provider")
    @set:PropertyName("provider")
    var provider: String = "",
    
    @get:PropertyName("created_at")
    @set:PropertyName("created_at")
    var createdAt: Long = System.currentTimeMillis(),
    
    @get:PropertyName("updated_at")
    @set:PropertyName("updated_at")
    var updatedAt: Long = System.currentTimeMillis(),
    
    @get:PropertyName("is_active")
    @set:PropertyName("is_active")
    var isActive: Boolean = true,
    
    @get:PropertyName("has_profile")
    @set:PropertyName("has_profile")
    var hasProfile: Boolean = false,
)
```

---

## 결과

### 수정 후 Firebase 데이터 구조

**Users Collection:**
```json
{
  "email": "user@example.com",
  "display_name": "사용자명",
  "profile_image_url": "https://firebasestorage.googleapis.com/...",
  "provider": "google",
  "created_at": 1724258400000,
  "updated_at": 1724258400000,
  "is_active": true,
  "has_profile": true
}
```

**Profiles Collection:**
```json
{
  "user_id": "abc***def",
  "profile_name": "돌멩",
  "profile_image_url": "https://firebasestorage.googleapis.com/...",
  "is_default": true,
  "is_active": true,
  "created_at": 1724258400000,
  "updated_at": 1724258400000
}
```

### 수정 후 정상 로그
```
[AuthRepositoryImpl::hasUserProfile-IoAF18A]프로필 존재 여부 확인 시작 - userId: abc***def
[AuthRepositoryImpl::hasUserProfile-IoAF18A]사용자 프로필 쿼리 결과 - 문서 개수: 1, 프로필 존재: true
[SplashViewModel::checkUserProfileAndNavigate]기존 사용자 - 홈 화면으로 이동
```

---

## 배운점

### 1. Firebase Firestore 네이밍 규칙
- Boolean 필드의 `is` 접두사 자동 제거 특성 이해
- 일관된 snake_case 네이밍 컨벤션 적용
- `@PropertyName` 어노테이션을 통한 명시적 필드명 매핑
