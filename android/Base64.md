# Base64

> 바이너리 데이터를 텍스트 환경에서 다뤄야 할 때, 우리는 Base64라는 방식으로 데이터를 안전하게 감쌀 수 있습니다.  
> 이 글에서는 Base64가 무엇인지, 왜 필요한지, 그리고 Android 개발에서 언제 사용되는지 살펴보겠습니다.  
  
---

## 1. Base64란?

Base64는 **이진 데이터를 텍스트 형태로 인코딩**하는 방식입니다.  
총 64개의 ASCII 문자 집합(`A-Z`, `a-z`, `0-9`, `+`, `/`)을 사용하며,  
**3바이트 데이터를 4문자로 변환**하여 텍스트로 표현할 수 있게 해줍니다.  

예시:

```text
hello → aGVsbG8=
```

Base64는 **암호화가 아닌 단순 인코딩**이기 때문에 보안 목적이 아닌 **전송 안정성 확보**를 위한 용도로 쓰입니다.  

---

## 2. 왜 Base64가 필요할까?

HTTP, JSON, XML, SMTP 등은 기본적으로 **텍스트 기반 프로토콜**입니다.  
이런 시스템에서 **바이너리 데이터를 그대로 다루면 손상되거나 파싱 오류**가 발생할 수 있기 때문에  
**안전한 전송을 위해** 바이너리를 텍스트로 바꾸는 Base64 인코딩이 사용됩니다.  

---

## 3. 사용 예시

### ✅ Android에서의 활용 사례

| 상황                | 설명                                          |
| ----------------- | ------------------------------------------- |
| 이미지 업로드           | Base64로 변환 후 POST body에 삽입                  |
| JWT 토큰            | Header.Payload.Signature 구조에서 각각 Base64 인코딩 |
| QR 코드             | QR 문자열에 이미지/파일 데이터 삽입 시 사용                  |
| SharedPreferences | 직렬화된 객체를 Base64로 인코딩해 문자열로 저장               |

---

## 4. 단점

| 단점    | 설명                                     |
| ----- | -------------------------------------- |
| 용량 증가 | 3바이트 → 4문자 → 약 33% 증가                  |
| 보안 없음 | 암호화 아님. 누구나 디코딩 가능                     |
| 성능 부담 | 큰 데이터를 처리할 경우 메모리 사용량 증가, 성능 저하 가능성 있음 |

---

## 5. Kotlin 코드 예시

```kotlin
import android.util.Base64

// 인코딩
val original = "hello"
val encoded = Base64.encodeToString(original.toByteArray(), Base64.DEFAULT)

// 디코딩
val decoded = String(Base64.decode(encoded, Base64.DEFAULT))
```

---

## 참고자료

* [Wikipedia - Base64](https://en.wikipedia.org/wiki/Base64)
* [RFC 4648 - Base64 Data Encodings](https://datatracker.ietf.org/doc/html/rfc4648)
* [Android Developers - Base64](https://developer.android.com/reference/android/util/Base64)
