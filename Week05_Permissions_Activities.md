# Week 5: Android Manifest, Permission & Activity Hacking

## 1. 앱 권한 및 컴포넌트 (App Permissions & Components)

### 앱 권한 (App Permissions)
- 앱이 요청하는 권한 목록

### 앱 컴포넌트 (App Components) - 4대 요소
1. **액티비티 (Activities)**: 화면 (로그인, 비밀번호, 옵션 화면 등)
2. **서비스 (Services)**: 백그라운드 작업 (음악 재생 유지 등)
3. **브로드캐스트 리시버 (Broadcast Receivers)**: 시스템 알림 (배터리 부족, 긴급 알림 등)
4. **콘텐츠 프로바이더 (Content Providers)**: 다른 앱과 데이터를 안전하게 공유하고 접근

---

## 2. 설정 및 보호 수준 (Filters & Protection)

### 인텐트 필터 (Intent Filters)
- 컴포넌트가 다른 앱이나 시스템 이벤트로부터 어떻게 시작될 수 있는지 지정

### 권한 및 보호 수준 (Permissions and Protection Levels)
- 사용자 지정 권한과 보호 수준을 정의 (예: normal, dangerous, signature)

---

## 3. 권한 설정 태그 비교

### `<uses-permission>` vs `<permission>`

#### `<uses-permission>`
- "이 앱이 어떤 권한을 요청하는가?" (What permissions does this app request)
- 외부의 권한을 사용하겠다고 선언

#### `<permission>`
- "내 앱이 새로운 권한을 정의함" (My app defines new permissions)
- 다른 앱이 내 앱에 접근할 때 필요한 권한을 설정

---

## 4. 액티비티 노출과 보안 (Activity Exported)

### 명시적 노출 (Explicit)
- **설정**: `exported="true"`
- **의미**: 외부(Outside world)에서 이 액티비티를 시작할 수 있음
  - 예: `.FileSelectActivity`

### 암시적 노출 (Implicit)
- **설정**: `<intent-filter>` 사용 시
- **의미**: 인텐트 필터가 있으면 암시적으로 외부 접근이 허용됨 (Allowing)

---

## 5. 액티비티 강제 실행 실습 (Activity Manager)

### Activity Manager (AM)
- 액티비티를 시작하는 도구

### 명령어 사용법
```bash
# 1. ADB 쉘 접속
adb shell

# 2. am 도움말 확인
am  # 사용법 출력

# 3. 특정 액티비티 강제 실행
am start -n <패키지명/액티비티명>
# 참고: <> 괄호는 입력하지 않음

# 예시
am start -n com.mwr.example.sieve/.PWList
```
