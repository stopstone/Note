# HashMap과 TreeMap의 차이

> Java의 Map 인터페이스를 구현한 대표적인 클래스인 `HashMap`과 `TreeMap`은 각각의 용도와 성능 특성이 다릅니다.  
> 면접에서도 종종 나오는 질문 중 하나이기도 합니다.
> 본 글에서는 이 둘의 차이점과 사용 시 고려할 점을 정리합니다.  

## 1. HashMap과 TreeMap이란?

### 1-1. HashMap

* 해시 테이블 기반으로 구현된 Map
* 키-값 쌍을 저장하며, 빠른 검색/삽입/삭제가 가능
* 키의 순서를 유지하지 않음 (무작위)
* `null` 키 1개 허용, `null` 값 다수 허용

### 1-2. TreeMap

* 레드-블랙 트리 기반으로 구현된 Map
* 키의 정렬 순서를 유지 (기본: 오름차순, 또는 사용자 지정 Comparator)
* 키에 `null` 허용하지 않음
* `null` 값 다수 허용

## 2. 주요 차이점

| 항목       | HashMap | TreeMap                 |
| -------- | ------- | ----------------------- |
| 내부 구조    | 해시 테이블  | 레드-블랙 트리                |
| 정렬 여부    | 정렬되지 않음 | 키의 정렬 순서 유지             |
| 성능       | 평균 O(1) | O(log n)                |
| `null` 키 | 1개 허용   | 허용 안 함                  |
| `null` 값 | 여러 개 허용 | 여러 개 허용                 |
| 인터페이스    | Map     | NavigableMap, SortedMap |

## 3. 언제 어떤 걸 써야 할까요?

### HashMap 사용 시점

* 키의 순서가 중요하지 않고, 빠른 접근 속도가 필요한 경우
* 예: 캐시, 빈도수 계산, 단순한 키-값 저장소 등

### TreeMap 사용 시점

* 키의 정렬이 필요한 경우
* 범위 기반 조회, 정렬된 순회가 필요한 경우
* 예: 랭킹 시스템, 날짜 기반 로그 정렬, 범위 필터링 등

## 4. 성능 비교

* HashMap은 해시 기반 구조로 평균적으로 매우 빠른 검색/삽입/삭제(O(1))가 가능하나,  
해시 충돌이 많을 경우 성능이 저하될 수 있습니다.
* TreeMap은 모든 연산이 O(log n)의 시간 복잡도를 가지며,  
항상 정렬된 상태를 유지하므로 정렬/범위 검색에는 유리합니다.

## 5. 예제 코드

```java
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("banana", 2);
hashMap.put("apple", 5);
hashMap.put("orange", 3);
System.out.println("HashMap: " + hashMap);

Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("banana", 2);
treeMap.put("apple", 5);
treeMap.put("orange", 3);
System.out.println("TreeMap: " + treeMap);
```

출력 예시:

```
HashMap: {orange=3, banana=2, apple=5} // 순서 보장 없음
TreeMap: {apple=5, banana=2, orange=3} // 정렬됨
```
