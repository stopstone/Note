# 이진 탐색(Binary Search)

> 업다운 게임을 잘 하는 방법은 범위를 절반씩 줄여나가는 것입니다. 1부터 100까지의 숫자 중 하나를 맞추는 게임에서  
> 50을 먼저 외치고, UP이면 75, DOWN이면 25를 외치는 식으로 말이죠.  
> 이렇게 범위를 절반씩 줄여나가는 탐색 방법이 바로 **이진 탐색**입니다

---

## 1. 이진 탐색이란?

이진 탐색(Binary Search)은 **정렬된 배열**에서 특정 값을 찾는 알고리즘입니다.  
탐색 범위를 절반씩 줄여나가며 원하는 값을 찾는 방식으로, 매우 효율적인 탐색 방법입니다.  
여기서 중요한 것은 정렬이 되어야 한다는 것입니다..  

### 이진 탐색의 동작 원리

1. 배열의 중간 값을 확인합니다.
2. 찾는 값이 중간 값보다 작으면 왼쪽 절반을 탐색합니다.
3. 찾는 값이 중간 값보다 크면 오른쪽 절반을 탐색합니다.
4. 값을 찾거나 탐색 범위가 없어질 때까지 반복합니다.

---

## 2. 이진 탐색의 장단점

### 장점
- **빠른 탐색 속도**: 시간 복잡도 O(log n)으로 매우 효율적
- **메모리 효율성**: 추가 메모리 공간이 거의 필요하지 않음
- **구현이 비교적 간단**: 재귀 또는 반복문으로 쉽게 구현 가능

### 단점
- **정렬된 데이터 필요**: 배열이 정렬되어 있어야만 사용 가능
- **삽입/삭제 비용**: 데이터 변경 시 정렬 상태 유지 필요
- **랜덤 액세스 필요**: 연결 리스트 등에서는 효율적이지 않음

---

## 3. 언제 사용할까?

### 이진 탐색을 사용하는 경우
- **정렬된 배열에서 특정 값 찾기**
- **범위 검색**: 특정 값 이상/이하의 첫 번째 위치 찾기
- **최적화 문제**: 특정 조건을 만족하는 최소/최대값 찾기
- **매개변수 탐색**: 조건을 만족하는 값의 범위 탐색
---

## 4. 구현 코드

### 4-1. 반복문을 이용한 구현

```java
public class BinarySearch {
    
    /**
     * 이진 탐색 - 반복문 버전
     * @param arr 정렬된 배열
     * @param target 찾으려는 값
     * @return 찾은 인덱스 (없으면 -1)
     */
    public static int binarySearch(int[] arr, int target) {
        int left = 0;
        int right = arr.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2; // 오버플로우 방지
            
            if (arr[mid] == target) {
                return mid; // 찾음!
            } else if (arr[mid] < target) {
                left = mid + 1; // 오른쪽 절반 탐색
            } else {
                right = mid - 1; // 왼쪽 절반 탐색
            }
        }
        
        return -1; // 못 찾음
    }
}
```

### 4-2. 재귀를 이용한 구현

```java
public class BinarySearchRecursive {
    
    /**
     * 이진 탐색 - 재귀 버전
     * @param arr 정렬된 배열
     * @param target 찾으려는 값
     * @param left 시작 인덱스
     * @param right 끝 인덱스
     * @return 찾은 인덱스 (없으면 -1)
     */
    public static int binarySearch(int[] arr, int target, int left, int right) {
        if (left > right) {
            return -1; // 탐색 실패
        }
        
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            return mid; // 찾음!
        } else if (arr[mid] < target) {
            return binarySearch(arr, target, mid + 1, right); // 오른쪽 탐색
        } else {
            return binarySearch(arr, target, left, mid - 1); // 왼쪽 탐색
        }
    }
    
    // 편의 메서드
    public static int binarySearch(int[] arr, int target) {
        return binarySearch(arr, target, 0, arr.length - 1);
    }
}
```

### 4-3. 사용 예제

```java
public class BinarySearchExample {
    public static void main(String[] args) {
        int[] sortedArray = {1, 3, 5, 7, 9, 11, 13, 15, 17, 19};
        int target = 7;
        
        int result = BinarySearch.binarySearch(sortedArray, target);
        
        if (result != -1) {
            System.out.println("값 " + target + "을(를) 인덱스 " + result + "에서 찾았습니다!");
        } else {
            System.out.println("값 " + target + "을(를) 찾을 수 없습니다.");
        }
        
        // 출력: 값 7을(를) 인덱스 3에서 찾았습니다!
    }
}
```

---

## 5. 시간 복잡도 분석

| 탐색 방법 | 시간 복잡도 | 공간 복잡도 |
|---------|-----------|-----------|
| **선형 탐색** | O(n) | O(1) |
| **이진 탐색** | O(log n) | O(1) |

### 성능 차이 예시
- **배열 크기가 1,000개일 때**
  - 선형 탐색: 최대 1,000번 비교
  - 이진 탐색: 최대 10번 비교
  
- **배열 크기가 1,000,000개일 때**
  - 선형 탐색: 최대 1,000,000번 비교  
  - 이진 탐색: 최대 20번 비교

---

## 6. 주의사항

### 6-1. 오버플로우 주의
```java
// 잘못된 방법 (오버플로우 가능)
int mid = (left + right) / 2;

// 올바른 방법
int mid = left + (right - left) / 2;
```

### 6-2. 경계 조건 체크
- `left <= right` 조건 확인
- 배열 인덱스 범위 체크
- 빈 배열 처리

### 6-3. 정렬 상태 확인
이진 탐색은 반드시 정렬된 배열에서만 동작합니다!

---

## 7. 문제

이진 탐색을 연습할 수 있는 백준 문제들입니다:

### 기초 문제
1. **[1920번 - 수 찾기](https://www.acmicpc.net/problem/1920)** `실버 4`
   - 기본적인 이진 탐색 구현 문제

### 중급 문제  
2. **[2805번 - 나무 자르기](https://www.acmicpc.net/problem/2805)** `실버 2`
   - 매개변수 탐색 (Parametric Search) 활용

### 고급 문제
3. **[2110번 - 공유기 설치](https://www.acmicpc.net/problem/2110)** `골드 4`
   - 최적화 문제에 이진 탐색 적용
