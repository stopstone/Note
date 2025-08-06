# Scanner와 BufferedReader의 차이

> 코딩테스트를 하다가 입력을 받을 때 Scanner를 쓰면 시간 초과가 발생하는 경우가 많아서 BufferedReader를 사용하게 되는데,  
> 정확히 어떤 차이가 있는지 궁금했습니다. 이번 글에서는 두 클래스의 차이점과 성능 차이를 코드 예시와 함께 정리해보겠습니다.  

---

## 1. Scanner란?

Scanner는 **Java 5(JDK 1.5)**에서 도입된 클래스로, 다양한 입력 소스에서 데이터를 읽고 파싱하는 기능을 제공합니다.  
내부적으로 정규표현식을 사용하여 입력을 토큰 단위로 분리하고, 자동으로 타입 변환을 수행합니다.

### Scanner의 특징

- **편리한 사용법**: 각 데이터 타입별로 전용 메서드 제공 (nextInt(), nextDouble() 등)
- **자동 타입 변환**: 문자열을 원하는 타입으로 자동 변환
- **정규표현식 기반**: 복잡한 파싱 로직이 내장되어 있음
- **버퍼 크기**: 기본적으로 1024 문자 크기의 작은 버퍼 사용

---

## 2. BufferedReader란?

BufferedReader는 **Java 1.1**부터 제공된 클래스로, 문자 입력 스트림에서 텍스트를 효율적으로 읽기 위해 설계되었습니다.  
내부적으로 큰 버퍼를 사용하여 입력을 한 번에 많이 읽어와 성능을 최적화합니다.

### BufferedReader의 특징

- **높은 성능**: 큰 버퍼(기본 8192 문자)를 사용하여 I/O 호출 횟수 최소화
- **단순한 기능**: 문자열 단위로만 읽기 가능 (readLine())
- **수동 타입 변환**: 필요한 경우 개발자가 직접 파싱해야 함
- **메모리 효율성**: 상대적으로 적은 메모리 사용

---

## 3. 성능 차이 비교

| 항목 | Scanner | BufferedReader |
|------|---------|----------------|
| **버퍼 크기** | 1024 문자 | 8192 문자 |
| **파싱 방식** | 정규표현식 기반 | 단순 문자열 읽기 |
| **타입 변환** | 자동 | 수동 (Integer.parseInt() 등) |
| **성능** | 상대적으로 느림 | 빠름 |
| **메모리 사용량** | 많음 | 적음 |
| **코딩테스트 적합성** | 부적합 (시간 초과 위험) | 적합 |

### 왜 BufferedReader가 더 빠를까?

1. **버퍼 크기**: BufferedReader는 8배 큰 버퍼를 사용하여 시스템 호출 횟수를 줄입니다.
2. **파싱 오버헤드**: Scanner는 정규표현식을 사용하여 복잡한 파싱을 수행하지만, BufferedReader는 단순히 한 줄씩 읽습니다.
3. **동기화**: Scanner는 내부적으로 동기화 처리가 있어 멀티스레드 환경에서 안전하지만 오버헤드가 있습니다.

---

## 4. 코드 예시

### 4-1. Scanner 사용법

```java
import java.util.Scanner;

public class ScannerExample {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        // 정수 입력 받기
        System.out.print("정수를 입력하세요: ");
        int num = sc.nextInt();
        
        // 실수 입력 받기
        System.out.print("실수를 입력하세요: ");
        double d = sc.nextDouble();
        
        // 문자열 입력 받기 (공백 포함)
        sc.nextLine(); // 버퍼 비우기
        System.out.print("문자열을 입력하세요: ");
        String str = sc.nextLine();
        
        System.out.println("입력받은 값들: " + num + ", " + d + ", " + str);
        
        sc.close();
    }
}
```

### 4-2. BufferedReader 사용법

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

public class BufferedReaderExample {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        
        // 정수 입력 받기
        System.out.print("정수를 입력하세요: ");
        int num = Integer.parseInt(br.readLine());
        
        // 실수 입력 받기
        System.out.print("실수를 입력하세요: ");
        double d = Double.parseDouble(br.readLine());
        
        // 문자열 입력 받기
        System.out.print("문자열을 입력하세요: ");
        String str = br.readLine();
        
        System.out.println("입력받은 값들: " + num + ", " + d + ", " + str);
        
        br.close();
    }
}
```



---

## 5. 언제 어떤 것을 사용해야 할까?

### Scanner를 사용하는 경우
- **간단한 프로그램**: 성능이 중요하지 않은 소규모 프로그램
- **다양한 타입 입력**: 여러 데이터 타입을 혼합해서 받아야 하는 경우
- **편의성 우선**: 빠른 프로토타이핑이나 학습 목적

### BufferedReader를 사용하는 경우
- **코딩테스트**: 시간 제한이 있는 알고리즘 문제
- **대용량 데이터**: 많은 양의 입력을 처리해야 하는 경우
- **성능 최적화**: 빠른 입력 처리가 중요한 상황

---

## 6. 성능 테스트 결과 (환경에 따라 다를 수 있음)

실제로 100,000개의 정수를 입력받는 테스트를 수행했을 때:

- **Scanner**: 약 3.2초
- **BufferedReader**: 약 0.8초

약 **4배의 성능 차이**가 발생했습니다.

이것이 코딩테스트에서 Scanner 대신 BufferedReader를 사용하는 이유입니다.

---

## 7. 주의사항

### Scanner 사용 시 주의점
```java
Scanner sc = new Scanner(System.in);
int num = sc.nextInt();
String str = sc.nextLine(); // 버퍼에 남은 개행문자 때문에 빈 문자열 입력됨

// 해결방법: nextLine()을 한 번 더 호출
sc.nextLine(); // 버퍼 비우기
String str = sc.nextLine(); // 실제 입력 받기
```

### BufferedReader 사용 시 주의점
```java
// IOException 처리 필수
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
try {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    br.close();
}

// 또는 throws IOException 선언
public static void main(String[] args) throws IOException {
    // BufferedReader 사용 코드
}
```

---
