# DAO

> DAO(Data Access Object)는 데이터베이스에 데이터를 조회하거나 조작하는 기능들을 전담하는 객체입니다.

---

## 1. 기본 개념

**DAO(Data Access Object)** 는 데이터베이스에 접근하는 객체로, 데이터를 조회하거나 조작하는 기능들을 전담합니다.

### 1-1. DAO의 역할

DAO는 데이터베이스 작업을 추상화하여 비즈니스 로직과 데이터 접근 로직을 분리하는 역할을 합니다.

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Insert
    suspend fun insertUser(user: User)
    
    @Update
    suspend fun updateUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
}
```

| 역할 | 설명 | 예시 |
|------|------|------|
| **조회(Read)** | 데이터베이스에서 데이터를 가져옴 | `@Query("SELECT * FROM users")` |
| **생성(Create)** | 새로운 데이터를 데이터베이스에 저장 | `@Insert` |
| **수정(Update)** | 기존 데이터를 수정 | `@Update` |
| **삭제(Delete)** | 데이터를 데이터베이스에서 제거 | `@Delete` |

---

## 2. DAO의 장점

### 2-1. 관심사의 분리
```kotlin
// DAO 없이 직접 데이터베이스 접근
class UserRepository {
    private val database = // 데이터베이스 인스턴스
    
    fun getUsers(): List<User> {
        // 직접 SQL 쿼리 작성
        // 데이터베이스 연결 관리
        // 결과 매핑
    }
}

// DAO 사용
class UserRepository(private val userDao: UserDao) {
    fun getUsers(): Flow<List<User>> = userDao.getAllUsers()
}
```

### 2-2. 테스트 용이성
```kotlin
// DAO를 Mock으로 쉽게 대체 가능
@RunWith(MockitoJUnitRunner::class)
class UserRepositoryTest {
    @Mock
    private lateinit var userDao: UserDao
    
    @InjectMocks
    private lateinit var userRepository: UserRepository
    
    @Test
    fun `사용자 목록 조회 테스트`() = runTest {
        // Given
        val expectedUsers = listOf(User(1, "테스트", "test@example.com"))
        whenever(userDao.getAllUsers()).thenReturn(flowOf(expectedUsers))
        
        // When
        val actualUsers = userRepository.getUsers().first()
        
        // Then
        assertEquals(expectedUsers, actualUsers)
    }
}
```

### 2-3. 코드 재사용성
```kotlin
// 여러 Repository에서 동일한 DAO 사용 가능
class UserRepository(private val userDao: UserDao) {
    fun getUsers() = userDao.getAllUsers()
}

class PostRepository(private val userDao: UserDao) {
    fun getUserPosts(userId: Long) = userDao.getUserPosts(userId)
}
```

---

## 3. DAO 설계 원칙

### 3-1. 단일 책임 원칙
```kotlin
// 좋은 예: 사용자 관련 작업만 담당
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Insert
    suspend fun insertUser(user: User)
    
    @Update
    suspend fun updateUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
}

// 나쁜 예: 여러 도메인 작업을 혼재
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Query("SELECT * FROM posts") // 포스트 관련 작업이 섞임
    fun getAllPosts(): Flow<List<Post>>
}
```

### 3-2. 명확한 네이밍
```kotlin
@Dao
interface UserDao {
    // 명확한 함수명 사용
    @Query("SELECT * FROM users WHERE email = :email LIMIT 1")
    suspend fun findUserByEmail(email: String): User?
    
    @Query("SELECT * FROM users WHERE name LIKE '%' || :searchQuery || '%'")
    fun searchUsersByName(searchQuery: String): Flow<List<User>>
    
    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int
}
```

### 3-3. 적절한 반환 타입
```kotlin
@Dao
interface UserDao {
    // 단일 객체 조회
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: Long): User?
    
    // 목록 조회
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    // 개수 조회
    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int
    
    // 존재 여부 확인
    @Query("SELECT EXISTS(SELECT 1 FROM users WHERE email = :email)")
    suspend fun isUserExists(email: String): Boolean
}
```

---

## 4. DAO 패턴의 장단점

### 4-1. 장점

**1. 관심사 분리**
- 비즈니스 로직과 데이터 접근 로직 분리
- 코드의 가독성과 유지보수성 향상

**2. 테스트 용이성**
- Mock 객체로 쉽게 대체 가능
- 단위 테스트 작성이 쉬움

**3. 코드 재사용성**
- 여러 Repository에서 동일한 DAO 사용
- 중복 코드 제거

**4. 데이터베이스 독립성**
- 데이터베이스 변경 시 DAO만 수정
- 비즈니스 로직에 영향 없음

### 4-2. 단점

**1. 복잡성 증가**
- 작은 프로젝트에서는 과도한 추상화
- 추가적인 인터페이스와 클래스 필요

**2. 성능 오버헤드**
- 추가적인 레이어로 인한 약간의 성능 저하
- 메모리 사용량 증가

**3. 학습 곡선**
- 패턴 이해와 적용에 시간 필요
- 초기 개발 시간 증가

---

## 5. DAO vs Repository

### 5-1. DAO
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Insert
    suspend fun insertUser(user: User)
}
```

**특징:**
- 데이터베이스 작업만 담당
- SQL 쿼리와 매핑에 집중
- 단순한 CRUD 작업

### 5-2. Repository
```kotlin
class UserRepository @Inject constructor(
    private val userDao: UserDao,
    private val userApi: UserApi
) {
    fun getUsers(): Flow<List<User>> = userDao.getAllUsers()
    
    suspend fun refreshUsers() {
        val remoteUsers = userApi.getUsers()
        userDao.insertAllUsers(remoteUsers)
    }
}
```

**특징:**
- 여러 데이터 소스 관리
- 비즈니스 로직 포함
- 데이터 변환 및 가공

---
