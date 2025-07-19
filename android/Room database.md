# Room database

> RoomDatabase는 Android 에서 제공하는 로컬 데이터를 저장하기 위한 라이브러리입니다.  
> 이전에 사용하던 SQLite 데이터베이스를 추상화하여 더 쉽고 안전하게 사용할 수 있도록 도와주는 Jetpack 컴포넌트입니다.  
> 이 글에서는 RoomDB가 나온 배경, SQLite와의 차이점, 그리고 각각의 장단점을 비교 분석하여  
> 언제 어떤 데이터베이스 접근 방식을 선택해야 하는지에 대해 정리합니다.  

## 1. RoomDB가 나온 이유

### 1. SQLite의 한계점
- **복잡한 SQL 쿼리 작성**: 개발자가 직접 SQL 쿼리를 작성해야 함
- **타입 안전성 부족**: 컴파일 타임에 SQL 오류를 발견하기 어려움
- **보일러플레이트 코드**: 반복적인 CRUD 작업 코드가 많음
- **스키마 변경 관리**: 데이터베이스 스키마 변경 시 마이그레이션 처리가 복잡

### 2. 개발 생산성 향상 필요
- **코드 생성**: 컴파일 타임에 SQL 쿼리를 자동 생성
- **타입 안전성**: Kotlin/Java 타입 시스템을 활용한 안전한 데이터베이스 접근
- **간편한 API**: 복잡한 SQL 대신 간단한 메서드 호출로 데이터베이스 조작

## 2. SQLite vs RoomDB 차이점

### 1. **개발 편의성**

| 구분 | SQLite | RoomDB |
|------|--------|--------|
| 쿼리 작성 | 개발자가 직접 SQL 작성 | 어노테이션 기반 간단한 API |
| 오류 검증 | 런타임에만 발견 | 컴파일 타임에 검증 |
| 코드량 | 반복적인 CRUD 코드 많음 | 코드 생성으로 자동화 |

### 2. **타입 안전성**

**SQLite**:
```kotlin
// 타입 안전성 없음
val cursor = db.rawQuery("SELECT * FROM users WHERE id = ?", arrayOf(userId))
val name = cursor.getString(cursor.getColumnIndex("name")) // 런타임 오류 가능
```

**RoomDB**:
```kotlin
// 타입 안전성 보장
@Query("SELECT * FROM users WHERE id = :userId")
suspend fun getUserById(userId: String): User? // 컴파일 타임 검증
```

### 3. **스키마 관리**

| 구분 | SQLite | RoomDB |
|------|--------|--------|
| 스키마 변경 | 수동 처리 | 자동 마이그레이션 지원 |
| 버전 관리 | 복잡한 로직 필요 | `@Database` 어노테이션으로 간단 관리 |
| 검증 | 런타임에만 확인 | 컴파일 타임에 검증 |

### 4. **성능**

| 구분 | SQLite | RoomDB |
|------|--------|--------|
| 기본 성능 | 직접 SQL 최적화 가능 | SQLite 기반으로 동일한 성능 |
| 메모리 사용량 | 적음 | 약간의 오버헤드 (무시할 수준) |
| 복잡한 쿼리 | 더 유연함 | 제한적일 수 있음 |


### 5. **유지보수성**

| 구분 | SQLite | RoomDB |
|------|--------|--------|
| 리팩토링 | 문자열 기반으로 어려움 | 타입 시스템 활용으로 안전 |
| 스키마 변경 | 수동 처리 필요 | 자동 마이그레이션 |
| 테스트 | 복잡한 설정 필요 | 간편한 테스트 작성 |

## 3. 장단점 비교

| 구분 | RoomDB | SQLite |
|------|--------|--------|
| **장점** | • 타입 안전성 보장<br>• 컴파일 타임 오류 검증<br>• 간편한 API 제공<br>• 자동 코드 생성<br>• Google 공식 지원<br>• Clean Architecture와 잘 어울림 | • 직접적인 제어 가능<br>• 복잡한 쿼리 유연성<br>• 메모리 효율성<br>• 다양한 플랫폼 지원 |
| **단점** | • 추상화 레이어로 약간의 오버헤드<br>• 복잡한 쿼리에서 제한적<br>• SQLite만 지원 (다른 DB 불가) | • 보일러플레이트 코드 많음<br>• 런타임 오류 위험<br>• 타입 안전성 부족<br>• 스키마 관리 복잡 |

## 참고자료

- [Android Room Persistence Library](https://developer.android.com/training/data-storage/room)
- [SQLite Documentation](https://www.sqlite.org/docs.html)
- [Android Jetpack Components](https://developer.android.com/jetpack)
- [Clean Architecture in Android](https://developer.android.com/topic/architecture)
- [Kotlin Coroutines with Room](https://developer.android.com/kotlin/coroutines) 
