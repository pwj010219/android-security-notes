# Week 7: Android Intents (인텐트의 개념과 종류)

## 1. 명시적 인텐트 (Explicit Intent)

### 정의
- 메시지 객체(Message Object) 또는 심부름 쪽지/택배 송장과 같음

### 인텐트의 3요소

#### 1. Action (할 일)
- 수행할 작업 (The JOB)
- 예시: "이것 좀 해줘" → "보여줘(Show)", "공유해줘(Share)", "찍어줘(Take a picture)"

#### 2. Component (대상/받는 이)
- 메시지를 받을 대상 (The RECEIVER)
- 예시: "누구에게" → 특정 화면(Activity) 또는 작업자(Service)

#### 3. Data (데이터)
- 전달할 내용물 (The STUFF)
- 예시: "무엇을 가지고" → 웹 링크(URL), 사진, 또는 텍스트

### 비유 (Analogy)
> "스미스 교수님께 이 쪽지를 전해드려."
> (받는 사람의 이름을 정확히 명시함)

---

## 2. 암시적 인텐트 (Implicit Intent)

### 비유 (Analogy)
> "사진 찍을 수 있는 사람 아무나 좀 찾아줘."
> (수행할 '작업'이 중요하며, 수행하는 대상의 이름은 중요하지 않음)

---

## 3. 명시적 인텐트 사용 코드 예제

### 방법 1: 클래스(Class) 참조 방식
```java
// MainActivity에서 SettingActivity로 이동
Intent intent = new Intent(MainActivity.this, SettingActivity.class);
startActivity(intent);
```

### 방법 2: ComponentName 사용 방식
```java
Intent intent = new Intent();
// 패키지명과 클래스명을 사용하여 대상 지정
ComponentName cn = new ComponentName("com.example.app", "com.example.app.TargetActivity");
intent.setComponent(cn);
startActivity(intent);
```

### 방법 3: setClassName 사용 방식
```java
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.example.app", "com.example.app.TargetActivity"));
startActivity(intent);
```
