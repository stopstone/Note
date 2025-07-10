# SQL과 NoSQL의 차이

> 관계형 데이터베이스(SQL)와 비관계형 데이터베이스(NoSQL)는 서로 다른 구조를 가지고 있어,  
> 요구사항에 따라 적절한 선택이 필요합니다.  
> 이번 글에서는 SQL과 NoSQL의 개념적 차이부터 장단점, 선택 기준까지 정리해보겠습니다.  

## 1. SQL이란?

SQL 기반 데이터베이스는 **관계형 데이터베이스(RDBMS)** 라고 부르며,  
데이터를 **테이블**이라는 정형화된 구조에 저장합니다.

* 데이터는 **스키마(schema)** 에 따라 명확한 구조로 저장됩니다.
* 테이블 간 **관계(relation)** 를 설정해 중복을 제거하고, 데이터 정합성을 유지합니다.
* 대표적인 SQL DB: **MySQL, PostgreSQL, Oracle, SQLite** 등

```sql
-- 예시: 사용자 정보 테이블
CREATE TABLE Users (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);
```

## 2. NoSQL이란?

NoSQL은 **비관계형 데이터베이스**로, 구조가 유연하고 다양한 형태의 데이터를 저장할 수 있습니다.  
대표적으로 **문서(document) 지향형**이 많이 사용되며, JSON과 유사한 형태의 데이터를 저장합니다.  

* **스키마가 없음**, 다양한 구조의 데이터를 하나의 컬렉션에 저장 가능
* **관계 없이** 하나의 문서에 필요한 정보를 모두 포함
* 대표적인 NoSQL DB: **MongoDB, Firebase Firestore** 등

```json
// 예시: 사용자 정보 문서 (MongoDB 기준)
{
  "id": 1,
  "name": "짱구",
  "email": "wltjr6432@gmail.com",
  "orders": [
    { "product": "점보 초코비", "quantity": 2 }
  ]
}
```

## 3. SQL vs NoSQL 핵심 비교

| 항목      | SQL (RDBMS)          | NoSQL (비관계형 DB)      |
| ------- | -------------------- | -------------------- |
| 구조      | 정형화된 테이블, 고정된 스키마    | 유연한 문서 구조, 스키마 없음    |
| 확장성     | 수직적 확장 (서버 스펙 업)     | 수평적 확장 (서버 분산)       |
| 관계      | JOIN 등으로 테이블 간 관계 설정 | JOIN 없음, 데이터 중복 허용   |
| 유연성     | 낮음 (스키마 변경 어려움)      | 높음 (구조 변경 자유로움)      |
| 성능      | 복잡한 쿼리 처리에 강함        | 읽기/쓰기 속도 빠름, 대용량에 유리 |
| 데이터 정합성 | 강함 (무결성 보장)          | 약함 (중복 데이터 수동 관리 필요) |

## 4. 언제 SQL을 쓸까?

* **데이터 구조가 명확하고, 관계형 설계가 필요한 경우**
* **데이터 정합성과 무결성이 중요한 경우**
* **빈번한 데이터 변경 및 정규화된 설계가 필요한 경우**
* 예: 금융 시스템, 전자상거래 플랫폼의 결제 시스템, 사용자 인증 DB 등

## 5. 언제 NoSQL을 쓸까?

* **데이터 구조가 유동적이거나 자주 확장/변경되는 경우**
* **빠른 읽기/쓰기 속도가 필요한 경우**
* **수평적 확장으로 대규모 트래픽을 감당해야 하는 경우**
* 예: 소셜미디어 피드, IoT 로그 저장, 실시간 채팅 시스템 등

## 6. 확장성: 수직 vs 수평

* **수직 확장 (Vertical Scaling)**: 더 좋은 CPU, RAM 등 하드웨어 업그레이드
* **수평 확장 (Horizontal Scaling)**: 서버를 여러 대로 분산 구성해 부하 분산
* 대부분의 SQL DB는 수직 확장에 초점을 두며, NoSQL은 수평 확장에 강점을 가집니다


## 참고자료

* [MongoDB 공식 문서](https://www.mongodb.com/nosql-explained)
* [Firebase Realtime Database](https://firebase.google.com/docs/database)
* [MySQL Docs](https://dev.mysql.com/doc/)
* [NoSQL vs SQL 개념 정리](https://www.geeksforgeeks.org/sql-vs-nosql/)

