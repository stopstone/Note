# Call by Value vs Call by Reference

> "Java는 Call by Reference 아닌가요?"  
> 면접에서 자주 나오는 질문 중 하나입니다.  
> Java와 Kotlin 모두 **Call by Value** 방식입니다.  
>
> 다만, **객체를 전달할 때 참조값을 복사해 전달**하기 때문에 Call by Reference처럼 보일 수 있습니다.   
> 이 글에서는 **Call by Value와 Call by Reference의 개념**을 먼저 짚고,  
> Java와 Kotlin에서 객체를 전달할 때의 동작 방식을 살펴보겠습니다.  

---

## 1. Call by Value와 Call by Reference의 차이

| 구분                | 설명                                       |
| ----------------- | ---------------------------------------- |
| Call by Value     | 함수에 인자의 값을 복사하여 전달. 내부에서 값을 변경해도 원본은 유지됨 |
| Call by Reference | 함수에 참조 자체를 전달. 내부에서 값을 변경하면 원본도 같이 변경됨   |

### 1-1. Call by Value 예시 (Java)

```java
void update(int value) {
    value = 100;
}

int x = 10;
update(x);
// x는 여전히 10
```

→ 값이 복사되므로 원본은 변경되지 않습니다.

### 1-2. Call by Reference 예시 (C++ 등)

```cpp
void update(int& value) {
    value = 100;
}

int x = 10;
update(x);
// x는 100으로 바뀜
```

→ 참조를 넘기면 원본이 변경됩니다.

---

## 2. Java는 Call by Value

Java는 항상 **Call by Value**입니다.
기본형은 값 자체가 복사되고, 객체는 **참조값(reference)** 이 복사됩니다.

### 2-1. 객체 내부 변경은 가능

```java
class Person {
    String name;
}

void rename(Person p) {
    p.name = "짱구";
}

Person p = new Person();
p.name = "훈이";
rename(p);
// p.name == "짱구"
```

→ 참조값을 복사했기 때문에 내부 상태 변경은 가능합니다.

### 2-2. 참조 자체 변경은 불가능

```java
void changePerson(Person p) {
    p = new Person();
    p.name = "철수";
}

Person p = new Person();
p.name = "짱구";
changePerson(p);
// p.name == "짱구"
```

→ 참조값 자체를 바꾸더라도, 복사된 참조이므로 원본은 영향을 받지 않습니다.

---

## 3. 요약 정리

| 언어     | 전달 방식         | 객체 내부 변경 | 참조 자체 변경     | 핵심 특징              |
| ------ | ------------- | -------- | ------------ | ------------------ |
| Java   | Call by Value | 가능       | 불가능          | 참조값이 복사됨           |
| Kotlin | Call by Value | 가능       | 불가능 (val 기준) | Java와 동일, 더 엄격한 제한 |
