# Week 9: Broadcast Receivers (브로드캐스트 리시버)

## 1. 핵심 개념: 정적 vs 동적 등록 ⭐ 시험 100% 출제 예상

### 정적 등록 (Static Registration) - "건물 간판"

**위치**: `AndroidManifest.xml` 파일 안의 `<receiver>` 태그

**설명**: 앱을 설치할 때 시스템에 "나 이거 들을 거야!"라고 명단에 올려두는 방식입니다.

**특징**: 앱이 꺼져 있어도 방송이 오면 자동으로 깨어나서 반응합니다.

**⭐ 치명적 단점**:
- 배터리 소모가 심해서 **Android 8.0 (Oreo)** 버전부터 막혔습니다.
- 정적 등록만 하고 커스텀 방송을 보내면 로그캣에 **"Background execution not allowed"** (백그라운드 실행 금지됨) 에러가 뜹니다.
- 💡 **시험 팁**: 이 에러가 나오면 "정적 등록이라서 막혔구나, 동적으로 바꿔야지"라고 생각하세요!

### 동적 등록 (Dynamic Registration) - "무전기"

**위치**: Java 코드 (주로 `MainActivity.java`)

**설명**: 앱이 실행되는 도중(코드 실행 중)에 "지금부터 들을게요"라고 등록하는 방식입니다.

**특징**: 앱이 켜져 있을 때만 방송을 수신합니다. 앱이 꺼지면 못 듣습니다.

**장점**: 
- 필요한 순간에만 켜니까 배터리를 아끼고 효율적입니다.
- 최신 안드로이드에서는 이 방식을 권장합니다. (실습의 정답!)

---

## 2. 필수 터미널 명령어 (ADB) 상세 해설

### dumpsys (시스템 상태 조작)
```bash
dumpsys battery set level 5
```

**해설**: 
- "Dump System info"의 약자입니다.
- 안드로이드 시스템(OS)에게 "지금 배터리가 5% 남은 걸로 쳐!"라고 거짓 정보를 입력하는 겁니다.
- 실제로 배터리가 닳는 게 아니라, 테스트를 위해 환경만 조작하는 것입니다.

### am (앱에게 강제 명령)
```bash
am broadcast -a com.example.BATTERY_LOW_TEST
```

**해설**:
- "Activity Manager"의 약자입니다.
- 시스템이 배터리 부족 신호를 안 보내줄 때, 답답하니까 우리가 직접 "야! 앱들아, 이 방송 받아라!" 하고 강제로 신호를 쏘는 겁니다.
- **옵션 `-a`**: Action(행동/신호 이름)을 지정하겠다는 뜻입니다.

### com.example... (패키지 이름 규칙)

**의미**: 
- 특별한 기능이 있는 명령어가 아닙니다. 그냥 "이름표"입니다.
- 전 세계 앱 이름이 겹치지 않게 하기 위해 `[com.회사이름.기능]` 형식으로 짓는 규칙입니다.

**⭐ 주의사항**: 
- 명령어에 적는 이름(보내는 쪽)과 코드에 적힌 이름(받는 쪽)이 띄어쓰기 하나까지 **완벽하게 똑같아야** 작동합니다.
- 오타 나면 아무 반응 없습니다.

---

## 3. 코드 추적 가이드 (탐정 놀이: 순서 찾는 법)

코드가 여러 파일로 나뉘어 있어서 헷갈릴 때는 "문자열(Action String)"을 암호처럼 따라가세요.

### 단계 1: 범인 식별 (무슨 방송이지?)
- 명령어(`am broadcast`) 뒤에 적힌 "com.example.BATTERY_LOW_TEST" 같은 이름을 확인합니다.

### 단계 2: 수신자 찾기 (누가 듣기로 했지?)
- `AndroidManifest.xml` 파일이나 `MainActivity.java` 파일을 봅니다.
- `<action android:name="...">` 또는 `filter.addAction("...")` 안에 방금 본 그 이름이 적혀 있는지 찾습니다.
- 찾았다면 그 코드가 연결된 **Java 파일 이름(예: SaveData)**을 확인합니다.

### 단계 3: 실행 확인 (무슨 행동을 하지?)
- 위에서 찾은 Java 파일(`SaveData.java`)을 엽니다.
- 방송을 받으면 무조건 **`onReceive()`** 함수가 실행됩니다.
- 그 함수 안의 `Log.d`나 `Toast` 코드가 실제 결과입니다.

---

## 4. 실습 과제 해결 (학번/이름 출력하기)

### 과제 목표
방송을 받았을 때 로그캣(Logcat)에 내 학번과 이름을 띄우기

### 수정할 파일
`SaveData.java` (리시버 파일)

### 수정할 위치
`onReceive()` 함수 중괄호 `{ }` 안

### 코드 예시 및 상세 설명
```java
@Override
public void onReceive(Context context, Intent intent) {
    // 1. 들어온 방송의 이름(Action)을 꺼내서 확인합니다.
    String action = intent.getAction();

    // 2. 로그(Log)를 출력합니다. (여기가 과제 핵심!)
    // Log.d("검색용태그", "출력할메시지"); 
    // "MyTag" 부분은 나중에 로그캣 검색창에서 찾기 쉽게 적는 키워드입니다.
    Log.d("MyTag", "학번: 20241234, 이름: 홍길동, 받은신호: " + action);
    
    // 3. (선택사항) 화면에 토스트 메시지 띄우기
    // 만약 받은 신호가 우리가 정한 테스트 신호와 같다면?
    if ("com.example.BATTERY_LOW_TEST".equals(action)) {
        // 화면 하단에 잠깐 떴다 사라지는 메시지를 보여줍니다.
        Toast.makeText(context, "배터리 부족! (홍길동)", Toast.LENGTH_SHORT).show();
    }
}
```

---

## 5. 동적 등록 구현 코드 (MainActivity.java 복붙용)

정적 등록이 막혔을 때(에러 날 때) 해결하는 코드입니다.
`MainActivity.java` 파일의 `onCreate()` 함수 안에 아래 내용을 적으면 됩니다.

```java
// 1. IntentFilter 만들기 (라디오 주파수 맞추기)
// "이 필터(귀)를 통해서 방송을 걸러서 듣겠다"는 뜻입니다.
IntentFilter filter = new IntentFilter();

// 2. 듣고 싶은 방송 추가하기 (주파수 채널 추가)
// 실제 배터리 부족 신호
filter.addAction("android.intent.action.BATTERY_LOW");
// 우리가 실습 때 강제로 쏘는 테스트 신호 (오타 주의!)
filter.addAction("com.example.BATTERY_LOW_TEST"); 

// 3. 리시버 등록하기 (라디오 전원 켜기)
// registerReceiver(누가들을건지, 어떤필터로들을건지);
// saveDataReceiver는 미리 만들어둔 SaveData 객체여야 합니다.
registerReceiver(saveDataReceiver, filter);

// [참고] 앱이 꺼질 때 해제하는 법 (onDestroy 함수에 작성)
// 리시버를 계속 켜두면 메모리가 낭비되므로 끄는 것이 정석입니다.
// unregisterReceiver(saveDataReceiver);
```
